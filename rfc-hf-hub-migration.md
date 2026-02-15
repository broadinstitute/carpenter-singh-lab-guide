# RFC: Migrate Lab Data Versioning to Hugging Face Hub (Xet Storage)

| Field | Value |
| --- | --- |
| **Status** | Draft |
| **Author** | Shantanu Singh |
| **Created** | 2026-02-15 |
| **Last updated** | 2026-02-15 |

## Summary

Replace rclone/s5cmd + S3 with Hugging Face Hub (Xet-backed) for all lab data
synchronization. GitHub remains the home for code. HF Hub becomes the home for
data. Everything else (Snakemake, Cookiecutter structure, pixi, pre-commit,
Justfile) stays the same.

## Motivation

The current S3-based workflow has no versioning, no branching, and no conflict
prevention. Specific problems:

| Problem | Current behavior |
| --- | --- |
| Two people push simultaneously | Last write wins, silent data loss |
| Pipeline update breaks data | Hope someone has a local copy |
| "Which data made this figure?" | Check Slack / guess |
| Review teammate's results | "Push to S3 and I'll pull" |
| Re-upload 5GB file with 1 column changed | Upload entire 5GB again |
| Audit trail | None |

## Proposal

Use HF Hub dataset repos as the data remote. Each repo is a Xet-backed Git
repository with chunk-level deduplication, branching, commit history, and pull
requests — no infrastructure to run.

### What changes

| Aspect | Before | After |
| --- | --- | --- |
| Data remote | S3 bucket directly | HF Hub (backed by S3 + Xet) |
| Sync tool | rclone + s5cmd | `hf` CLI |
| Auth | AWS credentials | HF token |
| Conflict prevention | Slack coordination | Branches + PRs |
| Cost | AWS S3 charges | HF Team plan ($20/user/mo) + excess storage |
| Infrastructure to maintain | None | None (HF is managed) |

### What does not change

- Snakemake pipelines
- Cookiecutter directory structure
- pixi environments
- Notebook naming conventions
- GitHub for code
- `.gitignore` patterns
- `src` layout and Python packages
- Pre-commit hooks

---

## Detailed Design

### 1. Hugging Face Organization

The lab org is live: <https://huggingface.co/carpenter-singh-lab>

Ask a maintainer to add you as a member.

### 2. Install tools

```bash
# Add to your pixi dependencies
pixi add huggingface_hub hf_xet

# Or install standalone
pip install -U huggingface_hub hf_xet

# Login (each team member, one-time)
hf auth login
```

**Quick testing without installing**: Use `pixi exec` to run `hf` in a
temporary environment. The CLI binary is `hf` (not `huggingface-cli`):

```bash
# Ad-hoc hf commands — nothing installed into your project
pixi exec --spec huggingface_hub -c conda-forge -- hf version
pixi exec --spec huggingface_hub -c conda-forge -- hf auth whoami

# hf_xet is not on conda-forge — use pixi add or pip for Xet-accelerated transfers
```

### 3. Project `.env` file

Each project needs a `.env` at the repo root (already in `.gitignore`):

```bash
HF_PROJECT=myproject
# HF_BRANCH=your-username  # defaults to `whoami` if unset
```

### 4. Create repos for your project

Each project gets one or more dataset repos on HF Hub. A natural mapping:

```bash
# One repo per data layer keeps things clean
hf repo create carpenter-singh-lab/myproject-inputs \
  --repo-type dataset --private

hf repo create carpenter-singh-lab/myproject-results \
  --repo-type dataset --private
```

**Why two repos?** Inputs (raw + external) are read-only for most team members
and change rarely. Results (interim + processed) change constantly. Separating
them means you can give different access levels and avoid syncing huge input
data when you only need results.

For very large projects you might also split by data type
(e.g., `myproject-images`, `myproject-profiles`).

### 5. Initial data upload

