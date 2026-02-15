---
name: scaffold-project
description: Scaffold a new data science project following lab CCDS conventions. Use when the user wants to create a new project from scratch.
user-invokable: true
disable-model-invocation: true
argument-hint: <project-name> [s3-bucket] [aws-profile]
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion
---

# Scaffold Project

Read `workflows.md` (repo root) before generating any files. Extract content from `<!-- scaffold:*:start -->` / `<!-- scaffold:*:end -->` anchor pairs — never hardcode template content.

## Arguments

Parse `$ARGUMENTS` as: `<project-name> [s3-bucket] [aws-profile]`

- `project-name` (required): kebab-case (e.g., `cellpainting-analysis`)
- `s3-bucket` (optional, default `your-bucket`)
- `aws-profile` (optional, default `default`)

If `project-name` is missing, ask. Derive **snake_case** (`cellpainting_analysis`) for the Python package and **UPPER_SNAKE** (`CELLPAINTING_ANALYSIS`) for env var prefixes.

Before starting, ask: "Where should I create the project? Provide an absolute path to the parent directory."

## Steps

Run in order. Stop and report on any failure. For each anchor, substitute `<PROJECT_NAME>` → snake_case, `<PROJECT_NAME_UPPER>` → UPPER_SNAKE, and S3/AWS values as appropriate.

1. Create `<target-dir>/<project-name>/`, `git init`. All remaining steps run inside it. (`scaffold:init-project`)
2. Create directories and `.gitkeep` / `__init__.py` files. (`scaffold:create-dirs`)
3. Run `pixi init --format pyproject` then `pixi add python=3.12`. (`scaffold:pixi-init`)
4. Merge `scaffold:pyproject` + `scaffold:ruff` into the `pyproject.toml` pixi created, preserving the existing `[tool.pixi.dependencies]` block.
5. Download Python gitignore and append data/snakemake exclusions. (`scaffold:gitignore`)
6. Write `.markdownlint.yaml`. (`scaffold:markdownlint`)
7. Write `.pre-commit-config.yaml`. (`scaffold:pre-commit`)
8. Write `src/<package>/config.py` and `gpu.py`. (`scaffold:config-py` + `scaffold:gpu-py`)
9. Write `Snakefile` and `rules/processing.smk`. (`scaffold:snakefile` + `scaffold:processing-smk`)
10. Combine `scaffold:configure-storage` + `scaffold:justfile-recipes` into one Justfile. Default `S3_PROJECT_PATH` to `projects/<project-name>/datastore`.
11. `pixi install`
12. `pixi run pre-commit install --hook-type pre-commit --hook-type pre-push`
13. `pixi run snakemake --cores 1 --scheduler greedy` — confirm success.
14. `git add . && git commit -m "feat: initial project structure with pipeline"` — if hooks modify files, stage again and re-commit.
15. `pixi run python -c "from <package>.config import PROJ_ROOT; print(PROJ_ROOT)"` — confirm it prints the project root.
