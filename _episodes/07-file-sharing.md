---
title: "Share files between host and container"
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

https://apptainer.org/docs/user/main/bind_paths_and_mounts.html

# Bind paths included by default

When executing a container d

> ## Disabling system binds
>
> --no-mount

# User-defined bind paths

Singularity provides mechanisms to specify additional binds when executing a container via command-line
or environment variables.

> ## Binding directories with Docker-like syntax using ``--mount``
>
> The flag --mount provides a method to bind directories using the syntax of Docker.
> The bind is specified with the format ``type=bind,src=<source>,dst=<dest>``.
> Currently, only ``type=bind`` is supported. Check the
> [documentation](https://apptainer.org/docs/user/main/bind_paths_and_mounts.html#mount-examples) for
> additional options available.
