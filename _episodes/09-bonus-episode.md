---
title: "Bonus Episode: Building and deploying an Apptainer container to Github Packages"
teaching: 40
exercises: 0
questions:
- How to build apptainer container for python packages?
- How to share apptainer images?
objectives:
- To be able to build an apptainer container and share it via GitHub packages
keypoints:
-  Python packages can be installed in apptainer images along with ubuntu packages.
-  It is possible to publish and share apptainer images over github packages.
---

> ## Prerequisites
> For this lesson, you will need,
> * Knowledge of Git [SW Carpentry Git-Novice Lesson](https://swcarpentry.github.io/git-novice/)
> * Knowledge of GitHub CI/CD [HSF Github CI/CD Lesson](https://hsf-training.github.io/hsf-training-cicd-github/)
{: .prereq}

<iframe width="427" height="251" src="https://www.youtube.com/embed/BRRaSsrK7-k" title="Intro to Apptainer/Singularity #7 - Bonus episode" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Apptainer Container for python packages

Python packages can be installed using an Apptainer image. The following example illustrates how to write a definition file for building an image containing python packages.

```text
BootStrap: docker
From: ubuntu:20.04

%post
    apt-get update -y
    apt-get install wget -y
    export DEBIAN_FRONTEND=noninteractive
    apt-get install dpkg-dev cmake g++ gcc binutils libx11-dev libxpm-dev \
    libxft-dev libxext-dev python3 libssl-dev libgsl0-dev libtiff-dev \
    python3-pip -y
    pip3 install numpy
    pip3 install awkward
    pip3 install uproot4
    pip3 install particle
    pip3 install hepunits
    pip3 install matplotlib
    pip3 install hist
    pip3 install mplhep
    pip3 install vector
    pip3 install fastjet
    pip3 install iminuit
```


As we see, several packages are installed.


## Publish Apptainer images with GitHub Packages and share them!

It is possible to publish apptainer images with [GitHub packages](https://github.com/features/packages).
To do so, one needs to use GitHub CI/CD. A step-by-step guide is presented here.

* **Step 1**: Create a GitHub repository and clone it locally.
* **Step 2**: In the empty repository, make a folder called `.github/workflows`. In this folder we will store the file containing the YAML script for a GitHub workflow, named `apptainer-build-deploy.yml` (the name doesn't really matter).
* **Step 3**: In the top directory of your GitHub repository, create a file named `Apptainer`.
* **Step 4**: Copy-paste the content above and add to the Apptainer file. (In principle it is possible to build this image locally, but we will not do that here, as we wish to build it with GitHub CI/CD).
* **Step 5**: In the `apptainer-build-deploy.yml` file, add the following content:

{% raw %}
```text
name: Apptainer Build Deploy

on:
  pull_request:
  push:
    branches: master

jobs:
  build-test-container:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    container:
        image: quay.io/singularity/singularity:v3.8.1
        options: --privileged

    name: Build Container
    steps:

      - name: Check out code for the container builds
        uses: actions/checkout@v2

      - name: Build Container
        run: |
           singularity build container.sif Apptainer

      - name: Login and Deploy Container
        run: |
           echo ${{ secrets.GITHUB_TOKEN }} | singularity remote login -u ${{ secrets.GHCR_USERNAME }} --password-stdin oras://ghcr.io
           singularity push container.sif oras://ghcr.io/${GITHUB_REPOSITORY}:${tag}
```
{% endraw %}

The above script is designed to build and publish a Apptainer image with [GitHub packages](https://github.com/features/packages).


* **Step 6**: Add LICENSE and README as recommended in the [SW Carpentry Git-Novice Lesson](https://swcarpentry.github.io/git-novice/), and then the repository is good to go.
