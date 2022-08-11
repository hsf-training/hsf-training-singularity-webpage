---
title: "Building Containers"
teaching: 40
exercises: 30
questions:
- "How to build containers with my requirements?"
objectives:
- "Download and assemble containers from the available images in the Singularity Library and the Docker Hub."
- "Create a container from a definition file."
keypoints:
- "The command `build` is the basic tool for the creation of containers."
- "A _sandbox_ is a writable directory where containers can be built interactively."
- "A Singularity definition file provides an easy way to build and deploy containers."
---

Running containers from the available public images is not the only option. In many cases, it is required to modify
an image or even to create a new one from scratch. For such purposes, Singularity provides the command `build`,
defined in the documentation as the _Swiss army knife_ of container creation.

While images contained in the `.sif` files are immutable objects, ideal for reproducibility, for building images is
more convenient the usage of a _sandbox_.

Building images requires access to a machine with Singularity and superuser permissions. The usual workflow is prepare
a container in a build environment (like your laptop), either with an interactive session or from a definition file,
and then deploy the container into a production environment for execution (as your institutional cluster).

<a href="https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0177459">
<img src="http://singularity.lbl.gov/assets/img/diagram/singularity-2.4-flow.png" alt="Singularity" height="400">
</a>

# Build a container in a interactive session

For building a container from a existing image in an interactive way, `build` provides a flag `--sandbox` that will
create a writable directory.

~~~bash
$ singularity build --sandbox myCentOS7 docker://centos:centos7
~~~
{: .source}

