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
- "Use interactive builds only for development and tests, use definition files for production or publicly distributed containers."
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/aqjxxc2kMlU?list=PLKZ9c4ONm-VkxWW98Gcn9H6WwykMiqtnF" title="Intro to Apptainer/Singularity #3 - Building Containers"  frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Running containers from the available public images is not the only option. In many cases, it is required to modify
an image or even to create a new one from scratch. For such purposes, Apptainer provides the command `build`,
defined in the documentation as the _Swiss army knife_ of container creation.

The usual workflow is to prepare and test a container in a build environment (like your laptop),
either with an interactive session or from a definition file,
and then to deploy the container into a production environment for execution (as your institutional cluster).
Interactive sessions are great to experiment and test your new container.
If you want to distribute the container or use it in production, then we recommend to build it from a definition file, as described in the next episode.
This ensures the greatest possibility of reproducibility and transparency.

<figure>
  <img src="https://journals.plos.org/plosone/article/figure/image?size=large&id=10.1371/journal.pone.0177459.g001" alt="Apptainer/Singularity usage workflow"/>
  <figcaption>'Apptainer/Singularity usage workflow' via <i>Kurtzer GM, Sochat V, Bauer MW (2017) Singularity: Scientific containers for mobility of compute. PLoS ONE 12(5): e0177459. <a href="https://doi.org/10.1371/journal.pone.0177459">https://doi.org/10.1371/journal.pone.0177459</a></i></figcaption>
</figure>

## Build a container in an interactive session

While images contained in the `.sif` files are more compact and immutable objects, ideal for reproducibility, for building and testing images is
more convenient the use of a _sandbox_, which can be easily modified.

The command `build` provides a flag `--sandbox` that will create a writable directory, `myCentOS7`, in your work directory:
```bash
apptainer build --sandbox myCentOS7 docker://centos:centos7
```