```bash
# Upload existing input data
hf upload carpenter-singh-lab/myproject-inputs ./data/external external/ \
  --repo-type dataset --commit-message "Initial external data"

hf upload carpenter-singh-lab/myproject-inputs ./data/raw raw/ \
  --repo-type dataset --commit-message "Initial raw data"

# Upload existing results
hf upload carpenter-singh-lab/myproject-results ./data/interim interim/ \
  --repo-type dataset --commit-message "Initial interim data"

hf upload carpenter-singh-lab/myproject-results ./data/processed processed/ \
  --repo-type dataset --commit-message "Initial processed results"
```

For very large initial uploads that need `path_in_repo`, use `upload_folder`:

```python
from huggingface_hub import HfApi
api = HfApi()
api.upload_folder(
    folder_path="./data/raw",
    path_in_repo="raw",
    repo_id="carpenter-singh-lab/myproject-inputs",
    repo_type="dataset",
)
```

For extremely large uploads (resumable across interruptions), use the CLI.
Note: `upload-large-folder` always uploads to the repo root (no `path_in_repo`),
so structure your local directory to match the desired repo layout:

```bash
hf upload-large-folder carpenter-singh-lab/myproject-inputs ./data/raw \
  --repo-type dataset --num-workers 16
```

### 6. Justfile

