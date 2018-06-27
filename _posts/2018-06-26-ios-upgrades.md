---
layout: post
title: Upgrading Cisco IOS Devices with Ansible
comments: true
---

Upgrading network images to stable and or later versions is nothing new in the networking world.  A recent discussion with a customer, however, encouraged the creation of a simple, yet effective playbook to help automate this process.

The two use cases were around:  
1. CVE.  A vulnerability is reported and an image upgrade is required to resolve it.

2. Compliance checking.  Ensure that the networking devices are at the compliant and approved version of code, and if not, make sure that it is.

At the time, the customer would SSH into each device to upgrade the image, wait for it to boot back up, check to make sure the image was upgraded to the correct version, then move onto the next device.  In the case of the vulnerability, this became an all hands on deck activity that resulted in many hours from multiple engineers to patch the network devices.

Now, let’s take a look at an Ansible Playbook that can automate this process.
```yaml
{%raw%}
---
- name: UPGRADE ROUTER FIRMWARE
  hosts: routers
  connection: network_cli
  gather_facts: no

  vars:
    compliant_ios_version: 16.08.01a

  tasks:
    - name: GATHER ROUTER FACTS
      ios_facts:

    - name: UPGRADE IOS IMAGE IF NOT COMPLIANT
      block:
      - name: COPY OVER IOS IMAGE
        command: "scp system-image-filename.bin {{inventory_hostname}}:/system-image-filename.bin"

      - name: SET BOOT SYSTEM FLASH
        ios_config:
          commands:
            - "boot system flash:system-image-filename.bin"

      - name: REBOOT ROUTER
        ios_command:
          commands:
            - "reload\n"

      - name: WAIT FOR ROUTER TO RETURN
        wait_for:
          host: "{{inventory_hostname}}"
          port: 22
          delay: 60
        delegate_to: localhost

      when: ansible_net_version != compliant_ios_version

    - name: GATHER ROUTER FACTS FOR VERIFICATION
      ios_facts:

    - name: ASSERT THAT THE IOS VERSION IS CORRECT
      assert:
        that:
          - compliant_ios_version == ansible_net_version
{%endraw%}
```
Let's step through the components!  

First, the playbook level variables:  
```yaml
---
- name: UPGRADE ROUTER FIRMWARE
  hosts: routers
  connection: network_cli
  gather_facts: no

  vars:
    compliant_ios_version: 16.08.01a

  tasks:
```
We're passing in the following:  
`name` - Naming our playbook.  
`hosts` - What nodes are we targeting? In this case, we are targeting a group called routers.  
`connection` - Defining the connection type.  
`gather_facts` - This is a server specific task and does not apply to us.  
`vars` - Defining a variable that we will use to compare IOS versions.  
`tasks` - Where we will list the tasks we want the playbook to accomplish.

The first task:
```yaml
- name: GATHER ROUTER FACTS
  ios_facts:
```
The above tasks runs the ios_facts module which collects facts from remote devices running Cisco IOS.  
[ios_facts_module](https://docs.ansible.com/ansible/2.5/modules/ios_facts_module.html)

The `ios_facts` module provides us with the `ansible_net_version` which defines the operating system version running on the remote device.  We'll be using this as conditional logic in our proceeding tasks.  

Next, we create a `block` which contains a list of tasks.  We put a conditional on this `block` with a `when` statement near the bottom of the playbook - or at the end of the `block`.  The `when` statement does a check to see if the current version of the remote device is *NOT* equal to the version we specify.  If it's *NOT* equal, the tasks in the block will execute and proceed to upgrade the remote device to the version of code that's required.
```yaml
{%raw%}
- name: UPGRADE IOS IMAGE IF NOT COMPLIANT
  block:
  - name: COPY OVER IOS IMAGE
    command: "scp system-image-filename.bin {{inventory_hostname}}:/system-image-filename.bin"

  - name: SET BOOT SYSTEM FLASH
    ios_config:
      commands:
        - "boot system flash:system-image-filename.bin"

  - name: REBOOT ROUTER
    ios_command:
      commands:
        - "reload\n"

  - name: WAIT FOR ROUTER TO RETURN
    wait_for:
      host: "{{inventory_hostname}}"
      port: 22
      delay: 60
    delegate_to: localhost

  when: ansible_net_version != compliant_ios_version
{%endraw%}
```

The first task within the block is to copy over the Cisco IOS image to the remote device.  We utilize the `SCP` command to do so.

Next, we utilize the `ios_config` module to set the boot system of the remote device so it'll use the image the new image if the version is not compliant or what we want it to be.

We are now ready to reboot the remote device so it boots into the correct image.  We pass in **"reload\n"** into the `ios_command` to perform the reload.

The next task, as the name suggests, `waits for` the remote device to reachable again.  We specify the port we want to test reachability against and an initial delay before we begin to poll for reachability.  Once the remote device is reachable, we do a final verification of the version.

```yaml
{%raw%}
- name: GATHER ROUTER FACTS FOR VERIFICATION
  ios_facts:

- name: ASSERT THAT THE IOS VERSION IS CORRECT
  assert:
    that:
      - compliant_ios_version == ansible_net_version
{%endraw%}
```
With the remote device having been rebooted, we execute the `ios_facts` module again to capture the new `ansible_net_version`.

Finally, we use the `assert` module to ensure that the upgrade was successful and the remote device is running the compliant IOS version.  By using the `assert` module, we make the playbook fail if it is *NOT* the correct version.
