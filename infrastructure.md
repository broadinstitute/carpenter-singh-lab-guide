# Infrastructure & Computing

> [!NOTE]
> Some links in this document require Broad Institute access. These are marked with 🔒.

## Computing Infrastructure

### Overview

- **Macbook**: For most daily tasks
- **DGX-1 / spirit / oppy servers**: For GPU computing
- **AWS**: For specific project needs (use judiciously)
- **Broad's compute cluster**: For easily parallelizable jobs

### DGX-1 Server (GPU Computing)

- Maintenance details: [https://github.com/broadinstitute/ip-chores/issues/25](https://github.com/broadinstitute/ip-chores/issues/25) 🔒
- Quick start guide: [https://new.ipwiki.app/dgx-1](https://new.ipwiki.app/dgx-1) 🔒

### Spirit/Oppy Server (GPU Computing)

- We are piloting nixOS [http://github.com/leoank/neusis/](http://github.com/leoank/neusis/)

### Broad's Compute Cluster

- Best for: Easily parallelizable jobs without interactivity or cloud data access needs
- Getting started: [https://intranet.broadinstitute.org/node/5811](https://intranet.broadinstitute.org/node/5811) 🔒
- Support: #bits Slack channel
- Also see #uger and #devnull-gpu-discuss Slack channels for discussions

### AWS

- Use cases: Projects requiring interactivity, large-scale cloud data access, or specific GPU needs
- Getting started:
    - Request an account through [CloudLab](https://cloud.nih.gov/resources/cloudlab/) for $500 in free credits with a 90-day limit, which will allow you to experiment freely
    - After initial experimentation, ask Shantanu to set up your project account
- Required reading:
    - Beth's [talk](https://drive.google.com/drive/u/0/folders/1EGnz4o4kBnHtMDjoYccuIyp-FVD-xer6) 🔒 and [slides](https://docs.google.com/presentation/d/1znPS90s9Yw22FrcsRn_Sd2tC_kaXELF48oEC7CoGXw8/edit?usp=sharing) 🔒 on Imaging Platform AWS Use. This is mostly relevant to Cimini Lab's usage of AWS but the concepts are useful to know.
    - [AWS Usage Policy](https://new.ipwiki.app/aws_amazon_web_services) 🔒
- We recommend interacting via [Skypilot](https://docs.skypilot.co/en/latest/docs/index.html); see this [Slack thread](https://broadinstitute.slack.com/archives/C3QFDHXC4/p1714062848412759) 🔒

## Development Environment Setup

### Package Management

- Install applications using [brew](https://brew.sh/)
- Use [uv](https://github.com/astral-sh/uv) for Python environment management
