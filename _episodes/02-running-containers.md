---
title: "Containers and Images"
teaching: 20
exercises: 5
questions:
- "How to pull Apptainer images from the libraries?"
- "How to run commands inside the containers?"
objectives:
- "Learn to search and pull images from the Sylabs Singularity library and Docker Hub."
- "Interact with the containers using the command-line interface."
keypoints:
- Use `singularity --version` to know what you are using and to communicate it if asking for support
- "A container can be started from a local `.sif` or directly with the URL of the image."
- "Apptainer is also compatible with Docker images, providing access to the large collection of images hosted by Docker Hub."
- Get a shell inside of your container with `apptainer shell <path/URL to image>`
- Execute a command inside of your container with `apptainer exec <path/URL> <command>`
- Bind outside directories with `--bind`
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/hiWQ1CNlxkE?list=PLwN-li4B10E3MtN4620yhGbctjTyQ9jna" title="Intro to Apptainer/Singularity #2 - Containers and Images"  frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# The Apptainer Command Line Interface

Apptainer provides a command-line interface (CLI) to interact with the containers. You can search, build or run
containers in a single line.

You can check the version of the Apptainer or Singularity command you are using with the `--version` option:
```bash
singularity --version
# This works for both Singularity and Apptainer, which installs a link named `singularity` to maintain compatibility.
# In the future you may need to use `apptainer --version`
```
For this training we recommend Apptainer >= 1.0 or Singularity >= 3.5. Older versions may not have some of the features or behave differently.
If you need to install or upgrade Apptainer/Singularity please refer to the [Setup section]({{ page.root }}{% link setup.md %}).
When asking for support please remember to include the version of Apptainer or Singularity being used, as in the output of the above command.

You can check the available options and subcommands using `--help`:

```bash
apptainer --help
```

# Downloading Images

