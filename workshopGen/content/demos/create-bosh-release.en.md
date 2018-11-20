+++
title = ""
menuTitle = "Create BOSH Release"
chapter = false
weight = 5
description = ""
draft = false
+++
## Goal

Deploying software systems with BOSH is done with BOSH Releases. Releases abstract code away from the underlying OS and create a specific packing structure all software systems must adhere to. Lets explore what a BOSH Release is by using a very simple release that prints log messages.

## Create the BOSH Release

To create the release directory, navigate into the workspace where you want the release to be, and run:
- `$ bosh init-release --dir <release_name>`

If you have `tree` (Hint: `$ brew install tree`) you can view the directory structure:

    tree .
    .
    ├── config
    │   ├── blobs.yml
    │   └── final.yml
    ├── jobs
    ├── packages
    └── src
    
    4 directories, 2 files
    
For this deployment we are going to make a very simple app that prints to `STDOUT`.

### Create Source Code
Inside the src folder make a new dir called sample_app 
        
    $ cd src/
    $ mkdir sample_app
    $ cd sample_app/
        
Using your favorite editor, create the `app`, add the source code for the script.

    #!/bin/bash
    while true
    do
        echo "Sample APP STDOUT!!!"
        sleep 1
    done        

### Create the Job
Navigate back to the release directory create the job skeleton
- `$ bosh generate-job sample_job`
    
Once again `tree` will show use the new files and folders generated
       
    tree .
    .
    ├── config
    │   ├── blobs.yml
    │   └── final.yml
    ├── jobs
    │   └── sample_job
    │       ├── monit
    │       ├── spec
    │       └── templates
    ├── packages
    └── src
       └── sample_app
           └── app
    
    7 directories, 5 files
       
A control script is used to tell the job how to start and stop. Navigate to the `templates` directory and create `ctl.erb` with an editor
 
    #!/bin/bash
    
    RUN_DIR=/var/vcap/sys/run/sample_job
    LOG_DIR=/var/vcap/sys/log/sample_job
    PIDFILE=${RUN_DIR}/pid
    
    case $1 in
    
      start)
        mkdir -p $RUN_DIR $LOG_DIR
        chown -R vcap:vcap $RUN_DIR $LOG_DIR
    
        echo $$ > $PIDFILE
    
        cd /var/vcap/packages/sample_app
    
         exec ./app \
          >>  $LOG_DIR/sample_app.stdout.log \
          2>> $LOG_DIR/sample_app.stderr.log
    
        ;;
    
      stop)
        kill -9 `cat $PIDFILE`
        rm -f $PIDFILE
    
        ;;
    
      *)
        echo "Usage: ctl {start|stop}" ;;
    
    esac
        
In order for BOSH to monitor the health of our running job it utilizes  [monit](https://en.wikipedia.org/wiki/Monit).
   
To configure monit for our process we create a monit file that explains what processid corresponds to our application we are asking to be monitored and how to start/stop the application.
    
    check process sample_app
      with pidfile /var/vcap/sys/run/sample_job/pid
      start program "/var/vcap/jobs/sample_job/bin/ctl start"
      stop program "/var/vcap/jobs/sample_job/bin/ctl stop"
      group vcap

BOSH requires a monit file for each job in a release. When developing a release, you can use an empty monit file to meet this requirement without having to first create a control script.

At compile time, BOSH transforms templates into files, which it then replicates on the job VMs. The template names and file paths are among the metadata for each job that resides in the job spec file. 

Edit the `spec` file:
    
    ---
    name: sample_job
    
    templates:
      ctl.erb: bin/ctl
    
    packages:
    - sample_app
    
    properties: {}
        
Our `sample_job` has a dependency on the `sample_app` so we will create the package to encapsulate that in the next section.
     
### Create the Package

Once again we navigate back to the release directory and run the generate command

    $ bosh generate-package sample_app
 
And tree
 
    tree .
    .
    ├── config
    │   ├── blobs.yml
    │   └── final.yml
    ├── jobs
    │   └── sample_job
    │       ├── monit
    │       ├── spec
    │       └── templates
    │           └── ctl.erb
    ├── packages
    │   └── sample_app
    │       ├── packaging
    │       └── spec
    └── src
    └── sample_app
        └── app
    
    8 directories, 8 files
   
Let's update the spec, since our app only uses it's source code, we can use the globbing pattern `<package_name>/**/*` to deep-traverse the directory in `src` where the source code should reside.

    ---
    name: sample_app
    
    dependencies: []
    
    files:
    - sample_app/**/*
        
Since this release only has a bash script which does not require compilation we will just copy the script into the compiled code location allow us to use later.
     
    set -e -x
     
    cp -a sample_app/* ${BOSH_INSTALL_TARGET}
 
We have all the pieces of our bosh release. We just need to use bosh to create and upload the release. We need to use --force because by default BOSH requires a blob to already exist. We don't need a blob for our sample_app.
- `$ bosh create-release --force`
        
           Adding package 'sample_app/e48b68edccfc35fc9bbd1fe37b652fd3bb261cb9'...
           Adding job 'sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9'...
           Added package 'sample_app/e48b68edccfc35fc9bbd1fe37b652fd3bb261cb9'
           Added job 'sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9'
           
           Added dev release 'bosh-release/0+dev.1'
            
           Name         bosh-release  
           Version      0+dev.1  
           Commit Hash  non-git  
            
           Job                                                  Digest                                    Packages  
           sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9  98c11bd46321008ed0802a75f5d26d3d05addb62  sample_app  
            
           1 jobs
           
           Package                                              Digest                                    Dependencies  
           sample_app/e48b68edccfc35fc9bbd1fe37b652fd3bb261cb9  e32a56fd1fb3fa819d18eda5f4342d7fac2c3cf0  -  
            
           1 packages
            
           Succeeded
       
- `$ bosh upload-release`
    
          Using environment 'https://10.0.0.6:25555' as client 'admin'
            
          ########################################################## 100.00% 2.08 KiB/s 0s
          Task 21
            
          Task 21 | 00:17:50 | Extracting release: Extracting release (00:00:00)
          Task 21 | 00:17:50 | Verifying manifest: Verifying manifest (00:00:00)
          Task 21 | 00:17:50 | Resolving package dependencies: Resolving package dependencies (00:00:00)
          Task 21 | 00:17:50 | Creating new packages: sample_app/e48b68edccfc35fc9bbd1fe37b652fd3bb261cb9 (00:00:00)
          Task 21 | 00:17:50 | Creating new jobs: sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9 (00:00:00)
          Task 21 | 00:17:50 | Release has been created: bosh-release/0+dev.1 (00:00:00)
            
          Task 21 Started  Thu Nov 15 00:17:50 UTC 2018
          Task 21 Finished Thu Nov 15 00:17:50 UTC 2018
          Task 21 Duration 00:00:00
          Task 21 done
            
          Succeeded

- `$ bosh releases`
        
          Using environment 'https://10.0.0.6:25555' as client 'admin'
            
          Name          Version  Commit Hash  
          bosh-dns      1.10.0*  7c6515f  
          bosh-release  0+dev.1  non-git  
          gogs          5.4.0*   ca84293  
            
          (*) Currently deployed
          (+) Uncommitted changes
            
          3 releases
            
          Succeeded
      
That's it! You have now seen all the parts of a full BOSH release! Now that we have our release, we need to create a manifest to deploy it.


