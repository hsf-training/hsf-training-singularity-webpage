---
title: "Building Containers"
teaching: 40
exercises: 30
questions:
- "How to build containers with my requirements?"
objectives:
- "Download and assemble containers from the available images in the Singularity Library and the Docker Hub."
keypoints:
- "The command `build` is the basic tool for the creation of containers."
- "A _sandbox_ is a writable directory where containers can be built interactively."
---

Running containers from the available public images is not the only option. In many cases, it is required to modify
an image or even to create a new one from scratch. For such purposes, Singularity provides the command `build`,
defined in the documentation as the _Swiss army knife_ of container creation.

While images contained in the `.sif` files are immutable objects, ideal for reproducibility, for building images is
more convenient the usage of a _sandbox_.

Building images requires access to a machine with Singularity and superuser permissions. The usual workflow is prepare
a container in a build environment (like your laptop), either with an interactive session or from a definition file,
and then deploy the container into a production environment for execution (as your institutional cluster).

<figure>
  <img src="https://journals.plos.org/plosone/article/figure/image?size=large&id=10.1371/journal.pone.0177459.g001" alt="Singularity usage workflow"/>
  <figcaption>'Singularity usage workflow' via <i>Kurtzer GM, Sochat V, Bauer MW (2017) Singularity: Scientific containers for mobility of compute. PLoS ONE 12(5): e0177459. <a href="https://doi.org/10.1371/journal.pone.0177459">https://doi.org/10.1371/journal.pone.0177459</a></i></figcaption>
</figure>


# Build a container in a interactive session

For building a container from a existing image in an interactive way, `build` provides a flag `--sandbox` that will
create a writable directory.

~~~bash
singularity build --sandbox myCentOS7 docker://centos:centos7
~~~
{: .source}

The sandbox name is `myCentOS7`, and it has been initialized from the [official Docker image](https://hub.docker.com/_/centos)
of CentOS7.
To initialize an interactive session use `shell`. And to write files within the sandbox directory  `--writable`.
Additionally, the installation of new components will require superuser access:

~~~bash
sudo singularity shell --writable myCentOS7

Singularity> whoami
~~~
{: .source}
~~~
root
~~~
{: .output}

As an example, let's create a container with Pythia8 available using the `myCentOS7` sandbox.
First, we need to install the development tools (remember that in this interactive session we are superuser):

~~~bash
Singularity> yum groupinstall 'Development Tools'
Singularity> yum install python3-devel
~~~
{: .source}

We will follow the
installation steps described in the [Pythia website](http://home.thep.lu.se/~torbjorn/Pythia.html).
In short, we just need to execute:

~~~bash
Singularity> mkdir /opt/pythia && cd /opt/pythia
Singularity> curl -o pythia8303.tgz http://home.thep.lu.se/~torbjorn/pythia8/pythia8303.tgz
Singularity> tar xvfz pythia8303.tgz
~~~
{: .source}

To enable the Python interface, we add the flag `--with-python-include` during the configuration, pointing to the
location of the Python headers:

~~~bash
Singularity> ./configure --with-python-include=/usr/include/python3.6m/
Singularity> make

Singularity> exit
~~~
{: .source}

Now, open an interactive session with your user (no `sudo`). You can use now the container with Pythia8 ready in a
few steps. Let's use the Python interface:

~~~bash
singularity shell myCentOS7

Singularity> export PYTHONPATH=/opt/pythia/pythia8303/lib:$PYTHONPATH
Singularity> export LD_LIBRARY_PATH=/opt/pythia/pythia8303/lib:$LD_LIBRARY_PATH
Singularity> python3

>>> import pythia8
>>> pythia = pythia8.Pythia()
~~~
{: .source}
~~~
 *------------------------------------------------------------------------------------*
 |                                                                                    |
 |  *------------------------------------------------------------------------------*  |
 |  |                                                                              |  |
 |  |                                                                              |  |
 |  |   PPP   Y   Y  TTTTT  H   H  III    A      Welcome to the Lund Monte Carlo!  |  |
 |  |   P  P   Y Y     T    H   H   I    A A     This is PYTHIA version 8.303      |  |
 |  |   PPP     Y      T    HHHHH   I   AAAAA    Last date of change:  1 Sep 2020  |  |
 |  |   P       Y      T    H   H   I   A   A                                      |  |
 |  |   P       Y      T    H   H  III  A   A    Now is 18 Dec 2020 at 18:50:33    |  |

...
 |  |   Copyright (C) 2020 Torbjorn Sjostrand                                      |  |
 |  |                                                                              |  |
 |  *------------------------------------------------------------------------------*  |
 |                                                                                    |
 *------------------------------------------------------------------------------------*
~~~
{: .output}

Notice that we need to define the environment variables `PYTHONPATH` and `LD_LIBRARY_PATH` in order to use Pythia on Python.
We will automate this in the next section.
