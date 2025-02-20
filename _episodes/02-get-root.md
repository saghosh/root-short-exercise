---
title: "Get ROOT"
questions:
- "How to install ROOT on my system or get access to systems with ROOT pre-installed?"
objectives:
- "Install ROOT"
keypoints:
- "ROOT is accessible in several different ways"
- "Detailed up-to-date instructions can be found at [https://root.cern/install](https://root.cern/install)"
---

> This section shows you multiple ways to get ROOT. Find below solutions to run ROOT locally, on LXPLUS, in batch systems and CIs!
{: .testimonial}

## Conda

[Conda](https://docs.conda.io/en/latest/) is a package management system and environment management system, which can install ROOT in a few minutes on Linux and MacOS.

The fastes way to get ROOT is [installing Miniconda](https://docs.conda.io/en/latest/miniconda.html) (a minimal Conda installation) and then run the following two commands:

```bash
conda create -c conda-forge --name <my-environment> root
conda activate <my-environment>
```

You can find detailed information about ROOT on Conda in [this blog post](https://iscinumpy.gitlab.io/post/root-conda/).

## CVMFS

[CVMFS](https://cernvm.cern.ch/portal/filesystem) is a software distribution service that is already set up on many HEP systems. CMSSW including ROOT is distributed via CVMFS, but also other software stacks are available that contain ROOT.

Most notably, the CERN service [LXPLUS](http://information-technology.web.cern.ch/services/lxplus-service) has CVMFS always installed and enables rapid access to software and computing resources.



## Docker

If you want to use ROOT in a CI system (e.g. GitLab pipelines or GitHub actions), most likely the software will be made available via Docker. The official ROOT docker containers can be found at [https://hub.docker.com/r/rootproject/root](https://hub.docker.com/r/rootproject/root). The different base images and ROOT versions are encoded in the tags, for example `6.22.00-ubuntu20.04`, and `latest` will get you the latest ROOT release (v6.22) based on Ubuntu 20.04. If you want to try it, [get Docker](https://docs.docker.com/get-docker/) and run the following command to start the container with a bash shell.

```bash
docker run --rm -it rootproject/root /bin/bash
```

## Binary releases and packages

The classic way to distribute software, besides the source code, are plain binary releases. You can download these from the release pages on [https://root.cern/install/all_releases](https://root.cern/install/all_releases) for all major MacOS and Linux versions. If you choose this installation method, make sure [ROOT dependencies](https://root.cern/install/dependencies) are installed on your system. Complete installation instructions for binary releases are available [here](https://root.cern/install/#download-a-pre-compiled-binary-distribution).

In addition, for some Linux distributions, the ROOT community maintains packages in the respective package managers. You can find a list of maintained packages at [https://root.cern/install/#linux-package-managers](https://root.cern/install/#linux-package-managers).

## Verify the ROOT version

Since ROOT has a long history and numerous releases, on old systems such Scientific Linux 6 you may find correspondingly old ROOT versions. However, with the following commands you can easily verify your ROOT version and also find expert details about the ROOT configuration!

```bash
# ROOT version and build tag
root --version

# Again the ROOT version (this also works with older ROOT versions)
root-config --version

# Check that ROOT was built with C++14 support
# The output must contain one of -std=c++{14,17,1z} so that all code examples of this lesson run!
root-config --cflags

# List all the ROOT configuration options that can be checked
root-config --help
```

> ## Find your way to access ROOT!
> For the exercises later you need at least ROOT 6.18 and C++14 support. Feel free to set up for yourself your preferred environment satisfying this requirement!
{: .challenge}


> ## Fallback solution
> As a fallback solution you can always connect to LXPLUS via `ssh -Y your_username@lxplus.cern.ch`. The `-Y` flag enables X forwarding, which allows you to forward the output of graphics application in case you run a system with an X server such as almost all Linux distributions.
{: .solution}

{% include links.md %}
