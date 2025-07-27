# Project Workflows

## How We Organize Projects

We follow [Cookiecutter Data Science](https://cookiecutter-data-science.drivendata.org/) principles with specific adaptations for our lab's needs.

### Core Principles

1. **Data flows in one direction**: `raw/` + `external/` → `interim/` → `processed/`
2. **Raw data is immutable**: Never edit raw data directly
3. **Clear separation of concerns**: Pipeline-managed data vs. personal analysis
4. **Everything is reproducible**: Code + raw data = any output

### Directory Structure

```text
project-name/
├── data/
│   ├── raw/          # Original, immutable data
│   ├── external/     # Third-party reference data
│   ├── interim/      # Pipeline-generated intermediate data
│   └── processed/    # Your analysis outputs (your workspace)
├── {project_name}/   # Python package with processing code
├── notebooks/        # Numbered analysis notebooks
├── scripts/          # Shell scripts for data import
└── docs/            # Documentation
```

### Experiment Tracking with GitHub Issues

**Issues are our primary method for tracking experiments and discussions.**

- **Title deliberately**: Start with hypothesis/question, update to conclusion when complete
- **Paste figures and results**: Include plots and analysis snippets directly in issues
- **Link everything**: Reference commits, notebooks, and data locations
- **Use labels**: `experiment`, `analysis`, `results`, etc.

Example issue progression:

1. Title: "Does batch correction improve compound clustering?"
2. Document approach, link to notebook `03.batch-correction-analysis.ipynb`
3. Paste key figures showing results
4. Update title: "PCA-based batch correction improves compound clustering by 15%"

### Notebook Organization

Notebooks follow the naming convention: `<phase>.<sequence>-<initials>-<description>.ipynb`

- **Phase numbers**:
    - `0`: Exploration
    - `1`: Data cleaning/feature engineering
    - `2`: Visualization
    - `3`: Modeling
    - `4`: Publication figures
- **Example**: `1.03-srs-merge-annotations.ipynb`

Within each analysis module, maintain:

- `input/`: Links or references to source data
- `output/`: Intermediate results
- `figures/`: Publication-ready visualizations

## Project Setup

This section covers creating a new project from scratch. Most team members will join existing projects and can skip to [Daily Workflow](#daily-workflow).

### Prerequisites

- Git
- Python 3.12+
- [uv](https://github.com/astral-sh/uv) package manager
- Cloud CLI tools configured (AWS CLI, Azure CLI, or gcloud)
- Access to cloud storage for DVC remote

### Complete Setup Process

#### 1. Initialize Project

```bash
# Create and enter project directory
mkdir {PROJECT_NAME}
cd {PROJECT_NAME}

# Initialize Git and DVC
git init
dvc init

# Configure DVC to automatically stage .dvc files
dvc config core.autostage true
```

#### 2. Configure Remote Storage

```bash
# Add default remote storage
dvc remote add -d {REMOTE_NAME} {REMOTE_URL}
# Example: dvc remote add -d my-s3 s3://my-bucket/dvc

# If using AWS profiles or cloud-specific authentication
dvc remote modify --local {REMOTE_NAME} profile {CLOUD_PROFILE}
# Example: dvc remote modify --local my-s3 profile imaging-platform
```

**Note**: The `--local` flag stores credentials in `.dvc/config.local` which is gitignored.

#### 3. Create Directory Structure

```bash
# Create all directories
mkdir -p data/{raw,external,interim,processed}
mkdir -p {PROJECT_NAME} notebooks scripts tests
mkdir -p docs/docs references

# Add .gitkeep files to track empty directories
touch data/{raw,external,interim,processed}/.gitkeep
touch notebooks/.gitkeep references/.gitkeep
```

#### 4. Set Up Python Environment

```bash
# Initialize Python project
uv init --name {PROJECT_NAME} --package

# Add core dependencies
uv add loguru typer "dvc[s3]"
uv add awscli  # or azure-cli, google-cloud-storage

# Add development dependencies
uv add --group lint ruff pre-commit
uv add --group test pytest pytest-cov
```

#### 5. Configure Pre-commit Hooks

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
        exclude: \.dvc$
      - id: check-added-large-files
        args: [--maxkb=10240]
      - id: check-yaml
      - id: end-of-file-fixer
        exclude: \.dvc$

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.9.1
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

Install hooks (order matters!):

```bash
# CRITICAL: Install DVC hooks first
dvc install --use-pre-commit-tool

# Then install pre-commit hooks
pre-commit install --hook-type pre-commit --hook-type pre-push --hook-type post-checkout
```

#### 6. Create Test Pipeline

Create `dvc.yaml` to verify setup:

```yaml
stages:
  prepare_data:
    cmd: echo "Preparing data..." && echo "test" > data/interim/test.txt
    outs:
      - data/interim/test.txt
```

Test it:

```bash
dvc repro
git add .
git commit -m "Initial project structure with DVC pipeline"
dvc push
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

### Start Your Day

```bash
# 1. Get code updates
git pull

# 2. Get latest data
dvc pull -R data/external data/interim
# Note: Skip data/raw/ if files are large

# 3. Check pipeline status
dvc repro --dry
# If it shows stages would run: dvc repro --pull
```

### Working with Large Datasets

For selective downloads:

```bash
# Download specific files
dvc pull data/raw/specific-file.parquet.dvc

# Download by pattern
dvc pull data/external/*.dvc

# Check what's missing locally
dvc status
```

**Note**: Files showing as "deleted" in `dvc status` are just not downloaded locally - they're safely tracked in remote storage.

### Running Your Analysis

1. **Create/open notebook**: `notebooks/1.01-abc-analysis-name.ipynb`
2. **Load data**: Read from `data/interim/`
3. **Save outputs**: Write to `data/processed/your-analysis/`
4. **Track experiments**: Create GitHub issue with hypothesis, link notebook, paste key figures

### Sharing Your Work

```bash
# Track your outputs
dvc add data/processed/your-analysis/

# Commit everything
git add notebooks/your-notebook.ipynb data/processed/your-analysis.dvc
git commit -m "Add analysis of X showing Y"

# Push (data sync happens automatically via hooks)
git push
```

### Pull Request Guidelines

- **Keep PRs small and focused** - one logical change per PR
- **Use PRs for code review**, Issues for experiment discussion
- **Fork-and-branch for external contributors**

## Data Management Standards

### Core Principles

1. **Pipeline manages foundation data**: `dvc repro` processes raw/external → interim
2. **Analysts work in processed/**: All personal analysis outputs go here
3. **Data flows one way**: raw/external → interim → processed
4. **Never edit upstream directories**: They're managed by the pipeline
5. **Maintainers coordinate updates**: Pipeline changes require coordination

### Adding New Data Sources (Maintainers Only)

**Decision Tree:**

- **Static URLs** (S3, HTTP) → Use shell scripts
- **Dynamic sources** (APIs, auth required) → Create Python scripts + DVC stages

#### Static URL Import Pattern

Create/update `scripts/setup_external_data.sh`:

```bash
#!/bin/bash
# For reference data (→ data/external/)
[ ! -f data/external/metadata.csv.dvc ] && \
dvc import-url --to-remote https://example.com/metadata.csv data/external/

# For primary datasets (→ data/raw/)
[ ! -f data/raw/dataset.parquet.dvc ] && \
dvc import-url --version-aware s3://bucket/dataset.parquet data/raw/
```

**Notes:**

- `--to-remote`: Skips local download, sends directly to DVC remote
- `--version-aware`: Enables updating when source changes
- Files show as "deleted" in `dvc status` until explicitly pulled

#### Dynamic Source Pattern

1. Create downloader: `{project_name}/downloading/fetch_api_data.py`
2. Add to `dvc.yaml`:

```yaml
stages:
  fetch_api_data:
    cmd: uv run python -m {project_name}.downloading.fetch_api_data
    deps:
      - {project_name}/downloading/fetch_api_data.py
    outs:
      - data/external/api_data.json
```

### Pipeline Management (Maintainers Only)

```bash
# Check for upstream changes
dvc status

# Update specific imported file
dvc update data/external/metadata.csv.dvc

# Run full pipeline
dvc repro

# Push all changes
dvc push
git add .
git commit -m "Update pipeline with new data"
git push
```

**Coordination Protocol:**

1. Announce in Slack before pipeline updates
2. Ensure no active analyses in progress
3. Document changes in commit message
4. Notify team when complete

## Appendix: Complete Examples

### Example Projects

Study these repositories to see our workflow in practice:

- [jump-cellpainting/pilot-cpjump1-analysis](https://github.com/jump-cellpainting/pilot-cpjump1-analysis/) - Full CCDS structure
- [broadinstitute/cell-health](https://github.com/broadinstitute/cell-health) - Notebook-focused analysis
- [jump-cellpainting/morphmap](https://github.com/jump-cellpainting/morphmap) - Complex pipeline example

### Common Commands Reference

```bash
# First time setup
git clone <repo> && cd <repo>
uv sync
dvc pull -R data/external data/interim

# Daily sync
git pull && dvc pull -R data/external data/interim

# Create analysis
jupyter notebook notebooks/
# Save outputs to data/processed/

# Share work
dvc add data/processed/my-analysis/
git add -A
git commit -m "Add analysis showing X"
git push

# Check data status
dvc status          # Local vs tracked
dvc status -c       # Cache vs remote
dvc diff            # See changes
```

### Troubleshooting

#### Pipeline is not up to date

- If you're an analyst: Wait for maintainer to update
- If you're a maintainer: Run `dvc repro` and push changes

#### DVC pull is slow

- Pull only what you need: `dvc pull data/interim/specific-file`
- Skip large raw files unless necessary

#### Can't find my data

- Check you're reading from `data/interim/`, not `data/raw/`
- Ensure you've run `dvc pull` for the files you need

## Documentation Guidelines

When updating this document, follow these principles:

### Target Audience

- **Expert data scientists** who are new to the lab
- They will be instructed to read this document thoroughly
- They want to understand THE way this lab operates, not explore options
- Assume technical competence - skip basic explanations

### Writing Style

- **Prescriptive over descriptive**: "Do X" not "You might consider X"
- **Dense over verbose**: One clear statement beats three explanatory paragraphs
- **Executable over abstract**: Every process needs exact, working commands
- **Real over generic**: Use actual filenames from real projects, not placeholders

### Content Organization

- **Role-based sections**: Clearly separate what analysts vs maintainers need
- **Most-used first**: Daily workflow before one-time setup
- **Progressive disclosure**: Core workflow in main text, edge cases in appendix
- **Decision trees**: "If X → do Y" for any branching logic

### Critical Elements to Include

- **Coordination requirements**: Bold warnings when team sync is needed
- **Order dependencies**: Explicit when sequence matters (e.g., hook installation)
- **Known gotchas**: Document issues with specific workarounds
- **The "why" sparingly**: Only when it affects implementation choices

### What to Avoid

- Beginner explanations ("Git is a version control system...")
- Multiple options without clear guidance
- Philosophical discussions about methodology
- Untested command sequences

Remember: This document is the authoritative reference that turns an expert data scientist into an expert lab member.
