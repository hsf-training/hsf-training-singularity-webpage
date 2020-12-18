---
title: "Introduction"
teaching: 10
exercises: 5
questions:
- "What issues motivated the creation of Singularity?"
objectives:
- "Learn the design goals behind Singularity."
keypoints:
- "Singularity is a container platform designed by and for scientists."
- "Single-file based container images facilitates the distribution."
- "Secure. User inside the container = user outside."
---

# Working with containers

Containers are packages of software that encapsulates a system environment. An OS-level virtualization is delivered
in a container, and any program running on it will use the contextualization isolated inside the container. They have 
the advantage that you can build a container on any system, your laptop for example, and then execute it anywhere
as far as the plataform compatible with the container is available. 

Concepts as reproducibility, preservation, distribution,
are important in the HEP community, and the containers provide a solution totally compatible with such concepts:
* The version of some specific software used to perform an analysis can be preserved in a container with exactly the same
environment used at that time.
* A "legacy software" with binaries only available for an outdated OS can be executed inside a container. 
* All the necessary packages to process data can be easily distributed in containers, independently of the operating
system available on the sites.  

# Why Singularity?

There are available many solutions to work with containers, for example [Docker](https://www.docker.com/), 
one of the most popular platforms. However, the enterprise-based container frameworks were motivated to provide 
_micro-services_, a solution that fits well in the models of the industry, but not so compatible with 
the workflow in the high-performance computing (HPC), in which usually complex applications run exhaustively with 
all the resources available. 
  
Singularity is a container platform created for the HPC use case. It allows to build and run containers with just 
a few steps in most of the cases, preserving in its design key concepts for the scientific community: 
* Single-file based container images, facilitating the distribution, archiving and sharing.
* Preserves the permissions in the environment. User outside the container is the same user inside.
* No root container daemon required, making it safely available in large computer centers.  
* Simple integration with resource managers and distributed computing frameworks.

 
 <a href="https://sylabs.io/guides/3.6/user-guide">
<img src="https://sylabs.io/guides/3.6/user-guide/_static/logo.png" alt="Singularity" width="220">
</a>

# Documentation

The [official Singularity documentation](https://sylabs.io/docs/) is available online. Contains basic and advanced 
usage of Singularity beyond the scope of this training document. Take a look and read the nice 
[introduction](https://sylabs.io/guides/3.6/user-guide/introduction.html), explaining the motivation behind the 
creation of Singularity. 


{% include links.md %}

