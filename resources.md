# Resources

## Educational Resources

### Computing Skills

- [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)
- [The Unix Workbench](https://www.coursera.org/learn/unix)
- [Data Science Cheatsheet](https://github.com/aaronwangy/Data-Science-Cheatsheet)

### Textbooks

- [Data Analysis for the Life Sciences](https://leanpub.com/dataanalysisforthelifesciences)

## Scientific Literature

### Image-based Profiling Essentials

- [Image-based profiling: due for a machine-learning upgrade?](https://www.nature.com/articles/s41573-020-00117-w) - 2020 review of applications in image-based profiling
- [Data-analysis strategies for image-based cell profiling](https://www.nature.com/articles/nmeth.4397) - Introduces key analysis steps
- [Applications in image-based profiling of perturbations](https://www.sciencedirect.com/science/article/pii/S0958166916301112) - Describes common applications
- [Cell Painting: a decade of discovery and innovation in cellular imaging](https://www.nature.com/articles/s41592-024-02528-8) - Systematic review of Cell Painting
- [Cell Painting protocol](https://www.nature.com/articles/s41596-023-00840-9) - Detailed protocol

## Analysis Tools & Libraries

### Profiling Libraries

- **pycytominer**: Core library for image-based profiling ([GitHub](https://github.com/cytomining/pycytominer))
- **copairs**: Evaluation metrics for profiling ([GitHub](https://github.com/cytomining/copairs))
- More tools at [https://github.com/cytomining/](https://github.com/cytomining/)

### Deep Learning

- **DeepProfiler**: Deep learning for image-based profiling ([GitHub](https://github.com/cytomining/DeepProfiler))

### Learning Resources

- [Profiling Handbook](https://cytomining.github.io/profiling-handbook/) - Step-by-step instructions for producing image-based profiles from raw images
- [LINCS Workflow](https://github.com/broadinstitute/lincs-cell-painting/tree/e9737c3e4e4443eb03c2c278a145f12efe255756/profiles#workflow) - Workflow diagram of image-based profiling

## Development Environment Setup

### Package Management

- Install applications using [brew](https://brew.sh/)
- Use [uv](https://github.com/astral-sh/uv) for Python environment management
- Install R following [these instructions](https://github.com/broadinstitute/imaging-configs/blob/master/R-instructions.md)

### Git Configuration

```bash
# Configure shared repositories
git config --global core.sharedRepository true
```

**Best practice**: Avoid [amending](https://stackoverflow.com/questions/253055/how-do-i-push-amended-commit-to-the-remote-git-repository) pushed commits

## General Principles

### Troubleshooting

Follow the 30-minute rule: Try to solve issues independently for ~30 minutes before asking for help.

1. Search existing resources (documentation, issues, archives)
2. Consult official documentation for relevant tools/libraries
3. Consider using AI tools (ChatGPT, Claude) with appropriate caution
4. Google error messages or problem descriptions
5. Post in relevant channels with details of what you've already tried
6. Document your troubleshooting process for future reference