The sandbox name is `myCentOS7`, and it has been initialized from the [official Docker image](https://hub.docker.com/_/centos)
of CentOS7.
To initialize an interactive session use `shell`. And to write files within the sandbox directory  `--writable`.
Additionally, the installation of new components will require superuser access:

~~~bash
$ sudo singularity shell --writable myCentOS7

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

# Containers from definition files

As shown below, build containers with an interactive session may take several steps, and it can become as complicated
as the setup is needed.
A Singularity definition file provides an easy way to build and deploy containers.

Let's look at the structure of the definition file with another example. Let's prepare a container from an [official
Ubuntu image](https://hub.docker.com/_/ubuntu), but this time we will install ROOT with RooFit and Python integration.


Following the ROOT instructions to
[download a pre-compiled binary distribution](https://root.cern/install/#download-a-pre-compiled-binary-distribution),
the definition file will look like

~~~
BootStrap: docker
From: ubuntu:20.04

%post
    apt-get update -y
    apt-get install wget -y
    export DEBIAN_FRONTEND=noninteractive
    apt-get install dpkg-dev cmake g++ gcc binutils libx11-dev libxpm-dev \
    libxft-dev libxext-dev python libssl-dev libgsl0-dev libtiff-dev -y
    cd /opt
    wget https://root.cern/download/root_v6.22.06.Linux-ubuntu20-x86_64-gcc9.3.tar.gz
    tar -xzvf root_v6.22.06.Linux-ubuntu20-x86_64-gcc9.3.tar.gz

%environment
    export PATH=/opt/root/bin:$PATH
    export LD_LIBRARY_PATH=/opt/root/lib:$LD_LIBRARY_PATH
    export PYTHONPATH=/opt/root/lib

%runscript
    python /opt/root/tutorials/roofit/rf101_basics.py

%labels
    Author HEPTraining
~~~
{: .source}

Let's take a look at the definition file:
* The first two lines define the base image. In this case, the image `ubuntu:20.04` from Docker Hub is used.
* `%post` are lines to execute inside the container after the OS has been set. In this example, we are listing the
steps that we would follow to install ROOT with a precompiled binary in an interactive session.
Notice that the binary used corresponds with the Ubuntu version defined at the second line.
* `%environment` is used to define environment variables available inside the container. Here we are setting the env
variables required to execute ROOT and PyROOT.
* Singularity containers can be executable. `%runscript` define the actions to take when the container is executed.
To illustrate the functionality, we will just run [rf101_basics.py](https://root.cern/doc/master/rf101__basics_8py.html)
from the RooFit tutorial.
* `%labels` add custom metadata to the container.

Save this definition file as `myUbuntu.def`. To build the container, just provide the definition file as argument
(executing as superuser):
~~~bash
$ sudo singularity build rootInUbuntu.sif myUbuntu.def
~~~
{: .source}

Then, an interactive shell inside the container can be initialized with `singularity shell`, or
execute a command with `singularity exec`. A third option is execute the actions defined inside `%runscript`
simply by calling the container as an executable

~~~bash
$ ./rootInUbuntu.sif
~~~
{: .source}

~~~
RooFit v3.60 -- Developed by Wouter Verkerke and David Kirkby
                Copyright (C) 2000-2013 NIKHEF, University of California & Stanford University
                All rights reserved, please read http://roofit.sourceforge.net/license.txt
...
 PARAMETER  CORRELATION COEFFICIENTS
       NO.  GLOBAL      1      2
        1  0.02723   1.000  0.027
        2  0.02723   0.027  1.000
[#1] INFO:Minization -- RooMinimizer::optimizeConst: deactivating const optimization
RooRealVar::mean = 1.01746 +/- 0.0300144  L(-10 - 10)
RooRealVar::sigma = 2.9787 +/- 0.0219217  L(0.1 - 10)
Info in <TCanvas::Print>: png file rf101_basics.png has been created
~~~
{: .output}

You will find the output file `rf101_basics.png` in the location where the container was executed.

> ## Deploying your containers
> Keep in mind that, while build a container requires superuser permissions, the execution can be performed anywhere.
> Once your container is built with the requirements of your analysis, you can deploy it in a large cluster and execute it
> as far as Singularity is available on the site.
>
> Be smart, and this will open endless possibilities in your workflow.
{: .callout}

Here we have covered the basics with a few examples focused to HEP software.
Check the (Singularity docs)[https://sylabs.io/guides/3.6/user-guide/build_a_container.html] to see all the available
options and more details related to the container creation.

> ## Write a definition file to build a container with Pythia8 available in Python
>
> Following the example of the first section in which a container is built with an interactive session,
> write a definition file to deploy a container with Pythia8 available.
>
> Take a look at
> [`/opt/pythia/pythia8303/examples/main01.py`](https://github.com/alisw/pythia8/blob/master/examples/main01.py)
> and define the `%runscript` to execute it using `python3`.
>
> (Tip: notice that main01.py requires `Makefile.inc`).
>
> > ## Solution
> > ~~~
> > BootStrap: docker
> > From: centos:centos7
> >
> > %post
> >     yum -y groupinstall 'Development Tools'
> >     yum -y install python3-devel
> >     mkdir /opt/pythia && cd /opt/pythia
> >     curl -o pythia8303.tgz http://home.thep.lu.se/~torbjorn/pythia8/pythia8303.tgz
> >     tar xvfz pythia8303.tgz
> >     cd pythia8303
> >     ./configure --with-python-include=/usr/include/python3.6m/
> >     make
> >
> > %environment
> >     export PYTHONPATH=/opt/pythia/pythia8303/lib:$PYTHONPATH
> >     export LD_LIBRARY_PATH=/opt/pythia/pythia8303/lib:$LD_LIBRARY_PATH
> >
> > %runscript
> >     cp /opt/pythia/pythia8303/Makefile.inc .
> >     python3 /opt/pythia/pythia8303/examples/main01.py
> >
> > %labels
> >     Author HEPTraining
> > ~~~
> > {: .source}
> >
> > Build your container executing
> >
> > ~~~bash
> > $ sudo singularity build pythiaInCentos7.sif myPythia8.def
> > ~~~
> > {: .source}
> >
> > And finally, execute the container to run [`main01.py`](https://github.com/alisw/pythia8/blob/master/examples/main01.py)
> >
>> > ~~~bash
> > $ ./pythiaInCentos7.sif
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}
