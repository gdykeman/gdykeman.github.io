---
layout: post
title: Running ansible at scale
comments: true
---
Last week I deployed my first "at scale" playbook. The overall objective was simple: Add new dhcp helper address to about 400 switches.
Like most things though, the devil is in the details. Right off the bat, I ran into "non-ansible" related issues (tacacs/ssh).
That brought the number of devices to about 270. Not a big number right?
At the most basic level, yes, if I was simply pushing configs using the ios\_config
module.
##Breakdown of the playbook
1. Execute a show run on the device and compile a local backup for each device
2. Run a pre-flight report, specific to the interfaces we are going to impact (multiple ssh sessions per host)
3. Build the configs locally
4. Deploy the configs
5. Validate the configs/Unit testing

<!--more-->

### A bit more about the unit testing:
For testing the changes were deployed, I had 2 criteria:

1. Assert that the new helpers are present within the interface configurations (of the specific interfaces)
2. Assert that the startup and running config are in sync (in other word, the new config has been saved)

For assertion 1, I took the approach of running a show running interface per interface - this implied multiple ssh sessions per host.

##Observations:
1. Running the playbook for backups result in a lot of SSH connection failures on the first run. Subsequent runs are significantly more successful - Still see some failures
2. Running the playbook for the preflight report/validation, results in ssh timeouts - these are not consistent across hosts: Meaning, for the same host, the show run int
 for Vlan101 will work but might fail for Vlan201 on the first run, but on the next run, there is no guarantee that a repeat play will reproduce this exact failure
3. My validation role uses dynamic includes like this [example](https://github.com/termlen0/ansible_dynamic_include_bug_demo). Running the playbook with a tag other
than "validate", still attempts to load all yaml files and results in failure.

##Tweaks and next steps:
1. I had mixed success with using the "serial" [option](http://docs.ansible.com/ansible/playbooks_delegation.html#rolling-update-batch-size). Needs more trial and error.
2. Rewrite pre-flight and validation scripts to only do a single show run. Collect specific interface details from a local copy
3. Tried pipelining and some other recommendations that seemed relevant based on [this](https://www.ansible.com/blog/ansible-performance-tuning).
However, it appears to be focussed on using ssh connections to the remote systems. As we know, for the ios\_\* modules, ansible ssh'es to the local host and then uses paramiko
within the modules. In short, pipeline did not seem to do much
4. Setting the timeout parameter for the ios\_\* modules seemed to have no affect on the ssh timeouts.
5. To further understand observation 1 (which is still the most vexing one), I tried connecting to the device using paramiko from the python interpreter and executing
the same commands. I could not recreate the issue. I had good connectivity each time

##Other issues:
For observation 3, I opened an [issue](https://github.com/ansible/ansible/issues/19345) with ansible. Based on the comments it was closed with, I guess, it is an expected
behavior. Which means, for any playbook that has dynamic includes, we need to remember to send any variables that role will need, even though your tags may not be calling
that role.
