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
apptainer instance stop mycentos
```
You can confirm the instance doesn't exist with `instance list`.

{% include links.md %}
