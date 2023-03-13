---
title: "Apptainer/Singularity instances"
teaching: 60
exercises: 10
questions:
- "How can I keep my container running in the background?"
- "What are the use cases for instances?"
objectives:
- "Run containers in a detached mode to keep services up."
- "Deploy instances via definition files."
keypoints:
- Instances allow to setup services via Apptainer images or definition files.
- Code provided in Jupyter notebooks can be accompanied by a Apptainer/Singularity image with the environment needed for its execution, ensuring the reproducibility of the results.
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/10s8yUScYM8" title="Intro to Apptainer/Singularity #6 - Instances"  frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

As we have studied in previous chapters, commands such as `run` and `shell` allocate Apptainer/Singularity
containers in the foreground, stopping any process running inside the container after logout. This behavior
suits the use case of containers for executing interactive commands in a well-defined environment, but there
are cases when running processes in the background is convenient. For example, when a web application like a
Jupyter notebook is deployed inside a container, it is desired to keep the container up while it waits for connections
from the web browser.

Apptainer provides the concept of _instances_ to deploy services in the background. While Docker is a common
choice of tool for setting services, Apptainer has the advantage of working without requiring any special permissions
(like when you are working in a cluster provided by your university/laboratory).
In this chapter we will learn the basics about their capabilities and some use cases as examples.

## Instances from image files

To start an instance, Apptainer provides the command `instance`. To exemplify, let's pull the CentOS image used
in previous chapters
```bash
apptainer pull docker://centos:centos7
```

The image must be started in the following way:
```bash
apptainer instance start centos_centos7.sif mycentos7
```
In this example, the `.sif` is the image downloaded from Dockerhub, and `mycentos7` is the name that we have
assigned to the instance. Instead of opening a shell session or executing a command, the container is running in
the background.

Confirm that the instance is running using the `instance list` command
```bash
apptainer instance list
```
~~~
INSTANCE NAME    PID      IP    IMAGE
mycentos7        10782          /home/myuser/centos_centos7.sif
~~~
{: .output}

To interact with the instance, the commands `exec` and `shell` are available. The instance must be referred as
`instance://name`.
For example, to open a shell inside the CentOS instance:
```bash
apptainer shell instance://mycentos7
```

Remember that exiting the shell instance will not stop the container. For doing so, use `instance stop`:
```bash
apptainer instance stop mycentos7
```
You can confirm the instance doesn't exist with `instance list`.

{% include links.md %}

> ## Instances with bind paths
>
> When starting an instance, the same options for bind directories between the host and the container as running
> an interactive session are available. For example, if you want a directory mounted inside the instance, use the
> `--bind` option:
> ```bash
> apptainer instance start --bind /home/user/mydata:/data centos_centos7.sif mycentos7
> ```
> binding the directory `mydata/` from the host as `/data` inside the instance.
{: .callout}


## A web server as an instance

One of the main purposes of the Apptainer instances is deploying services with customized environments. Before moving
to more complex use cases, let's start with a basic example: a web service showing a HTML with a message.