```just
set dotenv-load := true

# ==================== PROJECT CONFIGURATION ====================
# Hugging Face Hub configuration
HF_ORG := "carpenter-singh-lab"
HF_PROJECT := env_var_or_default("HF_PROJECT", "myproject")
HF_INPUTS_REPO := HF_ORG + "/" + HF_PROJECT + "-inputs"
HF_RESULTS_REPO := HF_ORG + "/" + HF_PROJECT + "-results"

# Branch name defaults to system username
HF_BRANCH := env_var_or_default("HF_BRANCH", `whoami`)

# Patterns to exclude from uploads (each needs its own --exclude flag)
HF_EXCLUDE := '--exclude "*.DS_Store" --exclude "__pycache__" --exclude "*.pyc" --exclude ".ipynb_checkpoints" --exclude "*.tmp"'

CORES := env_var_or_default("CORES", "all")
SNAKEMAKE := "pixi run snakemake"

# Directory names (unchanged from current workflow)
DATA_DIR := "data"
EXTERNAL_DIR := DATA_DIR + "/external"
RAW_DIR := DATA_DIR + "/raw"
INTERIM_DIR := DATA_DIR + "/interim"
PROCESSED_DIR := DATA_DIR + "/processed"

default:
    @just --list

# ==================== BRANCH MANAGEMENT ====================

# Create your working branch
branch-create:
    @echo "Creating branch '{{HF_BRANCH}}' on results repo..."
    hf repo branch create {{HF_RESULTS_REPO}} {{HF_BRANCH}} --repo-type dataset

# List all branches
branch-list:
    #!/usr/bin/env bash
    echo "Branches on {{HF_RESULTS_REPO}}:"
    python3 -c "
    from huggingface_hub import list_repo_refs
    refs = list_repo_refs('{{HF_RESULTS_REPO}}', repo_type='dataset')
    for b in refs.branches:
        print(f'  {b.name}')
    "

# ==================== MAIN WORKFLOW ====================

# Run full pipeline
run:
    @echo "Running pipeline with {{CORES}} cores..."
    @{{SNAKEMAKE}} all --cores {{CORES}} --printshellcmds --scheduler greedy

# Preview what will run
dry:
    @{{SNAKEMAKE}} --cores {{CORES}} -n -p --scheduler greedy

# ==================== DATA SYNC ====================

# Download input data from main (shared, read-only)
get-inputs:
    @echo "Downloading inputs from {{HF_INPUTS_REPO}} (main)..."
    @mkdir -p {{EXTERNAL_DIR}} {{RAW_DIR}}
    hf download {{HF_INPUTS_REPO}} --repo-type dataset \
      --include "external/*" --include "raw/*" \
      --local-dir {{DATA_DIR}}

# Download results from main
get-results:
    @echo "Downloading results from {{HF_RESULTS_REPO}} (main)..."
    @mkdir -p {{INTERIM_DIR}} {{PROCESSED_DIR}}
    hf download {{HF_RESULTS_REPO}} --repo-type dataset \
      --include "interim/*" --include "processed/*" \
      --local-dir {{DATA_DIR}}

# Download results from a specific branch
get-results-from branch:
    @echo "Downloading results from branch '{{branch}}'..."
    @mkdir -p {{INTERIM_DIR}} {{PROCESSED_DIR}}
    hf download {{HF_RESULTS_REPO}} --repo-type dataset \
      --revision {{branch}} \
      --include "interim/*" --include "processed/*" \
      --local-dir {{DATA_DIR}}

# Download specific analysis results
get-results-for run_path:
    @echo "Downloading results for: {{run_path}}"
    @mkdir -p {{PROCESSED_DIR}}/{{run_path}}
    hf download {{HF_RESULTS_REPO}} --repo-type dataset \
      --include "processed/{{run_path}}/*" \
      --local-dir {{DATA_DIR}}

# Upload results to YOUR BRANCH (safe, isolated)
put-results:
    @echo "Uploading results to branch '{{HF_BRANCH}}'..."
    hf upload {{HF_RESULTS_REPO}} {{INTERIM_DIR}} interim/ \
      --repo-type dataset --revision {{HF_BRANCH}} \
      {{HF_EXCLUDE}} \
      --commit-message "Update interim results"
    hf upload {{HF_RESULTS_REPO}} {{PROCESSED_DIR}} processed/ \
      --repo-type dataset --revision {{HF_BRANCH}} \
      {{HF_EXCLUDE}} \
      --commit-message "Update processed results"

# Upload specific analysis results to your branch
put-results-for run_path:
    @echo "Uploading {{run_path}} to branch '{{HF_BRANCH}}'..."
    hf upload {{HF_RESULTS_REPO}} \
      {{PROCESSED_DIR}}/{{run_path}} processed/{{run_path}}/ \
      --repo-type dataset --revision {{HF_BRANCH}} \
      {{HF_EXCLUDE}} \
      --commit-message "Update {{run_path}}"

# Upload results directly to main (maintainers only)
put-results-main:
    @echo "Uploading results directly to main..."
    hf upload {{HF_RESULTS_REPO}} {{INTERIM_DIR}} interim/ \
      --repo-type dataset {{HF_EXCLUDE}} \
      --commit-message "Update interim results"
    hf upload {{HF_RESULTS_REPO}} {{PROCESSED_DIR}} processed/ \
      --repo-type dataset {{HF_EXCLUDE}} \
      --commit-message "Update processed results"

# Upload results to main as a pull request for review
pr-create message="Merge results":
    @echo "Creating PR with results from branch '{{HF_BRANCH}}'..."
    hf upload {{HF_RESULTS_REPO}} {{INTERIM_DIR}} interim/ \
      --repo-type dataset --create-pr \
      {{HF_EXCLUDE}} \
      --commit-message "{{message}}: interim"
    hf upload {{HF_RESULTS_REPO}} {{PROCESSED_DIR}} processed/ \
      --repo-type dataset --create-pr \
      {{HF_EXCLUDE}} \
      --commit-message "{{message}}: processed"
    @echo "PR(s) created. Review and merge at:"
    @echo "  https://huggingface.co/datasets/{{HF_RESULTS_REPO}}/discussions"

# ==================== DATA ADMIN (Maintainers Only) ====================

# Upload new input data to main
put-inputs:
    @echo "Uploading inputs to {{HF_INPUTS_REPO}}..."
    hf upload {{HF_INPUTS_REPO}} {{EXTERNAL_DIR}} external/ \
      --repo-type dataset {{HF_EXCLUDE}} \
      --commit-message "Update external data"
    hf upload {{HF_INPUTS_REPO}} {{RAW_DIR}} raw/ \
      --repo-type dataset {{HF_EXCLUDE}} \
      --commit-message "Update raw data"

# Download from original sources then upload to HF
# NOTE: Assumes project has a downloading module — adjust import path per project
get-from-sources:
    @echo "Downloading from original sources..."
    pixi run python -m {{HF_PROJECT}}.downloading.download_data

# ==================== HISTORY & INSPECTION ====================

# Show commit log for results repo
log:
    #!/usr/bin/env bash
    python3 -c "
    from huggingface_hub import list_repo_commits
    commits = list_repo_commits('{{HF_RESULTS_REPO}}', repo_type='dataset', revision='{{HF_BRANCH}}')
    for c in commits[:20]:
        print(f'{c.commit_id[:8]}  {c.created_at:%Y-%m-%d %H:%M}  {c.title}')
    "

# Show commit log for main
log-main:
    #!/usr/bin/env bash
    python3 -c "
    from huggingface_hub import list_repo_commits
    commits = list_repo_commits('{{HF_RESULTS_REPO}}', repo_type='dataset')
    for c in commits[:20]:
        print(f'{c.commit_id[:8]}  {c.created_at:%Y-%m-%d %H:%M}  {c.title}')
    "

# Download a specific historical version
get-results-at revision:
    @echo "Downloading results at revision {{revision}}..."
    hf download {{HF_RESULTS_REPO}} --repo-type dataset \
      --revision {{revision}} \
      --include "interim/*" --include "processed/*" \
      --local-dir {{DATA_DIR}}

# Tag current state of a branch for reproducibility (e.g., just tag paper-v1)
tag name:
    @echo "Tagging '{{HF_BRANCH}}' as '{{name}}'..."
    hf repo tag create {{HF_RESULTS_REPO}} {{name}} --repo-type dataset --revision {{HF_BRANCH}}
    @echo "Retrieve with: just get-results-at {{name}}"

# ==================== CODE QUALITY ====================

snakelint:
    #!/usr/bin/env bash
    echo "Running snakemake lint..."
    output=$({{SNAKEMAKE}} --lint 2>&1)
    filtered=$(echo "$output" | grep -v -E "(Specify a conda environment|https://snakemake.readthedocs.io|This way, the used software|workflow can be executed on any machine|Also see:$)")
    echo "$filtered" | grep -v -E "^Lints for rule.*:$" | grep -v "^[[:space:]]*$" || echo "No lint warnings!"

# ==================== UTILITIES ====================

status:
    @{{SNAKEMAKE}} --summary

config:
    @echo "Current Configuration:"
    @echo "  HF_ORG: {{HF_ORG}}"
    @echo "  HF_PROJECT: {{HF_PROJECT}}"
    @echo "  HF_INPUTS_REPO: {{HF_INPUTS_REPO}}"
    @echo "  HF_RESULTS_REPO: {{HF_RESULTS_REPO}}"
    @echo "  HF_BRANCH: {{HF_BRANCH}}"
    @echo "  CORES: {{CORES}}"

# List files in the results repo
list-data:
    #!/usr/bin/env bash
    python3 -c "
    from huggingface_hub import list_repo_tree
    for item in list_repo_tree('{{HF_RESULTS_REPO}}', repo_type='dataset', revision='{{HF_BRANCH}}'):
        print(f'  {item.path}')
    "

# Purge local HF download cache
cache-clear:
    hf cache prune
```

