# Resources

> [!NOTE]
>
> - 🔒 = Broad Institute access required
> - 🟢 = Current and recommended (2020-2025 papers; actively maintained courses/tools)
> - 🟡 = Foundational but older (still valuable, supplement with modern content)
> - ⚠️ = Unvetted content ([see context](https://github.com/broadinstitute/carpenter-singh-lab-guide/pull/13#issuecomment-3386612560))
> - Last updated: October 2025

## Internal Resources

- Internal paper PDFs: [Google Drive folder](https://drive.google.com/drive/u/0/folders/1OWH25SMhGiXc_TGIhIEc2lxRRX-99Vxr) 🔒
- Biology 101: [Internal wiki resources](https://new.ipwiki.app/orientation_to_the_imaging_platform#biology) 🔒
- Slack discussion on data analysis: [discussion](https://broadinstitute.slack.com/archives/G01EEQFNZD0/p1658843416312999) 🔒

## Foundations

### Command Line & Programming

- [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) 🟢 - Essential tools for shell, Git, Vim, debugging (2020 lectures remain current for fundamentals)
- [Software Carpentry](https://software-carpentry.org/) 🟢 - Evidence-based programming workshops for scientists, actively maintained with lessons through 2025
- [Rosalind](http://rosalind.info/) ⚠️ - Interactive problem-solving platform for bioinformatics

### Statistics

- [All of Statistics](https://egrcc.github.io/docs/math/all-of-statistics.pdf) 🟡 - Classic compact book on statistics

### Biology for Computational Scientists

#### Core Texts

- [A Computer Scientist's Guide to Cell Biology (2nd Edition, 2024)](https://link.springer.com/book/10.1007/978-3-031-55907-5) 🟢 - Updated with CRISPR, NGS, mRNA vaccines, COVID-19 developments
- [A Primer for Computational Biology](https://open.oregonstate.education/computationalbiology/) 🟡 - Useful for Unix/command line basics, but Python content is dated (archived 2019)
- [Nature Biotechnology Primers](https://www.nature.com/collections/tmdlscdqmc) ⚠️ - Continuously updated series covering ML for biology, single-cell methods, spatial transcriptomics, CRISPR

#### Video Courses

- [Introduction to Biology: The Secret of Life](https://courses.edx.org/courses/course-v1:MITx+7.00x+3T2019/course/) 🟡 - Eric Lander's MIT course, excellent for biochemistry fundamentals but supplement with post-2019 advances (single-cell, spatial transcriptomics, AlphaFold)

## Image-Based Profiling

### Key Papers & Reviews

- [Cell Painting: a decade of discovery and innovation in cellular imaging](https://www.nature.com/articles/s41592-024-02528-8) 🟢 - 2024 systematic review of Cell Painting
- [Progress and new challenges in image-based profiling](https://arxiv.org/abs/2508.00097) 🟢 - August 2025 comprehensive review
- [Image-based profiling: due for a machine-learning upgrade?](https://www.nature.com/articles/s41573-020-00117-w) 🟡 - 2020 review of ML applications in image-based profiling
- [Data-analysis strategies for image-based cell profiling](https://www.nature.com/articles/nmeth.4397) 🟡 - 2016 Nature Methods, introduces key analysis steps
- [Applications in image-based profiling of perturbations](https://www.sciencedirect.com/science/article/pii/S0958166916301112) 🟡 - 2016 review describing common applications

### Protocols & Resources

- [Cell Painting Protocol Version 3 (2023)](https://www.nature.com/articles/s41596-023-00840-9) and [wiki](http://broad.io/CellPaintingWiki) 🟢 - Current gold standard protocol, quantitatively optimized
- [JUMP Cell Painting Consortium](https://jump-cellpainting.broadinstitute.org/) 🟢 - Largest public Cell Painting dataset with extensive documentation
- [Cytomining Profiling Handbook](https://cytomining.github.io/) 🟢 - Data processing handbook for image-based profiling
- [Interpreting image-based profiles](https://carpenter-singh-lab.broadinstitute.org/blog/help-interpreting-image-based-profiles) 🟢 - Blog post with practical guidance

### Software & Tools

- [Pycytominer](https://github.com/cytomining/pycytominer) 🟢 - Python package for reproducible image-based profiling (Nature Methods 2025)
- [copairs](https://github.com/cytomining/copairs) 🟢 - Python package for evaluating profile strength and similarity (Nature Comm 2025)

### Courses & Workshops

- [DSCI 2021 Lecture Slides](https://assafzar.wixsite.com/dsci2021/lecture-slides) 🟡 - Comprehensive lecture series (check for more recent alternatives)

## Image Analysis

### Core Learning Resources

- [Introduction to Bioimage Analysis](https://bioimagebook.github.io/README.html) 🟢 - Pete Bankhead's excellent interactive book, actively maintained
- [Creating and troubleshooting microscopy analysis workflows](https://arxiv.org/abs/2403.04520) 🟢 - 2024 short paper on common challenges

### Video Courses

- [iBiology Bioimage Analysis Course](https://www.ibiology.org/online-biology-courses/bioimage-analysis-course/) 🟡 - Complete archived series, valuable foundations despite no new content since 2020
- [iBiology Short Microscopy Series](https://www.ibiology.org/online-biology-courses/short-microscopy-series/) 🟡 - Intro to fluorescence microscopy, optical sectioning, cameras - fundamentals remain valid
- [NEUBIAS Academy YouTube](https://www.youtube.com/@neubias) ⚠️ - Active 2024-2025 webinars on modern tools (ZeroCostDL4Mic, ImJoy, TrackMate)
- [I2K Conference YouTube](https://www.youtube.com/@I2KConference) 🟢 - Virtual I2K 2024 materials posted October 2024, premier bioimage analysis community
- [COBA Center YouTube](https://www.youtube.com/@cobacenter) 🟢 - Active NIH-funded content on CellProfiler, ImageJ, Piximi

## Machine Learning & Deep Learning

### Foundations & Best Practices

- [R2D3 Visual Introduction to Machine Learning](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/) 🟡 - Outstanding interactive visualizations of decision trees and bias-variance tradeoff
- [StatQuest with Josh Starmer (YouTube)](https://www.youtube.com/@statquest) ⚠️ - Clear explanations of ML concepts including PCA, regularization, neural networks
- [3Blue1Brown: Neural Networks](https://www.3blue1brown.com/topics/neural-networks) 🟢 - Original 2017 series on neural networks, backpropagation, PLUS 2024 content on GPT, attention mechanisms, and transformers

### General Courses

- [UC Berkeley Data8: Foundations of Data Science](http://data8.org/) 🟢 - Active through 2025, Python/Jupyter notebooks, accessible for beginners
- [Fast.ai: Practical Deep Learning](https://course.fast.ai/) ⚠️ - Updated 2024, top-down teaching approach
- [DeepLearning.AI Specialization](https://www.deeplearning.ai/courses/deep-learning-specialization/) 🟢 - Andrew Ng's updated 2022+ courses
- [Stanford CS231n: Deep Learning for Computer Vision](https://cs231n.stanford.edu/) 🟢 - Spring 2025, actively maintained with PyTorch, CNNs, and transformers
- [MIT 6.S191: Introduction to Deep Learning](http://introtodeeplearning.com/) ⚠️ - Updated annually, 2024-2025 materials including transformers and modern architectures

### Modern Architectures

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) 🟢 - Best visual explanation of transformer architecture
- [Stanford CS25: Transformers United](https://web.stanford.edu/class/cs25/) ⚠️ - Dedicated course on transformer architectures and applications
- [Hugging Face Course](https://huggingface.co/learn/nlp-course/chapter1/1) ⚠️ - Transformers, LLMs, and modern NLP/foundation models
- [Attention is All You Need](https://arxiv.org/abs/1706.03762) 🟢 - Vaswani et al. 2017, the original transformer paper
- [The Annotated Transformer](http://nlp.seas.harvard.edu/annotated-transformer/) 🟢 - Line-by-line PyTorch implementation with explanations

### Textbooks & Reference

- [Understanding Deep Learning](https://udlbook.github.io/udlbook/) ⚠️ - Simon Prince 2023, free online with Jupyter notebooks, covers CNNs, RNNs, transformers, and diffusion models
- [The Little Book of Deep Learning](https://fleuret.org/public/lbdl.pdf) 🟢 - François Fleuret 2024, concise 140 pages explicitly covering attention, transformers, diffusion models
- [Dive into Deep Learning (d2l.ai)](http://www.d2l.ai/) ⚠️ - Interactive textbook with chapters on CNNs, RNNs, attention mechanisms, and transformers
- [Stanford CS231n Notes](http://cs231n.github.io) 🟢 - Comprehensive notes on neural networks and architectures
- [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/) 🟡 - Michael Nielsen's book, best for understanding backpropagation mechanics (pre-transformer era)
- [Deep Learning](http://www.deeplearningbook.org) 🟡 - Goodfellow, Bengio, Courville - excellent mathematical foundations, supplement Part III with post-2017 content
- [colah's blog](http://colah.github.io) 🟡 - Excellent for LSTM/RNN understanding and convolution fundamentals (2015, pre-transformer)

### Biology Applications

- [MIT 6.874: Deep Learning in the Life Sciences](https://mit6874.github.io/) ⚠️ - Genomics, protein structure, drug discovery applications
- [A guide to machine learning for biologists](https://www.nature.com/articles/s41580-021-00407-0) 🟢 - Nature Reviews 2021
- [AlphaFold 3 Paper](https://www.nature.com/articles/s41586-024-07487-w) ⚠️ - Nature 2024, protein-DNA/RNA/ligand interactions (Nobel Prize Chemistry 2024)
- [Deep Learning for Computational Biology](https://www.embopress.org/doi/full/10.15252/msb.20156651) 🟡 - 2016 review, foundational overview

## AI in Drug Discovery & Biology

> ![NOTE]
> Contributions are welcome!

## Community & Staying Current

> ![NOTE]
> We do not use the 🟢 🟡 markers in this section because everything here is expected to be current (except a separate archives section)

### Podcasts

#### Active Podcasts

- [Raising Health](https://podcasts.apple.com/us/podcast/raising-health/id1529318900) - a16z Bio + Health, 185 episodes through August 2025, premier biotech/computational biology podcast
- [Ground Truths](https://podcasts.apple.com/us/podcast/ground-truths/id1728526108) - Eric Topol on AI in healthcare, genomics, longevity (2025 active)
- [NEJM AI Grand Rounds](https://ai-podcast.nejm.org/) ⚠️ - Harvard Medical School's official AI in medicine podcast (2025 active)
- [AI For Pharma Growth](https://podcasts.apple.com/us/podcast/ai-for-pharma-growth/id1616728442) ⚠️ - Weekly updates on pharma/biotech AI implementation
- [Artificial Intelligence in Drug Discovery](https://open.spotify.com/show/2feFcQR2ZnM9wKhtsMev30) ⚠️ - ML/AI in drug development focus

#### Discontinued But Valuable Archives

- [The Data Pulse](https://podcasts.apple.com/us/podcast/the-data-pulse/id1524299727) - 25 episodes featuring Aviv Regev, Isaac Kohane (ended December 2020)
- [The AI Health Podcast](https://podcasts.apple.com/us/podcast/the-ai-health-podcast/id1542731019) - 41 episodes with excellent guests (ended July 2022)
- [Brave New Planet](https://podcasts.apple.com/us/podcast/brave-new-planet/id1531898121) - Eric Lander's 7-part limited series on AI ethics and technology (2020)

### Newsletters & Blogs

#### Tier 1: Essential

- [Decoding Bio](https://www.decodingbio.com/) ⚠️ - Weekly BioByte newsletter, 10,000+ subscribers, community-led AI×bio translations
- [In the Pipeline](https://www.science.org/blogs/pipeline) - Derek Lowe's legendary 23-year pharma/drug discovery blog (daily, October 2025 active)
- [Century of Bio](https://centuryofbio.com/) - Elliot Hershberg's best-in-class deep dives on biotech companies and technologies

#### Tier 2: Highly Recommended

- [Where Tech Meets Bio](https://www.techlifesci.com/subscribe) - Weekly AI/biotech/drug discovery trends by Andrii Buvailo
- [Scaling Biotech](https://scalingbiotech.substack.com/) - Jesse Johnson on AI/ML integration into biotech
- [Alex Telford's Blog](https://atelfo.github.io/) ⚠️ - Strategic analysis of biotech industry, innovation, business models

#### Tier 3: Specialized

- [Owl Posting](https://www.owlposting.com/) - Technical research updates on computational biology and ML (Abhishaike Mahajan)
- [Code to Cure](https://wyss.harvard.edu/news/from-data-to-drugs-the-role-of-artificial-intelligence-in-drug-discovery/) ⚠️ - Wyss Institute on AI×biology×healthcare

### Additional Resources

- [AI/ML Cheatsheets](https://github.com/kailashahirwar/cheatsheets-ai) 🟢 - Quick reference for various packages
- Reddit communities: r/MachineLearning, r/bioinformatics, r/computational_biology - for real-time discussions and tool recommendations
