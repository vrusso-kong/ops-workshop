+++
title = ""
menuTitle = "Deploy BOSH Release"
chapter = false
weight = 6
description = ""
draft = false
+++
## Goal

Before we can deploy a BOSH release we need to define how the release should be deployed. Do we deploy 3 or 1 instances? Do we give it an IP of 10.0.0.1 or 10.0.0.50? etc...
As you can see there are many different ways to deploy the release. With BOSH we call this definition a BOSH Manifest. Lets create a manifest and deploy our sample-bosh-release.

## Part 1: Create the manifest

Using your favorite command-line text editor to create a `sample-bosh-manifest.yml` text file.

    name: sample-bosh-deployment
    
    releases:
    - {name: bosh-release, version: latest}
    
    stemcells:
    - alias: trusty
      os: ubuntu-trusty
      version: latest
    
    instance_groups:
    - name: sample_vm
      instances: 1
      networks:
      - name: default
      azs: [z1]
      jobs:
      - name: sample_job
        release: bosh-release
      stemcell: trusty
      vm_type: default
    update:
      canaries: 1
      max_in_flight: 10
      canary_watch_time: 1000-100000
      update_watch_time: 1000-100000


  - The goal of this deployment is to deploy a single VM that runs our sample_job.

  - `name` - the unique name of this deployment within a BOSH director. When a BOSH director receives subsequent deployment manifests with the same name it will assume it is an upgrade of the existing deployment.

  - `releases` - lists the specific BOSH release versions that are to be used, ie the one we just made.

  - `instance_groups` - lists the sets of instances (VMs) that will run the same job templates/packages as each other. Instance groups will be deployed as long running instances by default.

  - `stemcells` - lists the stemcells (discussed earlier) we will run our instances with.

## Part 2: Deploy!

1. Here is where all the magic happens! By BOSH `deploy`ing we combine the Stemcells, Releases, and Manifest to create our software system. This can take a few minutes.

    `$ bosh -d sample-bosh-deployment deploy sample-bosh-manifest.yml`

  - If successful, you should see something similar to the following:

            Task 80

            Task 80 | 18:00:44 | Preparing deployment: Preparing deployment (00:00:00)
            Task 80 | 18:00:44 | Preparing package compilation: Finding packages to compile (00:00:00)
            Task 80 | 18:00:44 | Compiling packages: sample_app/3698d0cc632d3162a6a2fedcd36ac00364a7cd64 (00:00:57)
            Task 80 | 18:04:39 | Creating missing vms: sample_vm/71160b1a-faaa-487c-b107-7d4ed8fce7ce (0)
            Task 80 | 18:05:28 | Updating instance sample_vm: sample_vm/71160b1a-faaa-487c-b107-7d4ed8fce7ce (0) (canary)

            Task 80 Started  Mon Nov 27 18:00:44 UTC 2017
            Task 80 Finished Mon Nov 27 18:05:46 UTC 2017
            Task 80 Duration 00:05:02
            Task 80 done

            Succeeded
