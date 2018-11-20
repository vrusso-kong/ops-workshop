+++
title = ""
menuTitle = "Example Stemcell"
chapter = false
weight = 7
description = ""
draft = false
+++
## Goal

BOSH Stemcells contain both metadata and the raw image. Lets take some time to examine what is contained in the metadata and how a Stemcell is organized.

## Prerequisites

1. A file archiver to extract files with.

    Windows - [7 Zip](http://www.7-zip.org/)  
    Mac/Linux - [Tar](https://superuser.com/a/46521) - (Factory Installed)

2. OPTIONAL - [VirtualBox](https://www.virtualbox.org/wiki/Downloads) to boot a raw VMDK system image.

## Part 1: Extracting the stemcell image

1. Download the latest vSphere Stemcell [here](http://bosh.io/stemcells/bosh-azure-hyperv-ubuntu-trusty-go_agent)
2. Extract the downloaded stemcell

    `tar -xvf bosh-stemcell-*-azure-hyperv-ubuntu-trusty-go_agent.tgz`
3. Take a look at the contents of the `stemcell.MF` file

    `cat stemcell.MF`

    Notice versioning, and specific metadata for the stemcell

4. Take a look at the contents of the `stemcell_dpkg_l.txt` file

    `cat stemcell_dpkg_l.txt`

    Notice the utilities which come pre-installed on the stemcell

## Part 2 (OPTIONAL): Booting the stemcell image

The follow steps are OPTIONAL, and recommended only if you have VirtualBox installed.

5. Rename the `image` file to `image.tgz`

    `mv image image.tgz`

6. Extract the `image.tgz` file

    `tar -xvf image.tgz`

    Notice the raw operating system image in the vmdk format, since this is a vSphere Image.

7. Launch a vm based on the `image-disk1.vmdk` inside VirtualBox

1. Login with `vcap` and `c1oudc0w`
