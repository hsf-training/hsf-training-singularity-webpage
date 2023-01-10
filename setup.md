---
title: Setup
---

In this document we use Apptainer and Singularity interchangeably. See the [Introduction]({{ page.root }}{% link _episodes/01-introduction.md %})
for more details about existing Apptainer and Singularity versions and the differences between them.

## Option 1: Use pre-installed singularity on a cluster

Apptainer/Singularity has become popular and usually it is available in the institutional computing resources.
Check if singularity is available with
```bash
singularity --version
```
If installed, you will see `apptainer version ...` or `singularity version ...`, depending on the flavor installed.
If it is not in your PATH you may still be able to use it via CVMFS: check if you have user namespaces enabled and CVMFS to run singularity that way:
```bash
[[ $(cat /proc/sys/user/max_user_namespaces) -gt 0 ]] && ls /cvmfs/oasis.opensciencegrid.org/mis/ &>/dev/null && { export PATH=/cvmfs/oasis.opensciencegrid.org/mis/apptainer/bin/:"$PATH"; echo "Added to PATH"; singularity --version; } || echo "Unable to run Singularity/Apptainer via CVMFS"
```
If this works, it will be added to your path and you will see your singularity/apptainer version.

If your local computing system does not have Apptainer/Singularity installed, you may
[request it to your system administrator as suggested here](https://apptainer.org/docs/user/main/quick_start.html#apptainer-on-a-shared-resource).

## Option 2: Install singularity/Apptainer

You will need a **Linux system (including WSL on Windows computers)** to run Apptainer/Singularity natively.
**MacOS is not supported**.


### If you have root access

It is easiest to
[install if you have root access](https://apptainer.org/docs/user/main/quick_start.html#quick-installation).

### If not

If the above is not possible and you cannot use the CVMFS distribution you have still an option if user namespace is enabled on your system:
1. Check if user namespaces are enabled  (`cat /proc/sys/user/max_user_namespaces` is bigger than 0):
    ```bash
    [[ $(cat /proc/sys/user/max_user_namespaces) -gt 0 ]] && \
        echo "User namespaces enabled, continue the Apptainer installation" || \
        echo ""User namespaces NOT enabled, your use of Apptainer will be very limited"
    ```
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
