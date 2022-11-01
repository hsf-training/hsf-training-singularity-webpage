---
title: "Building Containers"
teaching: 40
exercises: 30
questions:
- "How to build containers with my requirements?"
objectives:
- "Download and assemble containers from available images in the repositories."
keypoints:
- "The command `build` is the basic tool for the creation of containers."
- "A _sandbox_ is a writable directory where containers can be built interactively."
- "Superuser permissions are required to build containers if you need to install packages or manipulate the operating system."
---

Running containers from the available public images is not the only option. In many cases, it is required to modify
an image or even to create a new one from scratch. For such purposes, Singularity provides the command `build`,
defined in the documentation as the _Swiss army knife_ of container creation.

Sometimes, building images requires access to a machine with Singularity and superuser permissions
(if you need to manipulate the packager manager, for example). In such case the usual workflow is to prepare
a container in a build environment (like your laptop), either with an interactive session or from a definition file,
and then deploy the container into a production environment for execution (as your institutional cluster).

<figure>
  <img src="https://journals.plos.org/plosone/article/figure/image?size=large&id=10.1371/journal.pone.0177459.g001" alt="Singularity usage workflow"/>
  <figcaption>'Singularity usage workflow' via <i>Kurtzer GM, Sochat V, Bauer MW (2017) Singularity: Scientific containers for mobility of compute. PLoS ONE 12(5): e0177459. <a href="https://doi.org/10.1371/journal.pone.0177459">https://doi.org/10.1371/journal.pone.0177459</a></i></figcaption>
</figure>

## Build a container in an interactive session

While images contained in the `.sif` files are immutable objects, ideal for reproducibility, for building images in an
interactive session is convenient the usage of a _sandbox_.
The command `build` provides a flag `--sandbox` that will create a writable directory:
```bash
singularity build --sandbox myPython docker://python:3.9
```
The container name is `myPython`, and it has been initialized from the [Docker image](https://hub.docker.com/_/python)
of Python using the tag 3.9.

To initialize an interactive session use `shell`. And to write files within the container use the flag `--writable`:
```bash
singularity shell --writable myUbuntu
```
Once inside the container, you can make changes as needed for your work. For example, making
[Uproot](https://uproot.readthedocs.io/en/latest/index.html) available
```bash
Singularity> pip install uproot awkward
```
Exit the container. Then confirm that uproot is available even after closing the shell:
```bash
singularity exec myPython python -c "import uproot; print(uproot.__doc__)"
```
~~~
Uproot: ROOT I/O in pure Python and NumPy.
...
~~~
{: .output}

## Building a container with superuser permissions

Some operations, like the installation of new components will require superuser access. To be superuser inside the container,
you must be also outside (in the host machine). You can either open a shell as root, or use `sudo`:

```bash
$ sudo singularity shell --writable myCentOS7
```
The container name is `myCentOS7`, and it has been initialized from the [official Docker image](https://hub.docker.com/_/centos)
of CentOS7. You can verify that opening a shell with `sudo` makes you root inside the container
```bash
Singularity> whoami
```
~~~
root
~~~
{: .output}

As a second example, let's create a container with Pythia8 available using the `myCentOS7` sandbox.
First, we need to install the development tools (remember that in this interactive session we are superuser):
```bash
Singularity> yum groupinstall 'Development Tools'
Singularity> yum install python3-devel
```

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