---

## Daily Workflow

### First day

```bash
git clone https://github.com/broadinstitute/PROJECT_NAME.git
cd PROJECT_NAME
pixi install
pre-commit install --hook-type pre-commit --hook-type pre-push

# Login to Hugging Face (one-time)
hf auth login

# Create your working branch
just branch-create

# Get data
just get-inputs
just get-results
```

### Start your day

```bash
git pull                  # Code updates
just get-inputs           # Shared input data (if changed)
just get-results          # Latest merged results from main
just dry                  # Check pipeline status
```

### Work and share

```bash
# Run your analysis...

# Push results to YOUR branch (no risk of clobbering anyone)
just put-results-for my-analysis

# Commit code
git add notebooks/3.01-srs-batch-correction.py
git commit -m "feat: batch correction analysis"
git push
```

### Share with the team

```bash
# Option A: Upload results as a PR for review (merge via HF web UI)
just pr-create "Add batch correction results"

# Option B: Let a teammate preview your branch directly
# (teammate runs:)
just get-results-from srs
```

### When things go wrong

```bash
# See what happened
just log
just log-main

# Download data from a known-good point in time
just get-results-at abc123def456
```

---

## Cost

**Team plan**: $20/user/month. Each seat includes 1 TB of private storage.
A 10-person lab = 10 TB included.

