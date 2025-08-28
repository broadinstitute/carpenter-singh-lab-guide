# Project Workflows

> [!IMPORTANT]
> This workflow is being actively tested in our lab. Expect rough edges and please share feedback!

> [!NOTE]
> We have migrated from DVC to a simpler workflow using Justfile + Snakemake + s5cmd for data management. This provides direct S3 operations without intermediate cache layers.

> [!NOTE]
> Dense documentation - see [README](README.md) for philosophy and links to comprehensive guides.

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
├── <PROJECT_NAME>/   # Python package with processing code
├── notebooks/        # Numbered analysis notebooks
├── scripts/          # Shell scripts for data import
└── docs/            # Documentation
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
- [uv](https://github.com/astral-sh/uv) package manager
- [just](https://github.com/casey/just) command runner
- [s5cmd](https://github.com/peak/s5cmd) for S3 operations
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

```bash
# Copy the Justfile template from:
# https://github.com/broadinstitute/jump_production/blob/main/Justfile
# This provides standard commands: pull, push, run, download-externals, etc.

# Edit the Justfile to set your S3 configuration:
# S3_BUCKET := "your-bucket"
# S3_PROJECT_PATH := "projects/your-project/datastore"

# Ensure AWS profile is configured
# Example: export AWS_PROFILE=broad-imaging
```

#### 3. Create Directory Structure

```bash
# Create all directories
mkdir -p data/{raw,external,interim,processed}
mkdir -p <PROJECT_NAME>
mkdir -p notebooks scripts tests
mkdir -p docs references

# Add .gitkeep files to track empty directories
touch data/{raw,external,interim,processed}/.gitkeep
touch notebooks/.gitkeep references/.gitkeep
touch README.md
```

#### 4. Set Up Python Environment

```bash
# Initialize Python project
uv init --name <PROJECT_NAME> --package

# Add core dependencies
uv add loguru typer python-dotenv pooch
uv add snakemake  # for pipeline execution

# Add typical analysis dependencies
uv add pandas

# Add development dependencies
uv add --group lint ruff pre-commit
uv add --group test pytest
```

Download Python gitignore template

```bash
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore
```

> **Note**: Add `data/` to `.gitignore` to prevent accidentally committing large data files.

#### 5. Configure Code Quality Tools

### Ruff Configuration

Add to your `pyproject.toml`:

Note: Replace `<PROJECT_NAME>` with the actual project name

```toml
[tool.ruff]
line-length = 120
src = ["<PROJECT_NAME>"]
target-version = "py312"
include = ["pyproject.toml", "<PROJECT_NAME>/**/*.py"]

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "W"]
ignore = [
    "E501",   # Line too long (handled by formatter)
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

#### 7. Create config file for project library

Create `<PROJECT_NAME>/config.py` to set up data paths, and do other setup needed for code you may write in the project library.

```py
from pathlib import Path

from dotenv import load_dotenv

# Load environment variables from .env file if it exists
load_dotenv()

# Paths
PROJ_ROOT = Path(__file__).resolve().parents[2]

DATA_DIR = PROJ_ROOT / "data"
RAW_DATA_DIR = DATA_DIR / "raw"
INTERIM_DATA_DIR = DATA_DIR / "interim"
PROCESSED_DATA_DIR = DATA_DIR / "processed"
EXTERNAL_DATA_DIR = DATA_DIR / "external"
```

#### 8. Create Test Pipeline

Create `Snakefile` to verify setup:

```python
rule prepare_data:
    output:
        "data/interim/test.txt"
    shell:
        "echo 'test' > {output}"
```

Test it:

```bash
snakemake --cores 1
git add .
git commit -m "Initial project structure with pipeline"
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
uv sync --all-groups

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
just pull-external  # Get reference data
just pull          # Get processed results

# 3. Check pipeline status
just dry           # Preview what would run
# If it shows work to do: just run
```

### Working with Large Datasets

For selective downloads:

```bash
# Download specific run
just pull-run specific-analysis



# List available data
s5cmd ls s3://bucket/path/
```

### Running Your Analysis

1. **Create notebook**: `notebooks/1.01-abc-analysis-name.py`
2. **Load data**: Read from `data/interim/`
3. **Save outputs**: Write to `data/processed/your-analysis/`
4. **Track experiment**: Create GitHub issue with hypothesis, link notebook, paste key figures

### Sharing Your Work

```bash
# Push your analysis to S3
just push-run your-analysis

# Commit code changes
git add notebooks/your-notebook.py
git commit -m "Add analysis of X showing Y"
git push
```

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

- **External URLs/APIs** → Create downloaders in `<PROJECT_NAME>/downloading/`
- **S3 data** → Use s5cmd directly
- **All sources** → Integrate with Snakemake rules if part of pipeline

#### External Data Import Pattern

Create `<PROJECT_NAME>/downloading/download_externals.py` to fetch reference data using Pooch:

```python
import pooch
from pathlib import Path
from loguru import logger

EXTERNAL_DIR = Path("data/external")

FILES = {
    "https://github.com/jump-cellpainting/datasets/raw/main/metadata/compound.csv.gz": "compound.csv.gz",
    "https://github.com/jump-cellpainting/datasets/raw/main/metadata/plate.csv.gz": "plate.csv.gz",
    # Add more URLs as needed
}

MANUAL_FILES = ["manual_data.xlsx"]  # Files that need manual download

def main():
    EXTERNAL_DIR.mkdir(parents=True, exist_ok=True)

    for url, filename in FILES.items():
        pooch.retrieve(url=url, known_hash=None, path=EXTERNAL_DIR, fname=filename)

    for filename in MANUAL_FILES:
        if not (EXTERNAL_DIR / filename).exists():
            logger.warning(f"{filename} missing - add manually")

if __name__ == "__main__":
    main()
```

Run downloads:

```bash
# Download all external data
uv run python -m <PROJECT_NAME>.downloading.download_externals

# Or use the Justfile
just download-externals
```

#### Integrating Downloads with Pipeline

Add to `Snakefile` for automatic downloads:

```python
rule download_externals:
    output:
        directory("data/external/")
    shell:
        "uv run python -m <PROJECT_NAME>.downloading.download_externals"
```

### Pipeline Management (Maintainers Only)

```bash
# Check what would run (dry run)
just dry
# or: snakemake --dry-run

# Update external data
just download-externals
# or: just pull-external

# Run full pipeline
just run
# or: snakemake --cores 4

# Push all results
just push

# Commit changes
git add .
git commit -m "Update pipeline with new data"
git push
```

**Coordination Protocol:**

1. Announce in Slack before pipeline updates
2. Ensure no active analyses in progress
3. Document changes in commit message
4. Notify team when complete

## Appendix

### Example Projects

<https://github.com/broadinstitute/jump_production>