Let's write a basic `index.html` file as:
```html
<!DOCTYPE html>
<html>
<head>
<title>My awesome service</title>
</head>
<body>
<h1>Hello world!</h1>
<p>If you see this page, my awesome service is up and running.</p>
</body>
</html>
```
If you are not familiar with HTML take a quick look at the [HTML Tutorial](https://www.w3schools.com/html/), but it is
not mandatory. What really matters is having a minimal webpage that our server will show.

Now, let's prepare a basic web server using [Python http.server](https://docs.python.org/3.9/library/http.server.html).
Create a definition file, saved as `basicServer.def`, which contains:
```
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get update -y
    apt-get install -y python3.9

%files
    index.html /tmp/index.html

%startscript
   cd /tmp
   python3.9 -m http.server 8850
```
If you recall the chapter about [definition files]({{ page.root }}{% link _episodes/05-definition-files.md %}),
this definition file will pull the official Ubuntu image from Dockerhub, and will install Python3.9.
In addition, it copies `index.html` in `/tmp` **inside** the container. When the instance starts, commands specified on
`%startscript` are executed. On this example, `http.server` will be executed, serving a page in the port 8850 (you can
use any other port if 8850 is busy with another service).

Let's build an image from the definition. Remember that building images requires either superuser permissions or
using the flag `--fakeroot` as
```bash
apptainer build --fakeroot basicServer.sif basicServer.def
```

Now, let's start an instance named `myWebService` with the image that we just built
```bash
apptainer instance start --no-mount tmp basicServer.sif myWebService
```
Reminder from the previous chapter: with `--no-mount tmp` we are asking Apptainer to NOT bind `/tmp` from the host
to the instance (it is mounted by default), we use instead an isolated `/tmp` inside the instance where index.html has
been copied.

You can confirm in the terminal that the web service is up using `curl` as
```bash
curl http://localhost:8850
```
~~~
<!DOCTYPE html>
<html>
<head>
<title>Welcome to my service!</title>
</head>
<body>
<h1>Hello world!</h1>
<p>If you see this page, my awesome service is up and running.</p>
</body>
</html>
~~~
{: .output}

If you are executing Apptainer locally, try to open http://localhost:8850.

> ## SSH tunneling
>
> If you are deploying a service in a cluster of your institution (as LXPLUS at CERN) it is likely that you need
> SSH tunneling for opening pages served by your service with a web browser. A basic port forwarding can be configured as:
> ```bash
> ssh -L <port>:localhost:<port> myuser@<server>
> ```
> where <port> is the one used by your service, and <server> is the address of your institutional resources. For example,
> for connecting to LXPLUS forwarding the port 8850:
> ```bash
>  ssh -L 8850:localhost:8850 myuser@lxplus.cern.ch
> ```
> Then you can open http://localhost:8850 in your machine!
>
> Port numbers less than 1024 are privileged ports and can be used only by root.
> Please consult the allowed ports and rules with your institution.
{: .callout}

Remember to stop the instance once you are done.

## Serving a Jupyter notebook with custom environment

As an example of the capabilities of instances as services, let's extend our definition file to deploy a
Jupyter notebook server with a customized environment.

What if we provide a Jupyter notebook ready to use ROOT? If you remember our example from the
[definition files chapter]({{ page.root }}{% link _episodes/05-definition-files.md %}),
at this point it must be almost straightforward:
```
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get update -y
    apt-get install -y python3
    apt-get install -y python3-pip
    pip install notebook

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

%startscript
   jupyter notebook --port 8850
```

Save the definition file as `jupyterWithROOT.def`, and let's build an image called `jupyterWithROOT.sif`
```bash
apptainer build --fakeroot jupyterWithROOT.sif jupyterWithROOT.def
```
Now, start an instance named `mynotebook` with our brand-new image
```bash
apptainer instance start jupyterWithROOT.sif mynotebook
```
and confirm that the instance is up
```bash
apptainer instance list
```
~~~
INSTANCE NAME    PID      IP    IMAGE
mynotebook       10720          /home/myuser/jupyterWithROOT.sif
~~~
{: .output}

If you go to http://localhost:8850 (with SSH tunneling if needed), you will find out that for security reasons the
Jupyter webapp will ask for an access token. Fortunately, you can get the token listing the URL of active servers using
the `jupyter notebook list` command. To execute the command inside the instance, use `sigularity exec`:
```bash
apptainer exec instance://notebook jupyter notebook list
```
~~~
Currently running servers:
http://localhost:8850/?token=12asldc9b2084f9b664b39a6246022312bc9c605b :: /home/myHome
~~~
{: .output}

Open the URL with the token, and you will be able to see the Jupyter interface. Try to open a new notebook and write in
a cell to confirm that ROOT is available:
```python
import ROOT
# Now you can work with PyROOT, creating a histogram for example
h = ROOT.TH1F("myHistogram", "myTitle", 50, -10, 10)
h.FillRandom("gaus", 10000)

c = ROOT.TCanvas("myCanvasName","The Canvas Title",800,600)
h.Draw()
c.Draw()
```

The bottom line: with any Jupyter notebook that you write, you can provide an Apptainer image that will
set the environment required to execute the cells. It doesn't matter if yourself or someone else comes in one, five,
ten years, your code will work independently of the software available in your computer as far as Apptainer/Singularity
is available!

> ## A Jupyter notebook with Uproot available
>
> Can you setup a Jupyter notebook server with [Uproot](https://uproot.readthedocs.io/en/latest/index.html) available in Apptainer?
>
> Hint: Uproot can be installed using `pip`.
>
> > ## Solution
> >
> >```
> >Bootstrap: docker
> >From: ubuntu:20.04
> >
> >%post
> >    apt-get update -y
> >    apt-get install -y python3
> >    apt-get install -y python3-pip
> >    pip install notebook
> >    pip install uproot
> >
> >%startscript
> >   jupyter notebook --port 8850
> >```
> >Confirm that Uproot is available opening a notebook and executing in a cell
>> ```python
>> import uproot
>> print(uproot.__doc__)
>>```
> {: .solution}
{: .challenge}