**Enterprise plan**: $50/user/month with advanced security, SSO, resource groups.

**Extra storage**: $25/month per additional TB beyond the included amount.

**Academic grants**: HF offers storage grants for impactful research.
Contact <datasets@huggingface.co> — Cell Painting Gallery is exactly the
kind of thing they support. Worth asking.

**Limits**:

- Max 200 GB per file (50 GB recommended shard size)
- Max 10k files per folder
- Max 100k files per repo (split across multiple repos if needed)

---

## FAQ

**Why not move everything to HF Hub and drop GitHub entirely?**

HF Hub is a great data host, not a development platform. You'd lose:

- **CI/CD**: No GitHub Actions equivalent. No automated tests or linting on push.
- **Code review**: HF PRs use `refs/pr/N` for data commits — no inline code suggestions, thread resolution, or CODEOWNERS.
- **Issue tracking**: HF Discussions are basic. No project boards, labels, or milestones.
- **Ecosystem**: No pre-commit.ci, Codecov, Dependabot, or GitHub Apps.
- **Branch protection**: Minimal compared to GitHub's required reviews and status checks.

The right split: **GitHub for code, HF Hub for data.**

**Is HF Hub designed for arbitrary project data, not just ML models/datasets?**

Technically, dataset repos can hold anything. HF won't police what you put in a
private repo. That said, you lose HF's Data Studio viewer for non-standard
formats. PNGs and Parquet will preview nicely; your DocDB files obviously won't.

**What about the 100k file limit per repo?**

Cell Painting image datasets can be enormous. If you have millions of PNGs, you'd
need to split across repos or use formats like WebDataset (tar archives of
images). For profiles and metadata this is unlikely to be an issue.

**What about bandwidth / download speed?**

Xet's chunk-based transfers are fast. For initial large downloads, performance
should be comparable to or better than rclone from S3, especially for subsequent
syncs where only changed chunks transfer.

**We already pay for S3 — is this double-paying?**

Yes, you'd be paying HF instead of (or in addition to) AWS for storage. But
you're buying versioning, branching, dedup, and a managed service. Evaluate
whether the operational simplicity and safety are worth the cost delta.

**Can we still access data programmatically?**

Not directly from S3 — data lives on HF's infrastructure. But `hf_hub_download`
is fast and caches locally:

```python
import pandas as pd
from huggingface_hub import hf_hub_download

path = hf_hub_download(
    repo_id="carpenter-singh-lab/myproject-results",
    filename="interim/profiles.parquet",
    repo_type="dataset",
)
df = pd.read_parquet(path)
```

**What about local cache eating disk space?**

`hf download` caches everything under `~/.cache/huggingface/`. This grows
silently. Run `just cache-clear` periodically, or set `HF_HOME` to a location
with more space:

```bash
export HF_HOME=/scratch/$USER/.cache/huggingface
```

**How do I authenticate on a cluster or in CI?**

Set `HF_TOKEN` as an environment variable instead of interactive login:

```bash
# In your SLURM job script or CI config
export HF_TOKEN=hf_xxxxxxxxxxxxxxxxxxxx
```

Generate a token at <https://huggingface.co/settings/tokens> (read access is
sufficient for downloading; write access for uploads).

---

## Rollout Plan

1. **Week 1**: Upload one project's data to <https://huggingface.co/carpenter-singh-lab>. Test the Justfile with 2 people.
2. **Week 2**: Run a real pipeline cycle — get inputs, run snakemake, push results on branches, merge.
3. **Week 3**: If it works, keep old S3 as read-only backup. New work goes through HF.
4. **Month 2**: Migrate remaining active projects. Decommission S3 sync for those projects.

Keep the S3 bucket as a backup during the transition. Don't burn bridges until
the team is comfortable.

---

## Open Questions

- Which project should pilot first?
- Do we need an academic storage grant, or is 10 TB sufficient for the pilot?
- Should we request Enterprise plan evaluation for SSO/resource groups?
