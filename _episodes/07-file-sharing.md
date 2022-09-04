---
title: "Sharing files between host and container"
teaching: 40
exercises: 30
questions:
- "How to read and write files on the host system from within the container?"
objectives:
- "Map directories on your host system to directories within your container."
- "Learn about the bind paths included automatically in all containers."
keypoints:
- "..."
---

Bind mounts map directories from your host system to directories within your container.


## Bind paths included by default

When executing a container,

> ### Disabling system binds
>
> --no-mount
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
