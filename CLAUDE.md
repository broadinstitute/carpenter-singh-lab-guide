# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Context

This is a documentation repository for the Carpenter-Singh Lab at the Broad Institute. The primary purpose is to maintain and improve lab standards documentation, particularly around data science workflows using Cookiecutter Data Science (CCDS) principles.

## Key Tasks When Working Here

### 1. Documentation Improvements

- Check for consistency in command examples across documents
- Ensure all code blocks are properly formatted and executable
- Verify that file paths and project names use consistent placeholder syntax (e.g., `{PROJECT_NAME}`)
- Look for outdated dependencies or version numbers that may need updating

### 2. Common Updates

When asked to update documentation:

- Preserve the prescriptive, writing style outlined in the Documentation Guidelines within each markdown
- Focus on executable commands over explanations

### 3. Adding New Sections

If adding new documentation:

- Follow the existing markdown structure and heading hierarchy
- Place new content in the appropriate file (workflows.md for process-related content)
- Use real-world examples from the referenced repositories when possible

### 4. Quality Checks

Before finalizing any changes:

- Ensure all bash commands use proper escaping and line continuations
- Verify that Python/YAML/TOML code blocks have correct syntax
- Check that any new commands follow the established patterns in the document

## Repository Structure

```text
carpenter-singh-lab-standards/
├── README.md          # Main entry point, links to other docs
├── workflows.md       # Comprehensive CCDS workflow guide
└── CLAUDE.md         # This file
```

## When Making Changes

- The workflows.md file is the authoritative source for lab procedures
- Any new documentation should complement, not duplicate, existing content
- Focus on making processes more clear and executable, not more flexible
