---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---
Apptainer (formerly known as Singularity) is a free and open-source container platform that allows you to create and run applications in isolated images (also called "containers") in a simple, portable, fast, and secure manner. It performs operating system level virtualization known as containerization. Many container platforms are available, but Apptainer is designed to bring containers and reproducibility to the scientific community and High-Performance Computing (HPC) use cases. Using Apptainer, developers can work in reproducible environments of their choice and design, and these complete environments can be easily copied and executed on other platforms.

This is an introduction to Apptainer/Singularity, its motivations and applications in HEP.

Based on the [Apptainer user guide](https://apptainer.org/docs/).

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
> * Basic knowledge of the Unix Shell, e.g., from the [carpentry course](https://swcarpentry.github.io/shell-novice/).
> * Access to a computing system with Apptainer/Singularity available. It can either be installed locally, or the machine can have user namespaces enabled and access to CVMFS.
{: .prereq}

{% include curriculum.html %}

{% include links.md %}
