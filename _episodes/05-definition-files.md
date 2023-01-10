---
title: "Containers from definition files"
teaching: 40
exercises: 30
questions:
- "How to easily build and deploy containers from a single definition file?"
objectives:
- "Create a container from a definition file."
keypoints:
- "A Singularity definition file provides an easy way to build and deploy containers."
---

As shown in the previous chapter, building containers with an interactive session may take several steps, and it can
become as complicated as the setup is needed.
A Singularity definition file provides an easy way to build and deploy containers.


## Hello World Singularity

The following recipe shows how to build a hello-world container, and run the container on your local computer.

- Step 1: Open a text editor (e.g., nano, vim, or gedit in a graphical environment)

  ```bash
  nano hello-world.def
  ```

- Step 2: Include the following script in the `hello-world.def` file to define the environment

  ```bash
  BootStrap: docker
  From: ubuntu:20.04

  %runscript
    echo "Hello World"
  # Print Hello world when the image is loaded
  ```

    In the above script, the first line - `BootStrap: docker` indicates that singularity will use the docker protocol to retrieve the base OS to start the image.
The `From: ubuntu:20.04` is given to singularity to start from a specific image/OS in Docker Hub.
Any content within the  `%runscript` will be written to a file that is executed when one runs the singularity image.
The `echo "Hello World"` command will print the `Hello World` on the terminal.
Finally the `#` hash is used to include comments within the definition file.

- Step 3: Build the image

  ```bash
  singularity build hello-world.sif hello-world.def
  ```

    The `hello-world.sif` file specifies the name of the output file that is built when using the `singularity build` command.

- Step 4: Run the image

  ```bash
  ./hello-world.sif
  ```

### Deleting Singularity image
To delete the hello-world Singularity image, simply delete the `hello-world.sif` file.

> Note that there is also a `singularity delete` command, but it is to delete an image from a remote library.
> To learn more about using remote endpoints and pulling and pushing images from or to libraries, read
> [Remote Endpoints](https://apptainer.org/docs/user/main/endpoint.html) and [Library API Registries](https://apptainer.org/docs/user/main/library_api.html).
{: .callout}


## Example of a more elaborated definition file

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

Let's take a look at the [definition file](https://apptainer.org/docs/user/main/definition_files.html):
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

Save this definition file as `rootInUbuntu.def`. To build the container, just provide the definition file as argument
(executing as superuser):
```bash
singularity build rootInUbuntu.sif rootInUbuntu.def
```


Then, an interactive shell inside the container can be initialized with `singularity shell`, or
a command executed with `singularity exec`. A third option is execute the actions defined inside `%runscript`
simply by calling the container as an executable

```bash
./rootInUbuntu.sif
```

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
If you don't have a DISPLAY setup, Root may complain. Ignore the error messages, the image will be created anyway,

Here we have covered the basics with a few examples focused to HEP software.
Check the [Apptainer docs](https://apptainer.org/docs/user/main/build_a_container.html) to see all the available
options and more details related to the container creation.


> ## Deploying your containers
> Keep in mind that, while building a container may be time consuming, the execution can be immediate and anywhere your image is available.
> Once your container is built with the requirements of your analysis, you can deploy it in a large cluster and execute it
> as far as Singularity is available on the site.
>
> Libraries like [Sylabs Cloud Library](https://cloud.sylabs.io/library) ease the distribution of images.
> Organizations like OSG provide instructions to [use available images](https://portal.osg-htc.org/documentation/htc_workloads/using_software/containers/)
and [distribute custom images via CVMFS](https://portal.osg-htc.org/documentation/htc_workloads/using_software/containers-docker/).
>
> Be smart, and this will open endless possibilities in your workflow.
{: .callout}


> ## Write a definition file to build a container with Pythia8 available in Python
>
> Following the example of the first section in which a container is built with an interactive session,
> write a definition file to deploy a container with Pythia8 available.
>
> Take a look at
> [`/opt/pythia/pythia8307/examples/main01.py`](https://gitlab.com/Pythia8/releases/-/blob/pythia8307/examples/main01.py)
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
> >     curl -o pythia8307.tgz https://pythia.org/download/pythia83/pythia8307.tgz
> >     tar xvfz pythia8307.tgz
> >     cd pythia8307
> >     ./configure --with-python-include=/usr/include/python3.6m/
> >     make
> >
> > %environment
> >     export PYTHONPATH=/opt/pythia/pythia8307/lib:$PYTHONPATH
> >     export LD_LIBRARY_PATH=/opt/pythia/pythia8307/lib:$LD_LIBRARY_PATH
> >
> > %runscript
> >     cp /opt/pythia/pythia8307/Makefile.inc .
> >     python3 /opt/pythia/pythia8307/examples/main01.py
> >
> > %labels
> >     Author HEPTraining
> > ~~~
> > {: .source}
> >
> > Build your container executing
> >
> > ```bash
> > singularity build pythiaInCentos7.sif myPythia8.def
> > ```
> >
> > And finally, execute the container to run [`main01.py`](https://gitlab.com/Pythia8/releases/-/blob/pythia8307/examples/main01.py)
> >
>> > ```bash
> > ./pythiaInCentos7.sif
> > ```
> {: .solution}
{: .challenge}
