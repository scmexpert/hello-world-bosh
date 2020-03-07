# hello-world-bosh
Hello World Bosh deployment
Use the BOSH command line to initialize a release:

`bosh init-release --dir hello-world-release --git`

This will create a directory called hello-world-release. Take a look at the files inside it. Note that it is also a Git repository.

Our “Hello world!” release needs a BOSH job. The BOSH command line can generate a job skeleton for you:

Inside the release directory, run the comand

bosh generate-job hello-world

This will create a directory called jobs/hello-world for you. Take a look at the files inside.

Create a file called `jobs/hello-world/templates/run.sh`
```
#!/usr/bin/env bash

echo "Hello world!"
```
Edit the job spec file `jobs/hello-world/spec` to tell BOSH about the script. Change the templates stanza to stay:
```
templates:
  run.sh: bin/run
```
A BOSH job with bin/run is called an errand.

To test our work, we can create and upload a dev release:

`bosh create-release --force`

Without the --force flag, BOSH will warn us that we have not committed our changes. We want to test our work before we commit it.

`bosh upload-release`

In order to use the release, we need to add it to a BOSH deployment. We did this on GCP, but with minor modifications it will work on any other IaaS.

First we need to upload a stemcell, which our release will be installed on:

`bosh upload-stemcell https://bosh.io/d/stemcells/bosh-google-kvm-ubuntu-trusty-go_agent`

Then we create a manifest.yml to describe the deployment. Here’s an example of the one that we used:
```
name: hello-world-deployment

releases:
- name: hello-world
  version: latest

stemcells:
- alias: default
  os: ubuntu-trusty
  version: latest

update:
  canaries: 1
  max_in_flight: 1
  canary_watch_time: 1000-30000
  update_watch_time: 1000-30000

instance_groups:
- name: server
  azs: [z1]
  instances: 1
  vm_type: default
  stemcell: default
  networks:
  - name: default
  jobs:
  - name: hello-world
    release: hello-world
    templates:
    - name: hello-world
      
```
Deploy the release with the deployment manifest

`bosh -d hello-world-deployment deploy manifest.yml`

We can see the errand job that we created with the comand:

`bosh -d hello-world-deployment errands`

You should see an errand called hello-world, which we can run with the command:

`bosh -d hello-world-deployment run-errand hello-world`

You should see the text “Hello world!” as standard out (stdout) from the errand.

Congratulations! You’ve created a “Hello world!” BOSH release. 
