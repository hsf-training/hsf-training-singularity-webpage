---
title: Setup
---

In this document we use Apptainer and Singularity interchangeably. See the [Introduction]({{ page.root }}{% link _episodes/01-introduction.md %})
for more details about existing Apptainer and Singularity versions and the differences between them.

Apptainer/Singularity has become popular and usually it is available in the institutional computing resources.
Check if Singularity is available with
```bash
singularity --version
```
You will need a Linux system (including WSL) to run Apptainer/Singularity natively and it’s easiest to install if you have root access.
If your local computing system does not have Singularity installed, you may
[request it to your system administrator as suggested here](https://apptainer.org/docs/user/main/quick_start.html#apptainer-on-a-shared-resource).
If the above is not possible you can also use the CVMFS distribution if you have CVMFS or
[install from source without root privileges](https://github.com/apptainer/apptainer/blob/main/INSTALL.md).

Preparation of containers with superuser permissions require Singularity in resources with
access as root. See the [quick installation steps](https://apptainer.org/docs/user/main/quick_start.html#quick-installation)
if you would like to install Singularity in your computer.


{% include links.md %}
