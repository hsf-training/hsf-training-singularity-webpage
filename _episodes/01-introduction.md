---
title: "Introduction"
teaching: 10
exercises:
questions:
- "What issues motivated the creation of Apptainer/Singularity?"
- "What are the differences between Docker, Apptainer and Singularity?"
objectives:
- "Learn the design goals behind Apptainer/Singularity."
keypoints:
- "Apptainer/Singularity is a container platform designed by and for scientists."
- "Single-file based container images facilitates the distribution."
- "Secure. User inside the container = user outside."
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/v5WbqtRbH6M?list=PLKZ9c4ONm-VkxWW98Gcn9H6WwykMiqtnF" title="Intro to Apptainer/Singularity #1 - Introduction"  frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Working with containers

Containers are packages of software that encapsulates a system environment. An OS-level virtualization is delivered
in a container, and any program running on it will use the contextualization isolated inside the container. They have
the advantage that you can build a container on any system, your laptop for example, and then execute it anywhere
as far as the platform compatible with the container is available.

Concepts such as reproducibility, preservation, and distribution
are important in the HEP community, and the containers provide a solution totally compatible with such concepts:
* The version of some specific software used to perform an analysis can be preserved in a container with exactly the same
environment used at that time.
* A "legacy software" with binaries only available for an outdated OS can be executed inside a container.
* All the necessary packages to process data can be easily distributed in containers, independently of the operating
system available on the sites.

> ## What about virtual machines?
> Virtual Machines (VMs) provide the same isolation and reproducibility.
> However, they emulate the hardware, so they are computationally heavier to run,
> require bigger files when distributed and are less flexible than containers, that run only what you require to be different.
{: .callout}

# Why Apptainer/Singularity?

Many solutions are available to work with containers, for example [Docker](https://www.docker.com/),
one of the most popular platforms. However, the enterprise-based container frameworks were motivated to provide
_micro-services_, a solution that fits well in the models of the industry, where system administrators with root privilege
install and run applications, each in its own container.
This is not so compatible with the workflow in the High-Performance Computing (HPC) and High Throughput Computing (HTC),
in which usually complex applications run exhaustively using all the available resources and without any special privilege.

Apptainer/Singularity is a container platform created for the HPC/HTC use case. It allows to build and run containers with just
a few steps in most of the cases, and its design presents key concepts for the scientific community:
* Single-file based container images, facilitating the distribution, archiving and sharing.
* Ability to run, and in modern systems also to be installed, without any root daemon or setuid privileges. This makes it safer for large computer centers with shared resources.
* Preserves the permissions in the environment. The user outside the container can be the same user inside.
* Simple integration with resource managers and distributed computing frameworks because it runs as a regular application.

 <a href="https://apptainer.org/docs/user/">
<img src="https://apptainer.org/docs/user/main/_static/logo.png" alt="Apptainer/Singularity" width="220">
</a>

> ## Docker and Apptainer/Singularity
> As you will learn in this training module, Apptainer/Singularity [*can* be used with Docker images](https://apptainer.org/docs/user/main/docker_and_oci.html).
> If you want to learn more about docker, check out [our docker training](https://hsf-training.github.io/hsf-training-docker/index.html).
{: .callout}

# Apptainer vs Singularity
In these lessons you see the name *Apptainer* or *Apptainer/Singularity*, and the command `apptainer`.
As stated in the [move and renaming announcement](https://apptainer.org/news/community-announcement-20211130/), "Singularity IS Apptainer".
Currently there are three products derived from the original Singularity project from 2015:
* *Singularity*: commercial software by [Sylabs](https://sylabs.io/).
* [*SingularityCE*](https://sylabs.io/2022/06/singularityce-is-singularity/): open source Singularity supported by Sylabs.
* *Apptainer*: open source Singularity, recently renamed and hosted by the [Linux Foundation](https://www.linuxfoundation.org/).
As of Fall 2022 all three Apptainer/Singularity versions are compatible and practically the same, but have different roadmaps.
There is hope that in the future they will join forces, but this is not currently the case.
To understand how this came to be you can read the [Singularity history on Wikipedia](https://en.wikipedia.org/wiki/Singularity_%28software%29#History).

We are following Apptainer, the most adopted variation in the scientific community, so we are using the `apptainer` command.
If you are using Singularity or SingularityCE, just replace the command `apptainer` with `singularity` and the
`APPTAINER_` and  `APPTAINERENV_` variable prefixes  with `SINGULARITY_` and  `SINGULARITYENV_`.
 But since its previous version was named Singularity and it
If you have older scripts still using the `singularity` command they will work also in Apptainer because it is providing the `singularity` alias
and [full compatibility with the previous Singularity environment](https://apptainer.org/docs/user/main/singularity_compatibility.html).

# Documentation

The [official Apptainer documentation](https://apptainer.org/docs/) is available online. Contains basic and advanced
usage of Apptainer/Singularity beyond the scope of this training document. Take a look and read the nice
[introduction](https://apptainer.org/docs/user/main/introduction.html), explaining the motivation behind the
creation of Apptainer/Singularity.


{% include links.md %}
