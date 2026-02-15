# Project Workflows

> [!IMPORTANT]
> This workflow is being actively tested in our lab. Expect rough edges and please share feedback!

> [!NOTE]
> We use Justfile + Snakemake + rclone + s5cmd for data management. The Justfile provides a clean get/put interface for S3 operations (rclone for sync, s5cmd for listing). Snakemake handles pipeline execution. All data downloads include SHA256 hash verification for integrity.

> [!NOTE]
> Dense documentation - see [README](README.md) for philosophy and links to comprehensive guides.

## How We Organize Projects

We follow [Cookiecutter Data Science](https://cookiecutter-data-science.drivendata.org/) principles with specific adaptations for our lab's needs.

### Core Principles

1. **Data flows in one direction**: `raw/` + `external/` → `interim/` → `processed/`
2. **Raw data is immutable**: Never edit raw data directly
3. **Clear separation of concerns**: Pipeline-managed data vs. personal analysis
4. **Everything is reproducible**: Code + raw data = any output
5. **Data integrity**: All downloaded files verified with SHA256 hashes

### Directory Structure

```text
project-name/
├── data/
│   ├── raw/          # Original, immutable data
│   ├── external/     # Third-party reference data
│   ├── interim/      # Pipeline-generated intermediate data
│   └── processed/    # Your analysis outputs (your workspace)
├── src/
│   └── <PROJECT_NAME>/  # Python package (src layout)
├── rules/            # Modular Snakemake rule files (.smk)
├── configs/          # Pipeline and tool configuration (YAML)
├── notebooks/        # Numbered analysis notebooks (.py)
├── scripts/          # Shell scripts, SQL, and pipeline orchestration (not Python analysis)
├── references/       # Reference documents and data dictionaries
└── docs/             # Documentation
```

### Experiment Tracking

**Use GitHub Issues to track all experiments.**

1. **Create issue before starting**: Title with hypothesis or question
2. **Document everything**: Link notebooks, paste figures, note data locations
3. **Update title when complete**: Change from question to conclusion
4. **Label consistently**: `experiment`, `analysis`, `results`

Example progression:

- Initial: "Does batch correction improve compound clustering?"
- Document approach, link `notebooks/3.01-srs-batch-correction.py`
- Paste key figures showing results
- Final: "PCA-based batch correction improves compound clustering by 15%"

### Notebook Naming

**Format**: `<phase>.<sequence>-<initials>-<description>.py`

**Phase numbers**:

(Adapted from from [CCDS](https://cookiecutter-data-science.drivendata.org/using-the-template/))

- `0`: Exploration
- `1`: Data cleaning/feature engineering
- `2`: Analysis
- `3`: Publication figures

**Example**: `1.03-srs-merge-annotations.py`

**Within each notebook**:

- Read inputs from `data/interim/` or `data/external/`
- Save all outputs to `data/processed/{your-analysis}/`
- Organize outputs by type: `data/processed/{your-analysis}/figures/`, `/tables/`, etc.

## Project Setup

This section covers creating a new project from scratch. Most team members will join existing projects and can skip to [Daily Workflow](#daily-workflow).

### Prerequisites

- Git
- Python 3.12+
- [pixi](https://pixi.sh/) package manager (handles conda + pip dependencies, GPU/RAPIDS support, feature environments)
- [just](https://github.com/casey/just) command runner
- [rclone](https://rclone.org/) for S3 sync operations (properly tracks changes, avoids re-downloads)
- [s5cmd](https://github.com/peak/s5cmd) for S3 listing/browsing (fast, simple output)
- [Snakemake](https://snakemake.github.io/) for pipeline execution
- AWS CLI configured with appropriate credentials

### Complete Setup Process

> [!NOTE]
> Throughout this guide, replace `<PROJECT_NAME>` with your actual project name in all commands and examples.

#### 1. Initialize Project

```bash
# Create and enter project directory
mkdir <PROJECT_NAME>
cd <PROJECT_NAME>

# Example: mkdir cellpainting-analysis
#          cd cellpainting-analysis

# Initialize Git
git init
```

#### 2. Configure Storage

Copy the Justfile template from [jump_production](https://github.com/broadinstitute/jump_production/blob/main/Justfile) and edit the project-specific values:

```just
set dotenv-load := true

# ==================== PROJECT CONFIGURATION ====================
S3_BUCKET := "your-bucket"
S3_PROJECT_PATH := "projects/your-project/datastore"
S3_PREFIX := "s3://" + S3_BUCKET + "/" + S3_PROJECT_PATH

CORES := env_var_or_default("CORES", "all")
AWS_PROFILE := env_var_or_default("AWS_PROFILE", "default")

# Rclone for sync (tracks changes, avoids re-downloads)
RCLONE_FLAGS := env_var_or_default("RCLONE_FLAGS", "--transfers 16 --checkers 16")
RCLONE_SYNC := "rclone sync " + RCLONE_FLAGS + " -v --stats-one-line --s3-provider=AWS --s3-env-auth --s3-region=us-east-1 --exclude '.DS_Store' --exclude '__pycache__/**' --exclude '*.pyc'"

# S5CMD for listing only (faster than rclone for browsing)
S5CMD_FLAGS := env_var_or_default("S5CMD_FLAGS", "--numworkers 16")
S5CMD := "s5cmd " + S5CMD_FLAGS

SNAKEMAKE := "pixi run snakemake"

# Default recipe (shows help)
default:
    @just --list
```

Ensure AWS profile is configured: `export AWS_PROFILE=broad-imaging`

#### 3. Create Directory Structure

```bash
# Create all directories
mkdir -p data/{raw,external,interim,processed}
mkdir -p src/<PROJECT_NAME>
mkdir -p rules configs
mkdir -p notebooks scripts tests
mkdir -p docs references

# Add .gitkeep files to track empty directories
touch data/{raw,external,interim,processed}/.gitkeep
touch notebooks/.gitkeep references/.gitkeep
touch README.md
```

#### 4. Set Up Python Environment

```bash
# Initialize pixi project
pixi init --format pyproject

# Add Python and conda-only dependencies
pixi add python=3.12

# Add Python (pip) dependencies in [project] dependencies in pyproject.toml
# (not [tool.pixi.dependencies] — that's for conda-only packages)
```

Add to `pyproject.toml`:

```toml
[project]
name = "<PROJECT_NAME>"
version = "0.1.0"
requires-python = ">= 3.11"
dependencies = [
    "pandas>=2.2,<3",
    "loguru>=0.7,<0.8",
    "typer>=0.21,<0.22",
    "python-dotenv>=1.0,<2",
    "pooch>=1.8,<2",
    "snakemake>=9.9,<10",
    "pre-commit>=4.5,<5",
    "rust-just>=1.46,<2",
]

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.pixi.pypi-dependencies]
<PROJECT_NAME> = { path = ".", editable = true }

[dependency-groups]
lint = ["ruff>=0.12.0"]
test = ["pytest>=8.4.0"]

[tool.pixi.environments]
default = { solve-group = "default" }
lint = { features = ["lint"], solve-group = "default" }
test = { features = ["test"], solve-group = "default" }
```

> **Why pixi over uv?** pixi handles conda + pip dependencies in one tool. GPU libraries (RAPIDS, CuPy, CUDA), R, and other system-level dependencies require conda channels. Add specialized feature environments as needed (e.g., `rapids`, `cheminformatics`, `marimo`), each with `no-default-feature = true` to isolate conflicting dependencies.

Download Python gitignore template:

```bash
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore
```

> **Note**: Add `data/` to `.gitignore` to prevent accidentally committing large data files.

#### 5. Configure Code Quality Tools

### Ruff Configuration

Add to your `pyproject.toml`:

```toml
[tool.ruff]
line-length = 120
src = ["src"]
target-version = "py312"
include = ["pyproject.toml", "src/**/*.py", "notebooks/**/*.py"]

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "W"]
ignore = [
    "E501",   # Line too long (handled by formatter)
    "N803",   # Argument name should be lowercase (ML convention: X, Y for matrices)
    "N806",   # Variable in function should be lowercase (ML convention: X_train, X_test)
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

### Markdown Linting

Create `.markdownlint.yaml`:

```yaml
# Markdown style configuration
MD007:
  indent: 4          # List indent
MD013: false         # Line length
MD024: false         # Multiple headers with same content
MD029:
  style: ordered     # Ordered list style
MD033: false         # Inline HTML
MD046: false         # Code block style
```

#### 6. Configure Pre-commit Hooks

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: check-added-large-files
        args: [--maxkb=10240]
      - id: check-yaml
      - id: end-of-file-fixer
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.9.1
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

Install hooks:

```bash
# Install pre-commit hooks
pre-commit install --hook-type pre-commit --hook-type pre-push
```

#### 7. Create config and GPU modules

Create `src/<PROJECT_NAME>/config.py`:

```py
import os
from pathlib import Path

from dotenv import load_dotenv
from loguru import logger

load_dotenv()

# Support override via environment variable for development
if os.environ.get("<PROJECT_NAME_UPPER>_ROOT"):
    PROJ_ROOT = Path(os.environ["<PROJECT_NAME_UPPER>_ROOT"]).resolve()
else:
    PROJ_ROOT = Path(__file__).resolve().parents[2]
logger.info(f"PROJ_ROOT path is: {PROJ_ROOT}")

DATA_DIR = PROJ_ROOT / "data"
RAW_DATA_DIR = DATA_DIR / "raw"
INTERIM_DATA_DIR = DATA_DIR / "interim"
PROCESSED_DATA_DIR = DATA_DIR / "processed"
EXTERNAL_DATA_DIR = DATA_DIR / "external"

# CPU count — respects Slurm, cgroups, and taskset limits
CPUS: int = len(os.sched_getaffinity(0))
```

Create `src/<PROJECT_NAME>/gpu.py` (for projects with GPU workloads):

```py
from __future__ import annotations

from functools import lru_cache

from loguru import logger


@lru_cache(maxsize=1)
def has_gpu() -> bool:
    """Check whether a CUDA GPU is available via CuPy."""
    try:
        import cupy

        cupy.cuda.runtime.getDeviceCount()
        return True
    except ImportError:
        return False
    except Exception as e:
        logger.warning(f"CuPy installed but GPU unavailable ({type(e).__name__}: {e}), falling back to CPU")
        return False
```

Usage: `from <PROJECT_NAME>.gpu import has_gpu; HAS_GPU = has_gpu()` — no `--no-gpu` CLI flags. Limit GPUs with `CUDA_VISIBLE_DEVICES=0`.

#### 8. Create Test Pipeline

Create `Snakefile` with modular rule includes:

```python
configfile: "configs/pipeline.yaml"  # if using pipeline config

# Include modular rule files
include: "rules/processing.smk"

rule all:
    input:
        "data/interim/test.txt",
```

Create `rules/processing.smk`:

```python
rule prepare_data:
    output:
        "data/interim/test.txt"
    shell:
        "echo 'test' > {output}"
```

As the pipeline grows, split rules into separate `.smk` files by function (e.g., `rules/analysis.smk`, `rules/visualization.smk`).

Test it:

```bash
pixi run snakemake --cores 1 --scheduler greedy
git add .
git commit -m "feat: initial project structure with pipeline"
# pre-commit hooks might update files upon commit, so you may need to git add again
```

## Daily Workflow

This section covers the standard workflow for team members working on existing projects.

### Understanding Roles

**Analysts/Scientists (most team members):**

- Work in `data/processed/` - your personal workspace
- Read from `data/interim/` - pipeline outputs
- Never modify `raw/`, `external/`, or `interim/`

**Data Maintainers (1-2 designated people):**

- Run pipelines when upstream data changes
- Update processing scripts
- Coordinate team-wide data updates

### First Day

Once the repo is setup here's what someone else - or you starting over - would need to do:

```bash
# Clone and install
git clone https://github.com/yourusername/<PROJECT_NAME>.git
cd <PROJECT_NAME>
pixi install

# Configure AWS profile if needed
export AWS_PROFILE=your-profile
# Example: export AWS_PROFILE=imaging-platform

pre-commit install --hook-type pre-commit --hook-type pre-push
```

### Start Your Day

```bash
# 1. Get code updates
git pull

# 2. Get latest data
just get-inputs    # Get input data (external + profiles) from team S3
just get-results   # Get processed results from team S3

# 3. Check pipeline status
just dry           # Preview what would run
# If it shows work to do: just run
```

### Working with Large Datasets

For selective downloads:

```bash
# Download specific run
just get-results-for specific-analysis

# List available data
just list-s3
# Or directly: s5cmd ls s3://bucket/path/
```

### Running Your Analysis

1. **Create notebook**: `notebooks/1.01-abc-analysis-name.py`
2. **Load data**: Read from `data/interim/`
3. **Save outputs**: Write to `data/processed/your-analysis/`
4. **Track experiment**: Create GitHub issue with hypothesis, link notebook, paste key figures

### Sharing Your Work

```bash
# Push your analysis to S3
just put-results-for your-analysis

# Commit code changes
git add notebooks/your-notebook.py
git commit -m "feat: add analysis of X showing Y"
git push
```

### Commits

Use [Conventional Commits](https://www.conventionalcommits.org/) format:

- `feat:` — new analysis or feature
- `fix:` — bug fix
- `docs:` — documentation only
- `refactor:` — code restructuring without behavior change
- `chore:` — maintenance (dependency updates, CI config)

### Pull Requests

- **One logical change per PR**
- **Code review via PRs, experiment discussion via Issues**
- **External contributors use fork-and-branch**

## Data Management Standards

### Core Principles

1. **Pipeline manages foundation data**: `snakemake` processes raw/external → interim
2. **Analysts work in processed/**: All personal analysis outputs go here
3. **Data flows one way**: raw/external → interim → processed
4. **Never edit upstream directories**: They're managed by the pipeline
5. **Maintainers coordinate updates**: Pipeline changes require coordination

### Adding New Data Sources (Maintainers Only)

**Decision Tree:**

- **External URLs/APIs** → Create downloaders in `src/<PROJECT_NAME>/downloading/`
- **S3 data** → Use s5cmd for listing, rclone for download
- **All sources** → Integrate with Snakemake rules if part of pipeline

#### Unified Data Download Pattern

Create `src/<PROJECT_NAME>/downloading/download_data.py` to fetch both external reference data and profile files using Pooch with SHA256 verification:

```python
import pooch
from pathlib import Path
from loguru import logger

EXTERNAL_DIR = Path("data/external")
PROFILES_DIR = Path("data/raw/profiles")

# External files with SHA256 hashes for integrity
EXTERNAL_FILES = {
    "https://github.com/jump-cellpainting/datasets/raw/main/metadata/compound.csv.gz": (
        "compound.csv.gz",
        "8885960e92ebd99eb33699a79129f517e668d78dd94f0d7478d39c9825bd3c0a",
    ),
    # Add more URLs with (filename, hash) tuples
}

# Profile files from CellPainting Gallery
PROFILE_FILES = {
    "https://cellpainting-gallery.s3.amazonaws.com/.../profiles.parquet": (
        "profiles.parquet",
        "fd54e2f1d7113e511e261715932f1efbd5a52d31d34fe6579a87c49f57682b85",
    ),
}

MANUAL_FILES = ["manual_data.xlsx"]  # Files that need manual download

def main():
    # Download external files with hash verification
    EXTERNAL_DIR.mkdir(parents=True, exist_ok=True)
    for url, (filename, hash_value) in EXTERNAL_FILES.items():
        logger.info(f"Downloading {filename}")
        pooch.retrieve(url=url, known_hash=hash_value, path=EXTERNAL_DIR, fname=filename)

    # Download profile files
    PROFILES_DIR.mkdir(parents=True, exist_ok=True)
    for url, (filename, hash_value) in PROFILE_FILES.items():
        pooch.retrieve(url=url, known_hash=hash_value, path=PROFILES_DIR, fname=filename)

    for filename in MANUAL_FILES:
        if not (EXTERNAL_DIR / filename).exists():
            logger.warning(f"{filename} missing - add manually")

if __name__ == "__main__":
    main()
```

Run downloads:

```bash
# Download all data from original sources (admin task)
pixi run python -m <PROJECT_NAME>.downloading.download_data

# Or use the Justfile
just get-from-sources  # Downloads from original sources
just put-inputs       # Uploads to team S3 (admin only)
just get-inputs       # Downloads from team S3 (daily workflow)
```

#### Data Management Workflow

**Admin (one-time setup):**

```bash
just get-from-sources  # Download from original sources
just put-inputs        # Upload to team S3 for sharing
```

**Team (daily workflow):**

```bash
just get-inputs        # Download inputs from team S3
just run               # Run pipeline
just put-results       # Share results to team S3
```

The team S3 bucket becomes the single source of truth, eliminating metadata warnings and simplifying data management.

### Pipeline Management (Maintainers Only)

```bash
# Check what would run (dry run)
just dry
# or: pixi run snakemake --dry-run --cores 4 --scheduler greedy

# Get latest input data from team S3
just get-inputs

# Run full pipeline
just run
# or: pixi run snakemake --cores 4 --printshellcmds --scheduler greedy

# Push all results to team S3
just put-results

# Commit changes
git add .
git commit -m "fix: update pipeline with new data"
git push
```

**Development shortcut** — when iterating on annotations or config without re-running expensive downstream jobs:

```bash
# Rebuild database, then touch downstream outputs to prevent re-runs
pixi run snakemake augment_metadata_db --cores 4
pixi run snakemake --touch data/processed/expensive-output/.complete
```

**Coordination Protocol:**

1. Announce in Slack before pipeline updates
2. Ensure no active analyses in progress
3. Document changes in commit message
4. Notify team when complete

## Appendix

### Justfile Standard Recipes

Beyond the project-specific S3 paths, include these standard recipes:

```just
# ==================== MAIN WORKFLOW ====================

# Run full pipeline
run:
    @echo "Running pipeline with {{CORES}} cores..."
    @echo "Note: Run 'just get-inputs' first if you haven't downloaded the input data"
    @{{SNAKEMAKE}} all --cores {{CORES}} --printshellcmds --scheduler greedy

# Preview what will run without executing
dry:
    @echo "Performing dry run..."
    @{{SNAKEMAKE}} --cores {{CORES}} -n -p --scheduler greedy

# ==================== DATA SYNC ====================

# Get input data from team S3 (syncs local to match S3 exactly)
get-inputs:
    @echo "Getting input data from team S3..."
    @mkdir -p {{EXTERNAL_DIR}} {{RAW_DIR}}/profiles
    @echo "Syncing external/..."
    AWS_PROFILE={{AWS_PROFILE}} {{RCLONE_SYNC}} ":s3:{{S3_BUCKET}}/{{S3_PROJECT_PATH}}/external/" {{EXTERNAL_DIR}}/
    @echo "Syncing profiles/..."
    AWS_PROFILE={{AWS_PROFILE}} {{RCLONE_SYNC}} ":s3:{{S3_BUCKET}}/{{S3_PROJECT_PATH}}/profiles/" {{RAW_DIR}}/profiles/

# Upload your results to team S3
put-results:
    @echo "Uploading results to team S3..."
    AWS_PROFILE={{AWS_PROFILE}} {{RCLONE_SYNC}} "{{INTERIM_DIR}}/" ":s3:{{S3_BUCKET}}/{{S3_PROJECT_PATH}}/interim/"
    AWS_PROFILE={{AWS_PROFILE}} {{RCLONE_SYNC}} "{{PROCESSED_DIR}}/" ":s3:{{S3_BUCKET}}/{{S3_PROJECT_PATH}}/processed/"

# ==================== CODE QUALITY ====================

# Run snakemake linting (filters conda/container warnings since we use pixi)
snakelint:
    #!/usr/bin/env bash
    echo "Running snakemake lint..."
    output=$({{SNAKEMAKE}} --lint 2>&1)
    filtered=$(echo "$output" | grep -v -E "(Specify a conda environment|https://snakemake.readthedocs.io|This way, the used software|workflow can be executed on any machine|Also see:$)")
    echo "$filtered" | grep -v -E "^Lints for rule.*:$" | grep -v "^[[:space:]]*$" || echo "No lint warnings!"

# ==================== UTILITIES ====================

# See pipeline status
status:
    @{{SNAKEMAKE}} --summary

# Show current configuration
config:
    @echo "Current Configuration:"
    @echo "  AWS_PROFILE: {{AWS_PROFILE}}"
    @echo "  S3_BUCKET: {{S3_BUCKET}}"
    @echo "  S3_PREFIX: {{S3_PREFIX}}"
    @echo "  CORES: {{CORES}}"

# List what's in S3
list-s3:
    @{{S5CMD}} ls {{S3_PREFIX}}/ | head -20
```

> [!WARNING]
> `get-inputs` uses `rclone sync` which makes the local directory match S3 exactly — local files not present in S3 will be **deleted**. This is intentional (S3 is the source of truth) but be aware of it.

### Example Projects

- <https://github.com/broadinstitute/jump_production>
- <https://github.com/broadinstitute/2025_04_13_OASIS_CellPainting>