Container Images are executables that bundle together all necessary components for an application or an environment,
like a template for containers.
Containers are the runtime instances of images — they are images with a state. CircleCI has a nice
[explanation of the differences](https://circleci.com/blog/docker-image-vs-container/).

Apptainer/Singularity can store, search and retrieve images in registries (searchable catalogs and repositories for images and containers).
Images built by other users can be accessible using the CLI, can be pulled down, and become containers at runtime.

Sylabs, the developer of one Singularity flavor, hosts a public image registry, the
[Singularity Container Library](https://cloud.sylabs.io/library) where many user built images are available.

Apptainer, the Linux Foundation flavor, does not point by default to the Sylab registry via the
[Library API](https://singularityhub.github.io/library-api/#/) as previous versions did.
You can change that running these commands (documented [here](https://apptainer.org/docs/user/main/endpoint.html#restoring-pre-apptainer-library-behavior)):
```bash
apptainer remote add --no-login SylabsCloud cloud.sycloud.io
```
~~~
INFO:    Remote "SylabsCloud" added.
~~~
{: .output}
```bash
apptainer remote use SylabsCloud
```
~~~
INFO:    Remote "SylabsCloud" now in use.
~~~
{: .output}
```bash
apptainer remote list
```
~~~
Cloud Services Endpoints
========================

NAME           URI                  ACTIVE  GLOBAL  EXCLUSIVE
DefaultRemote  cloud.apptainer.org  NO      YES     NO
SylabsCloud    cloud.sycloud.io     YES     NO      NO
...
~~~
{: .output}

> ## Remote Endpoints, Library API and OCI Registries
> [Remotes](https://apptainer.org/docs/user/main/endpoint.html) are service endpoints Apptainer interacts with.
> These include [Library API Registries](https://apptainer.org/docs/user/1.0/library_api.html),
> [OCI Registries](https://apptainer.org/docs/user/1.0/docker_and_oci.html), and keyservers.
> The first two are used to search, pull and push images.
> The [Library](https://singularityhub.github.io/library-api/#/) API, `library://`,
> was designed for SIF images, the [Singularity Image Format](https://github.com/apptainer/sif).
> The [Docker](https://docs.docker.com/docker-hub/api/latest/)/[ORAS](https://oras.land/) API, `docker://`,
> is used for Docker Hub, and other OCI ([Open Containers Initiative](https://opencontainers.org/)) registries.
> These include [Quay.io](https://quay.io), [NVIDIA NGC](https://ngc.nvidia.com/),
> the [GitHub Container Registry](https://github.com/features/packages),
> [AWS ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html) and many more.
{: .callout}

Once you have setup a working registry you can use search and pull.
The command `search` lists containers of interest
and shows information about users (owners or managers of stored containers) and collections (sets of containers).
For example:

```bash
# this command can take around a minute to complete
apptainer search centos7
```

~~~
No users found for 'centos7'

Found 1 collections for 'centos7'
        library://shahzebmsiddiqui/easybuild-centos7

Found 15 containers for 'centos7'
        library://gmk/default/centos7-devel
                Tags: latest
...
~~~
{: .output}

Downloading an image from the Container Library is pretty straightforward:
```bash
apptainer pull library://gmk/default/centos7-devel
```
and the image is stored locally as a `.sif` file (`centos7-devel_latest.sif`, in this case).

> ## Docker Images
>
> Fortunately, Apptainer is also compatible with Docker images. There are many more registries with Docker images.
> [Docker Hub](https://hub.docker.com/) is one of the largest libraries available,
> and any image hosted on the hub can be easily downloaded with the `docker://` URL as reference:
> ```bash
> apptainer pull docker://centos:centos7
> ```
{: .callout}

# Running Containers

There are several ways to interact with images and start containers. Here we will review how to initialize a shell
environment and how to execute directly a command.

## Initializing a shell and exiting it

The `shell` command initializes a new interactive shell inside the container.

```bash
apptainer shell centos7-devel_latest.sif
```

~~~
Apptainer>
~~~
{: .output}
In this case, the container works as a lightweight virtual machine in which you can execute commands.
Remember, inside the container you have the same user and permissions.

```bash
Apptainer> id
```

~~~
uid=1001(myuser) gid=1001(myuser) groups=1001(myuser),500(myothergroup)
~~~
{: .output}

Now quit the container by typing

```bash
Apptainer> exit
```

or hitting `Ctrl + D`.
Note that when exiting from the Apptainer image all the running processes are killed (stopped).
Changes saved into bound directories are preserved. By default anything else in the container is lost (we'll see later about writable images).

## Bound directories

When an outside directory is accessible also inside Apptainer we say it is *bound*, or bind mounted. The path to access it
may differ but anything you do to its content outside is visible inside and vice-versa.
By default, Apptainer binds the home of the user, `/tmp` and `$PWD` into the container. This means your files
at `hostname:~/` are accessible inside the container. You can specify additional bind mounts using the `--bind` option.
For example, let's say `/cvmfs` is available in the host, and you would like to have access to CVMFS inside the
container (here, *host* refers to the computer/server that you are running apptainer on). Then let's do

```bash
apptainer shell --bind /cvmfs:/mnt centos7-devel_latest.sif
```

Here, the colon `:` separates the path to the directory on the host (`/cvmfs/`) from the mounting point (`/mnt/`) inside of the
container.
If you do not have CVMFS, you can try the command with [`/opt`](https://stackoverflow.com/a/12649407/), for example.
More information on binding is provided [later]({{ site.baseurl }}/07-file-sharing).

Let's check that this works:

~~~
Apptainer> ls /mnt/cms.cern.ch
bin                        etc                  SITECONF           slc7_aarch64_gcc530
bootstrap.sh               external             slc5_amd64_gcc434  slc7_aarch64_gcc700
...
~~~
{: .output}

> ## URLs as input
> Each of the different commands to set a container from a local `.sif` also accepts the URL of the image
> as input. For example, starting a shell with Rocky Linux 8 is as easy as
> ```bash
> apptainer shell docker://rockylinux:8
> ```
> ~~~
> INFO:    Converting OCI blobs to SIF format
> INFO:    Starting build...
> Getting image source signatures
> Copying blob 7ecefaa6bd84 done
> Copying config a8f7ea56a4 done
> Writing manifest to image destination
> Storing signatures
> 2024/02/24 20:32:30  info unpack layer: sha256:7ecefaa6bd84a24f90dbe7872f28a94e88520a07941d553579434034d9dca399
> INFO:    Creating SIF file...
> Apptainer>
> ~~~
> {: .output}
{: .callout}

## Executing commands

The command `exec` starts the container from a specified image and executes a command inside it.
Let's use the official [Docker image of ROOT](https://hub.docker.com/r/rootproject/root) to start [ROOT](https://root.cern/)
inside a container:

```bash
apptainer exec docker://rootproject/root root -b
```

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
...
2024/02/24 20:36:21  info unpack layer: sha256:7aea3382b6b1676fdc2742fef246a9ec593b44cf8ddc81f0f7b1638f2dda6f65
INFO:    Creating SIF file...
   ------------------------------------------------------------------
  | Welcome to ROOT 6.30/04                        https://root.cern |
  | (c) 1995-2024, The ROOT Team; conception: R. Brun, F. Rademakers |
  | Built for linuxx8664gcc on Jan 31 2024, 10:01:37                 |
  | From heads/master@tags/v6-30-04                                  |
  | With c++ (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0                   |
  | Try '.help'/'.?', '.demo', '.license', '.credits', '.quit'/'.q'  |
   ------------------------------------------------------------------

root [0]

~~~
{: .output}

And just like that, ROOT can be used in any laptop, large-scale cluster or grid system
with Apptainer available.

> ## Execute Python with PyROOT available
>
> Using the official Docker image of ROOT, start a Python session with PyROOT available.
>
> > ## Solution
> >
> > ```bash
> > apptainer exec docker://rootproject/root python3
> > ```
> >
> > ~~~
> > INFO:    Using cached SIF image
> > Python 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] on linux
> > Type "help", "copyright", "credits" or "license" for more information.
> > >>> import ROOT
> > >>> # Now you can work with PyROOT, creating a histogram for example
> > >>> h = ROOT.TH1F("myHistogram", "myTitle", 50, -10, 10)
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

{% include links.md %}