> ## Notes on shared file systems like AFS
> Avoid using the [`AFS` (Andrew File System)](https://en.wikipedia.org/wiki/Andrew_File_System) and possibly other shared file systems
> as sandbox directory, as these systems can lead to permission issues.
> In particular, this applies to your home directory on [`lxplus`](https://cern.service-now.com/service-portal?id=service_element&name=lxplus-service).
> Instead, make sure to use the local file system by creating a folder in `/tmp/`: `mkdir /tmp/$USER`.
> Then, replace `myCentOS7` in the previous (and next) command with `/tmp/$USER/myCentOS7`.
{:.callout}

The container name is `myCentOS7`, and it has been initialized from the [official Docker image](https://hub.docker.com/_/centos)
of CentOS7.
To initialize an interactive session use the `shell` command. And to write files within the sandbox directory use the `--writable` option.
Finally, the installation of new components will require superuser access inside the container, so use also the `--fakeroot` option, unless you are already root also outside.
```bash
apptainer shell --writable --fakeroot myCentOS7
Apptainer> whoami
```
~~~
root
~~~
{: .output}
Depending on the Apptainer/Singularity installation (privileged or unprivileged) and the version, you may have some requirements, like the `fakeroot` utility or `newuidmap` and `newgidmap`.
If you get an error when using `--fakeroot` have a look at the [fakeroot documentation](https://apptainer.org/docs/user/main/fakeroot.html).
> ## `--fakeroot` is not root
> ATTENTION! [`--fakeroot`](https://apptainer.org/docs/user/main/fakeroot.html) allows you to be root inside a container that you own but is not changing who you are outside.
> All the outside actions and the writing on bound files and directories will happen as your outside user, even if in the container is done by root.
{: .callout}

As an example, let's create a container with Pythia8 available using the `myCentOS7` sandbox.
First, we need to install the development tools (remember that in this interactive session we are superuser):
```bash
Apptainer> yum groupinstall 'Development Tools'
Apptainer> yum install python3-devel
```
Where `yum` is the [package manager used in RHEL distributions](https://en.wikipedia.org/wiki/Yum_(software))
(like CentOS).

We will follow the
installation steps described in the [Pythia website](https://pythia.org/).
Here is a summary of the commands you will need (you may need to adjust link and commands if there is a new Pythia version):

```bash
Apptainer> mkdir /opt/pythia && cd /opt/pythia
Apptainer> curl -o pythia8307.tgz https://pythia.org/download/pythia83/pythia8307.tgz
Apptainer> tar xvfz pythia8307.tgz
Apptainer> ln -s pythia8307 pythia8
```

To enable the Python interface, we add the flag `--with-python-include` during the configuration, pointing to the
location of the Python headers:

```bash
Apptainer> cd pythia8307
Apptainer> ./configure --with-python-include=/usr/include/python3.6m/
Apptainer> make

Apptainer> exit
```

Now, open an interactive session with your user (no `--fakeroot`). You can use now the container with Pythia8 ready in a
few steps. Let's use the Python interface:

```bash
apptainer shell myCentOS7

Apptainer> export PYTHONPATH=/opt/pythia/pythia8/lib:$PYTHONPATH
Apptainer> export LD_LIBRARY_PATH=/opt/pythia/pythia8/lib:$LD_LIBRARY_PATH
Apptainer> python3

>>> import pythia8
>>> pythia = pythia8.Pythia()
```
~~~
 *------------------------------------------------------------------------------------*
 |                                                                                    |
 |  *------------------------------------------------------------------------------*  |
 |  |                                                                              |  |
 |  |                                                                              |  |
 |  |   PPP   Y   Y  TTTTT  H   H  III    A      Welcome to the Lund Monte Carlo!  |  |
 |  |   P  P   Y Y     T    H   H   I    A A     This is PYTHIA version 8.307      |  |
 |  |   PPP     Y      T    HHHHH   I   AAAAA    Last date of change: 25 Feb 2022  |  |
 |  |   P       Y      T    H   H   I   A   A                                      |  |
 |  |   P       Y      T    H   H  III  A   A    Now is 03 Nov 2022 at 18:38:40    |  |

...
 |  |   Copyright (C) 2022 Torbjorn Sjostrand                                      |  |
 |  |                                                                              |  |
 |  *------------------------------------------------------------------------------*  |
 |                                                                                    |
 *------------------------------------------------------------------------------------*
~~~
{: .output}

Notice that we need to define the environment variables `PYTHONPATH` and `LD_LIBRARY_PATH` in order to use Pythia on Python.
We will automate this in the next section.

> ## Execute Python with PyROOT available
>
> Build a container to use uproot in Python 3.9.
>
> > ## Solution
> >
> > Start from the [Python 3.9 Docker image](https://hub.docker.com/_/python) and create the `myPython` sandbox:
> > ```bash
> > apptainer build --sandbox myPython docker://python:3.9
> > apptainer shell myPython
> > ```
> > Once inside the container, you can install [Uproot](https://uproot.readthedocs.io/en/latest/index.html).
> > ```bash
> > Apptainer> python3 -m pip install --upgrade pip
> > Apptainer> python3 -m pip install uproot awkward
> > ```
> > Exit the container and use it as you like:
> > ```bash
> > apptainer exec myPython python -c "import uproot; print(uproot.__doc__)"
> > ```
> > ~~~
> > Uproot: ROOT I/O in pure Python and NumPy.
> > ...
> > ~~~
> > {: .output}
> > Notice how we did not need neither `--writable ` nor `--fakeroot` for the installation, but everything worked fine since pip installs user packages in the user $HOME directory.
> > In addition, Apptainer/Singularity by default mounts the user home directory as read+write, even if the container is read-only.
> > This is why a _sandbox_ is great to test and experiment locally, but should not be used for containers that will be shared or deployed. Manual changes and local directories are difficult to reproduce and control. Once you are happy with the content, you should use definition files, described in the next episode.
> {: .solution}
{: .challenge}
