---
title: "Singularity/Apptainer instances"
teaching: 40
exercises: 0
questions:
- "How can I keep my container running in the background?"
- "What are the use cases for instances?"
objectives:
- "Run containers in a detached mode to keep services up."
- "Deploy instances via definition files."
- "Bind directories from the host to instances."
keypoints:
- TBD
---

As we have studied in previous chapters, commands such as `run` and `shell` allocate Singularity/Apptainer
containers in the foreground, stopping any process running inside the container after logout. This behavior
suits the use case of containers for executing interactive commands in a well-defined environment, but there
are cases when running processes in the background is convenient. For example, when a web application like a
Jupyter notebook is deployed inside a container, it is desired to keep the container up waiting for connections
from the web browser.

Singularity/Apptainer provide the concept of instances for deploy services in the background. In this chapter
we will learn the basics about their capabilities and some use cases as examples.

## Instances from image files

To start an instance, Singularity provide the command `instance`. To exemplify let's pull the CentOS image used
in previous chapters
```bash
singularity pull docker://centos:centos7
```

The image must be started in the following way:
```bash
singularity instance start centos_centos7.sif mycentos7
```
In this example, the `.sif` is the image downloaded from Dockerhub, and `mycentos7` is the name that we have
assigned to the instance. Instead of opening a shell session or executing a command, the container is running on
the background.

Confirm that the instance is running using the `instance list` command
```bash
singularity instance list
```
~~~
INSTANCE NAME    PID      IP    IMAGE
mycentos7        10782          /home/myuser/centos_centos7.sif
~~~
{: .output}

To interact with the instance, the commands `exec` and `shell` available. The instance must be referred as
`instance://name`.
For example, to open a shell inside the CentOS instance:
```bash
singularity shell instance://mycentos7
```

Remember that exiting the shell instance will not stop the container. For doing so, use `instance stop`:
```bash
singularity instance stop mycentos
```
You can confirm the instance doesn't exist with `instance list`.

{% include links.md %}

> ## Instances with bind paths
>
> When starting an instance, the same options for bind directories between the host and the container when running
> an interactive session are available. For example, if you want a directory mounted inside the instance, use the
> `--bind` option:
> ```bash
> singularity instance start --bind /home/user/mydata:/data centos_centos7.sif mycentos7
> run --no-mount tmp my_container.sif
> ```
> binding the directory `mydata/` from the host as `/data` inside the instance.
{: .callout}


## A web server as an instance

One of the main purposes of the Singularity instances is deploying services with customized environments. Before moving
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
If you remember the chapter about [definition files](https://hsf-training.github.io/hsf-training-singularity-webpage/05-definition-files/index.html),
this definition file will pull the official Ubuntu image from Dockerhub, and will install Python3.9.
In addition, it copies `index.html` on `/tmp` **inside** the container.

When the instance starts, commands specified on `%startscript` are executed. On this example, `http.server` will be
executed serving a page in the port 8850.

Now, let's build an image from the definition as
```bash
singularity build --fakeroot basicServer.sif basicServer.def
```

And let's start an instance with the image called `myWebService`:
```bash
singularity instance start --no-mount tmp basicServer.sif myWebService
```
Reminder from the previous chapter: with `--no-mount tmp` we are asking Singularity to NOT bind `/tmp` from the host
to the instance (it is mounted by default), we use instead an isolated `/tmp` inside the instance where index.html has
been copied.

You can confirm in the terminal that the web service is up with
```bash
curl http://localhost:8850
```
If you are executing Singularity locally, try to open http://localhost:8850.

> ## SSH tunneling
>
> If you are deploying a service in the machines of your institution (as LXPLUS at CERN) it is likely that you need
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

As an example of the capabilities of instances as services, let's extend our definition file example to deploy a
JupyterLab server with a customized environment.

```
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get update -y
    apt-get install -y python3
    apt-get install -y python3-pip
    pip install notebook

%files

%startscript
   jupyter notebook --port 8850
```


