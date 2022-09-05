---
title: "Sharing files between host and container"
teaching: 40
exercises: 0
questions:
- "How to read and write files on the host system from within the container?"
objectives:
- "Map directories on your host system to directories within your container."
- "Learn about the bind paths included automatically in all containers."
keypoints:
- "Bind mounts allow reading and writing files within the container."
- "In Singularity, you have same owner and permissions for files inside and outside the container."
- "Some paths are mounted by default by Singularity."
- "Additional directories to bind can be defined using the ``--bind`` option or the environment variable ``$SINGULARITY_BIND``."
---

One of the key features about containers is the isolation of the processes running inside them. It means,
files on the host system are not accessible within the container.
However, it is
very common that some files on the host system are needed inside the container, or you want to write files from the
container to some directory in the host.

We have already used the option ``--bind`` earlier in the module when exploring the options available to run Singularity
containers. In this chapter we will explore further options to bind directories from your host system to directories
within your container.

Remember that in Singularity, your user outside is the same inside the container. And the same happens with permissions and
ownership for files in bind directories.

## Bind paths included by default

For each container executed, Singularity binds automatically some directories by default, and other defined the system
admin in the Singularity configuration. By default, Singularity binds:
* The user's home directory ($HOME)
* The current directory when the container is executed ($PWD)
* System-defined paths: ``/tmp``, ``/proc``, ``/dev``, etc.
Since this is defined in the configuration, it may vary from site to site.


> ## Disabling system binds
>
> If for any reason you want to execute a container removing the default binds, the command-line option ``--no-mount``
> is available. For example, to disable bind of ``/tmp``
> ~~~
> run --no-mount tmp my_container.sif
> ~~~
> {: .source}
{: .callout}


## User-defined bind paths

Singularity provides mechanisms to specify additional binds when executing a container via command-line
or environment variables.

### Bind with command-line options

The command-line option ``--bind (-B)`` will specify the directories that must be linked between the
host and the container. It is available for ``run``, ``exec`` and ``shell`` (as well for ``instance`` that is
not covered yet).

The syntax for using the bind option is ``"source:destination"``, and the paths must be absolute (relative
paths will be rejected). For example
~~~bash
singularity shell --bind /home/user/mydata:/data my_container.sif
~~~
{: .source}
will bind the directory ``mydata/`` from the host as ``/data`` inside the container. If multiple directories must be
available in the container, they can be defined with a comma between each pair of directories, i.e. using the
syntax ``source1:destination1,source2:destination2``.

Also. If the destination is not specified, it will be set as equal as the source. For example
~~~bash
singularity shell --bind /cvmfs,/home/user/my_data:/data my_container.sif
~~~
{: .source}

> ## Binding directories with Docker-like syntax using ``--mount``
>
> The flag --mount provides a method to bind directories using the syntax of Docker.
> The bind is specified with the format ``type=bind,src=<source>,dst=<dest>``.
> Currently, only ``type=bind`` is supported. Check the
> [documentation](https://apptainer.org/docs/user/main/bind_paths_and_mounts.html#mount-examples) for
> additional options available.
{: .callout}

### Bind with environment variables

If the environment variable ``$SINGULARITY_BIND`` is defined, singularity will bind inside any container
the directories specified in the format ``source``, with the destination being optional (in the same way as using
``--bind``). For example:
~~~bash
export SINGULARITY_BIND="/cvmfs"
~~~
{: .source}
will bind CVMFS to all your Singularity containers (``/cvmfs`` must be available in the host, of course).
You can also bind multiple directories using commas between each ``source:destination``.
