---
name: scaffold-project
description: Scaffold a new data science project following lab CCDS conventions
user-invocable: true
disable-model-invocation: true
argument-hint: <project-name> [s3-bucket] [aws-profile]
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion
---

# Scaffold Project

Automate the complete project setup described in workflows.md. This skill orchestrates the steps — **workflows.md is the source of truth for all template content**.

## Arguments

Parse `$ARGUMENTS` as: `<project-name> [s3-bucket] [aws-profile]`

- `project-name` (required): kebab-case project name (e.g., `cellpainting-analysis`)
- `s3-bucket` (optional, default `your-bucket`): S3 bucket name
- `aws-profile` (optional, default `default`): AWS profile name

If `project-name` is missing, ask the user for it.

## Name Derivation

From the kebab-case project name, derive:

- **snake_case**: Replace `-` with `_` (e.g., `cellpainting-analysis` → `cellpainting_analysis`) — used for Python package name
- **UPPER_SNAKE**: Uppercase of snake_case (e.g., `CELLPAINTING_ANALYSIS`) — used for env var prefix in config.py

## Before Starting

**Ask the user**: "Where should I create the project? Provide an absolute path to the parent directory (the project folder will be created inside it)."

Wait for the answer. All subsequent paths are relative to `<target-dir>/<project-name>/`.

## Source of Truth

Read `workflows.md` from this repository before generating any files. The file is at the repo root alongside this skill. Every template, config, and code block below comes from workflows.md — do NOT hardcode content; read it fresh each time.

## Execution Steps

Run these in order. For each step that writes a file, read the corresponding section from workflows.md and substitute `<PROJECT_NAME>` with the snake_case name, `<PROJECT_NAME_UPPER>` with the UPPER_SNAKE name, and S3/AWS values as appropriate.

### 1. Create project directory and init git

Create `<target-dir>/<project-name>/` and run `git init` inside it. All remaining steps operate within this directory.

(Reference: workflows.md "Step 1. Initialize Project")

### 2. Create directory structure

Create all directories and `.gitkeep` / `__init__.py` files.

(Reference: workflows.md "Step 3. Create Directory Structure" — the `mkdir -p` and `touch` commands)

### 3. Initialize pixi

Run `pixi init --format pyproject` then `pixi add python=3.12`.

(Reference: workflows.md "Step 4. Set Up Python Environment" — first code block)

### 4. Write pyproject.toml

Read the `[project]`, `[build-system]`, `[tool.pixi.pypi-dependencies]`, `[dependency-groups]`, and `[tool.pixi.environments]` sections from workflows.md Step 4 **and** the `[tool.ruff]` sections from workflows.md Step 5. Merge these into the `pyproject.toml` that `pixi init` created, preserving any existing `[tool.pixi.dependencies]` block pixi generated. Substitute the project name in `name`, `[tool.pixi.pypi-dependencies]`, and the ruff `src` paths.

(Reference: workflows.md "Step 4" toml block + "Step 5 / Ruff Configuration" toml block)

### 5. Write .gitignore

Run `curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore`, then append two lines: `data/` and `.snakemake/`.

(Reference: workflows.md Step 4, after the pyproject.toml section)

### 6. Write .markdownlint.yaml

Write the markdown lint config from workflows.md Step 5 "Markdown Linting".

(Reference: workflows.md "Step 5 / Markdown Linting" yaml block)

### 7. Write .pre-commit-config.yaml

Write the pre-commit config from workflows.md Step 6.

(Reference: workflows.md "Step 6. Configure Pre-commit Hooks" yaml block)

### 8. Write src/<package>/config.py and gpu.py

Write both files from workflows.md Step 7, substituting `<PROJECT_NAME>` (snake_case) and `<PROJECT_NAME_UPPER>` (UPPER_SNAKE) in the code.

(Reference: workflows.md "Step 7. Create config and GPU modules" — both python blocks)

### 9. Write Snakefile and rules/processing.smk

Write both files from workflows.md Step 8.

(Reference: workflows.md "Step 8. Create Test Pipeline" — both python blocks)

### 10. Write Justfile

Combine the **project configuration header** from workflows.md Step 2 and the **standard recipes** from the Appendix "Justfile Standard Recipes" section into one Justfile. Substitute `S3_BUCKET`, `S3_PROJECT_PATH`, and `AWS_PROFILE` defaults with the user-provided values (or placeholders).

For `S3_PROJECT_PATH`, use `projects/<project-name>/datastore` as the default.

(Reference: workflows.md "Step 2. Configure Storage" just block + "Appendix / Justfile Standard Recipes" just block)

### 11. Install dependencies

Run `pixi install`.

### 12. Install pre-commit hooks

Run `pixi run pre-commit install --hook-type pre-commit --hook-type pre-push`.

### 13. Test pipeline

Run `pixi run snakemake --cores 1 --scheduler greedy` and confirm it succeeds.

### 14. Initial commit

Stage all files with `git add .` and commit: `git commit -m "feat: initial project structure with pipeline"`. If pre-commit hooks modify files, stage again and re-commit.

### 15. Verify

Run `pixi run python -c "from <package>.config import PROJ_ROOT; print(PROJ_ROOT)"` (using the snake_case package name) and confirm it prints the project root path.

## Error Handling

- If any step fails, stop and report the error to the user — do not continue to subsequent steps.
- If `pixi` or `git` is not installed, tell the user to install prerequisites per workflows.md "Prerequisites" section.
