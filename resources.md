# Resources

> [!NOTE]
> Some links in this document require Broad Institute access. These are marked with 🔒.

## Internal Resources

- Internal paper PDFs: [Google Drive folder](https://drive.google.com/drive/u/0/folders/1OWH25SMhGiXc_TGIhIEc2lxRRX-99Vxr) 🔒
- Biology 101: [Internal wiki resources](https://new.ipwiki.app/orientation_to_the_imaging_platform#biology) 🔒
- Slack discussion on data analysis: [discussion](https://broadinstitute.slack.com/archives/G01EEQFNZD0/p1658843416312999) 🔒

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
- [Cell Painting protocol](https://www.nature.com/articles/s41596-023-00840-9) and [wiki](http://broad.io/CellPaintingWiki).

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

## Learning Paths

> [!NOTE]
> These resources were primarily curated 2015-2021 and may be outdated.
> They remain useful for foundational concepts, but verify current best practices before deep diving. 
> Updates welcome via PR.

Curated educational resources organized by topic. These complement the scientific literature above and are designed to help lab members build foundational knowledge.

### Image-based Profiling

#### Understanding Profiles

- [Interpreting image-based profiles](https://carpenter-singh-lab.broadinstitute.org/blog/help-interpreting-image-based-profiles) - Blog post with practical guidance
- [Profile interpretation strategies](https://zenodo.org/record/7348094#.Y34QIuzMKrN) - Protocol preprint

#### Course Materials

- [DSCI 2021 Lecture Slides](https://assafzar.wixsite.com/dsci2021/lecture-slides) - Comprehensive lecture series (check for updates)

### Image Analysis

#### Getting Started

- [Creating and troubleshooting microscopy analysis workflows](https://arxiv.org/abs/2403.04520) - Short paper on common challenges and solutions
- [Introduction to Bioimage Analysis](https://bioimagebook.github.io/README.html) - Comprehensive online book by Pete Bankhead

#### Video Courses

- [iBiology Bioimage Analysis Course](https://www.ibiology.org/online-biology-courses/bioimage-analysis-course/) - Complete video series
- [iBiology Short Microscopy Series](https://www.ibiology.org/online-biology-courses/short-microscopy-series/) - Recommended: Intro to fluorescence microscopy, Optical sectioning and confocal microscopy, Cameras and digital image analysis
- [Microcourses YouTube Channel](https://www.youtube.com/c/Microcourses/videos)
- [I2K Conference YouTube](https://www.youtube.com/@I2KConference)
- [COBA Center YouTube](https://www.youtube.com/@cobacenter)

### Programming & Command Line

#### Command Line

- [Command Line Bootcamp](http://command-line-bootcamp.wurmlab.com/) - Interactive tutorial
- [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) - Command line, version control, and essential tools

#### General Programming

- [Software Carpentry](http://software-carpentry.org/) - Programming tutorials for biologists (Python and more)
- [Udacity](https://www.udacity.com) - Video tutorials with quizzes and coding exercises on various technologies

#### Reference Materials

- [AI/ML Cheatsheets](https://github.com/kailashahirwar/cheatsheets-ai) - Quick reference for various packages

### Biology for Computational Scientists

#### Foundational Texts

- [A Computer Scientist's Guide to Cell Biology](http://www.cs.cmu.edu/~./wcohen/GuideToBiology-sampleChapter-release1.4.pdf) - Tailored for CS backgrounds
- [A Biology Primer for Computer Scientists](http://web.stanford.edu/class/cs173/papers/bioprimer.pdf)

#### Video Courses

- [Introduction to Biology: The Secret of Life](https://courses.edx.org/courses/course-v1:MITx+7.00x+3T2019/course/#block-v1:MITx+7.00x+3T2019+type@chapter+block@Week_1) - Eric Lander's MIT course
- [Tales from the Genome](https://www.youtube.com/playlist?list=PLAwxTw4SYaPmSI2j3EpYmC8_VHE7cP9YT) - Introduction to genetics by 23andMe (limited scope for lab work)

### Statistics

- [Introduction to Modern Statistics](https://openintro-ims.netlify.app/) - Foundational concepts and core principles

### Machine Learning

#### Overview & Best Practices

- [Pitfalls in Machine Learning](https://www.nature.com/articles/d41586-019-02307-y) - Nature article on common mistakes
- [A Few Useful Things to Know about Machine Learning](http://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf) - Pedro Domingos' essential 9-page guide

#### Interactive Courses

- [Udacity Intro to Machine Learning (ud120)](https://www.udacity.com/course/intro-to-machine-learning--ud120) - ~500 short videos with quizzes and in-browser coding. Highly accessible for non-quantitative backgrounds. Estimated 12 hours for videos, 5-10 hours for code projects.
- [R2D3 Visual Introduction](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/) - Visualization-based learning

#### Textbooks & Written Materials

- [Data8 Course Materials](http://data8.org/) - UC Berkeley's "Computational and Inferential Thinking" with interactive Jupyter notebooks. Accessible for non-quantitative backgrounds.

#### Advanced Topics

- [Advanced ML/DL/RL Courses](https://www.reddit.com/r/MachineLearning/comments/fdw0ax/d_advanced_courses_update/) - Reddit thread with curated advanced courses
- [Eigenvectors and Eigenvalues Explained](https://soundcloud.com/edwardoneill/steven-strogatz-on-teaching-eigenvectors-and-eigenvalues) - Short audio explanation

### Deep Learning

#### Introductory Resources (Non-Quantitative Backgrounds)

##### Written Materials

- [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/) - Michael Nielsen's accessible online book with intuitive explanations
- [Deep Learning Nature Review](http://www.nature.com/nature/journal/v521/n7553/abs/nature14539.html) - Broad scientific overview
- [Representation Learning and Deep Learning](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6472238) - Review by Bengio et al. ([talk](https://www.youtube.com/watch?v=9XMlaBoDbY4))
- [Deep Learning for Computational Biology](http://onlinelibrary.wiley.com/doi/10.15252/msb.20156651/abstract)
- [Stanford CS231n Notes](http://cs231n.github.io) - Neural networks course materials
- [Colah's Blog](http://colah.github.io) - Intuitive deep learning explanations
- [Otoro.net Visualizations](http://otoro.net/ml/index.html) - Easy-to-grasp visualizations
- [ConvNetJS](http://cs.stanford.edu/people/karpathy/convnetjs/) - Neural networks in the browser
- [Deep Learning Historic Overview](http://www.sciencedirect.com/science/article/pii/S0893608014002135) - Schmidhuber's perspective
- [NakedTensor Examples](https://github.com/jostmey/NakedTensor/blob/master/tensor.py) - Bare-bones TensorFlow examples (linear regression in 10 lines)
- [Deep Learning Architectures](http://aiplaybook.a16z.com/docs/guides/dl-architectures)
- [Deep Learning Book](https://www.deeplearningbook.org/) - Goodfellow, Bengio, and Courville
- [The Little Book of Deep Learning](https://fleuret.org/public/lbdl.pdf) - Concise overview

##### Video Courses

- [3Blue1Brown Neural Networks Series](https://www.3blue1brown.com/lessons/neural-networks) - Visual, intuitive explanations
- [Intro to Deep Learning/TensorFlow](https://www.youtube.com/watch?v=u4alGiomYP4#action=share) - 1-hour introduction

#### In-Depth Coverage

- [Deep Learning Book](http://www.deeplearningbook.org) - Goodfellow, Bengio, and Courville's comprehensive textbook

#### Tips & Tricks

- [Must Know Tips/Tricks in Deep Neural Networks](http://www.weixiushen.com/project/CNNTricks/CNNTricks.html) - Practical guidance by Xiu-Shen Wei

## Podcasts & Newsletters

### AI + Biology Podcasts

- [The Data Pulse](https://podcasts.apple.com/us/podcast/the-data-pulse/id1524299727) - Features Aviv Regev, Dennis Wall, Peyton Greenside, Andy Beck, Kyle Swanson, Imran Haque, Daphne Koller, Anthony Philippakis
- [The AI Health Podcast](https://podcasts.apple.com/gb/podcast/dr-daphne-koller-of-insitro-on-digital-biology-and/id1542731019) - Features Daphne Koller, Eric Topol, Vijay Pande, Andrew Ng, Zavain Dar, Cynthia Rudin, Blake Wu
- [Raising Health](https://podcasts.apple.com/us/podcast/raising-health/id1529318900) - a16z Bio + Health podcast exploring biotechnology and healthcare innovation (includes Bio Eats World content on genetic testing, cell shape science, rare diseases, genetics of risk, biology of aging, embodied intelligence)
- [Brave New Planet](https://podcasts.apple.com/us/podcast/brave-new-planet/id1531898121) - All episodes recommended
- [Ground Truths](https://podcasts.apple.com/us/podcast/ground-truths/id1728526108) - Eric Topol on biomedical matters

### Newsletters & Blogs

- [Where Tech Meets Bio](https://www.techlifesci.com/subscribe)
- [Decoding Bio](https://www.decodingbio.com/)
- [Scaling Biotech](https://open.substack.com/pub/scalingbiotech) - Jesse Johnson
- [Century of Bio](https://substack.com/@centuryofbio) - Elliot Hershberg
- [Owl Posting](https://www.owlposting.com/)
- [In the Pipeline](https://www.science.org/blogs/pipeline) - Derek Lowe's pharma perspective, frequent AI commentary
- [BiotechBio](https://biotechbio.substack.com/p/deep-learning-intuition-for-biologists) - Jason Steiner, deep learning for biologists
- [Alex Telford's Blog](https://atelfo.github.io/)
- [Simon Willison's Blog](https://simonwillison.net/) - AI and software development

## General Principles

### Troubleshooting

Follow the 30-minute rule: Try to solve issues independently for ~30 minutes before asking for help ([group norm](https://broadinstitute.slack.com/archives/C3PNSTV09/p1621283882195500) 🔒).

1. Search existing resources:
   - Slack archives
   - GitHub issues
   - Lab documentation
   - Official tool/library documentation
2. Try automated help (Google, AI tools with appropriate caution)
3. Post in relevant Slack channels:
   - #ip-it for computing
   - #ip-profiling for data science
   - #ip-dear-computer-scientist for computer science
   - #ip-dear_biologist for biology
4. Explain what you've already tried and where you're stuck
5. Escalate to PIs or senior lab members if still blocked
6. Document your solution for future reference
