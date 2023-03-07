---
title: Setup
---

<iframe width="427" height="251" src="https://www.youtube.com/embed/g0cCErlveiI?list=PLKZ9c4ONm-VkxWW98Gcn9H6WwykMiqtnF" title="Intro to Apptainer/Singularity #0 - Setup" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In this document we use the names *Apptainer* and *Singularity* interchangeably. See the [Introduction]({{ page.root }}{% link _episodes/01-introduction.md %})
for more details about existing Apptainer and Singularity versions and the differences between them.

## Option 1: Use pre-installed apptainer on a cluster

Apptainer/Singularity has become popular and usually it is available in the institutional computing resources.
Check if apptainer or singularity are available with
```bash
apptainer --version
singularity --version
```
If installed, you will see `apptainer version ...` or `singularity version ...`, depending on the flavor installed.
Apptainer is preferable but either one is OK, so if apptainer is there, no need to check for singularity (which most likely will be a link to apptainer).
This tutorial requires at least Apptainer 1.0.x or Singularity 3.5.x. Previous versions may not have all the required features.
If none is in your [`$PATH`](https://www.makeuseof.com/set-path-variable-in-linux/#what-is-path-in-linux) or if the available version is too old,
you may still be able to use an updated apptainer via [CVMFS](https://cernvm.cern.ch/fs/): check if you have user namespaces enabled and CVMFS to run singularity that way:
```bash
[[ $(cat /proc/sys/user/max_user_namespaces) -gt 0 ]] && ls /cvmfs/oasis.opensciencegrid.org/mis/ &>/dev/null && { export PATH=/cvmfs/oasis.opensciencegrid.org/mis/apptainer/bin/:"$PATH"; echo "Success: Added to PATH"; singularity --version; } || echo "Failure: Unable to run Singularity/Apptainer via CVMFS"
```
If this works, it will be added to your path and you will see your apptainer/singularity version.

If your local computing system does not have Apptainer/Singularity installed, you may
[request it to your system administrator as suggested here](https://apptainer.org/docs/user/main/quick_start.html#apptainer-on-a-shared-resource).

## Option 2: Install Apptainer/Singularity

You will need a **Linux system (including WSL on Windows computers)** to run Apptainer/Singularity natively.
**MacOS is not supported**.


### If you have root access

It is easiest to
[install if you have root access](https://apptainer.org/docs/user/main/quick_start.html#quick-installation).

### If not

If the above is not possible and you cannot use the CVMFS distribution you have still an option if user namespace is enabled on your system:
1. Check if user namespaces are enabled:
    ```bash
    # on Debian/Ubuntu
    grep -q 'kernel.unprivileged_userns_clone=1' /etc/sysctl.d/90-unprivileged_userns.conf && \
        echo "User namespaces enabled, continue the Apptainer installation" || \
        echo "User namespaces NOT enabled, your use of Apptainer will be very limited"
    # on RHEL/CentOS cat /proc/sys/user/max_user_namespaces` is bigger than 0
    [[ $(cat /proc/sys/user/max_user_namespaces) -gt 0 ]] && \
        echo "User namespaces enabled, continue the Apptainer installation" || \
        echo "User namespaces NOT enabled, your use of Apptainer will be very limited"
    ```
   See [these full instructions](https://apptainer.org/docs/admin/main/user_namespace.html#user-namespace-requirements) for more about checking for and enabling user namespaces.
1. If enabled, install unprivileged Apptainer with one of these three methods (in order of preference):
    1. Chose your `INSTALL_DIR` and [install there the relocatable Apptainer (recommended)](https://apptainer.org/docs/admin/main/installation.html#install-unprivileged-from-pre-built-binaries). Run:
        ```bash
        curl -s https://raw.githubusercontent.com/apptainer/apptainer/main/tools/install-unprivileged.sh | \
            bash -s - INSTALL_DIR
        ```
    1. Alternatively [install from source without root privileges](https://github.com/apptainer/apptainer/blob/main/INSTALL.md).
    1. Or use [cvmfsexec](https://github.com/cvmfs/cvmfsexec) to get CVMFS. This is a bit more complex, you can follow the instrictions summarized also in
[this paper](https://indico.cern.ch/event/885212/contributions/4120683/attachments/2181040/3684201/CernVMWorkshopCvmfsExec20210201.pdf).

If user namespaces are not enabled, apptainers/singularity is not installed, and you have no root access to the host, then your use of apptainer/singularity will be very limited even if you install it with one of the three methods above. You need to request to your system administrator to either [install Apptainer/Singularity](https://apptainer.org/docs/user/main/quick_start.html#apptainer-on-a-shared-resource) or to enable [user namespaces](https://apptainer.org/docs/admin/main/user_namespace.html).

{% include links.md %}
