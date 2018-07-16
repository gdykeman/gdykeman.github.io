---
layout: post
title: Grabbing L3 Firewall Policy Information from Cisco Meraki
comments: true
---

Had a friend of mine from Cisco ask about how/if Ansible can work with Cisco Meraki.  As with most answers with "can Ansible do this" my initial response was, of course!

Quick background:
Cisco Meraki is one of the largest LAN SDN infrastructures today. Using their open APIs, we sat down and wrote an Ansible playbook around checking and provisioning policies on a single access point. The goal of this exercise is to eventually tie together other security products and define specific security policies to deploy across a Cisco Meraki Infrastructure.

Here's a look at the playbook in its entirety:

```yaml
{%raw%}
---
- name: GRAB MERAKI SECURITY POLICY OF US AP DEPOT IN CISCO DEVNET
  hosts: localhost
  connection: local
  gather_facts: no

  vars_files:
    - ./vars.yml

  tasks:
    - name: CHECK CONNECTION
      uri:
        url: https://api.meraki.com/api/v0/organizations
        method: GET
        headers:
          X-Cisco-Meraki-API-Key: "{{ MERAKI_API_KEY }}"
      register: meraki_org

    - name: CHANGE MERAKI_ORG TO LOWER
      set_fact:
        meraki_org_lower: "{{ meraki_org.json | lower }}"

    - name: GRAB DEVNET SANDBOX ORGANIZATION ID
      set_fact:
        meraki_org_id: "{{item.id}}"
      with_items: "{{meraki_org_lower}}"
      when: item.name == 'devnet sandbox'

    - name: GRAB ORGANIZATION'S LIST OF NETWORKS
      uri:
        url: https://api.meraki.com/api/v0/organizations/{{meraki_org_id}}/networks
        method: GET
        headers:
          X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
      register: meraki_network

    - name: GRAB ID OF WIRELESS TYPE AND NAME OF US AP DEPOT
      set_fact:
        meraki_network_id: "{{ item.id }}"
      with_items: "{{ meraki_network.json }}"
      when: item.type == 'wireless' and item.name == 'US AP Depot'

    - name: GRAB SSID OF NETWORK
      uri:
        url: https://api.meraki.com/api/v0/networks/{{meraki_network_id}}/ssids
        method: GET
        headers:
          X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
      register: meraki_ssid

    - name: GRAB ID OF US AP DEPOT WIFI
      set_fact:
        meraki_ssid_number: "{{item.number}}"
      with_items: "{{ meraki_ssid.json}}"
      when: item.name == 'US AP Depot WiFi'

    - name: GRAB L3 FIREWALL POLICIES
      uri:
        url: https://api.meraki.com/api/v0/networks/{{meraki_network_id}}/ssids/{{meraki_ssid_number}}/l3FirewallRules
        method: GET
        headers:
          X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
      register: meraki_l3firewallrules

    - debug:
        msg: "{{meraki_l3firewallrules}}"
{%endraw%}
```
Now, let's step through the tasks!

The first task is to grab the ID of the "DevNet SandBox" organization which is a Cisco provided sandbox environment.


```yaml
{%raw%}
- name: CHECK CONNECTION
  uri:
    url: https://api.meraki.com/api/v0/organizations
    method: GET
    headers:
      X-Cisco-Meraki-API-Key: "{{ MERAKI_API_KEY }}"
  register: meraki_org

- name: CHANGE MERAKI_ORG TO LOWER
  set_fact:
    meraki_org_lower: "{{ meraki_org.json | lower }}"

- name: GRAB DEVNET SANDBOX ORGANIZATION ID
  set_fact:
    meraki_org_id: "{{item.id}}"
  with_items: "{{meraki_org_lower}}"
  when: item.name == 'devnet sandbox'
{%endraw%}
```

Here we use the `uri` module to query the Cisco Meraki API in order to, first, grab the organizations that exist, and from there, grab the organization ID that has the name of "DevNet SandBox."  We have an additional task to utilize the `set_fact` module to alter the registered variable to be lowercase so we don't miss the name due to a capitalized letter here and there.  With this task, we now have the organization ID, which we will use in the next task to point to the *new* API endpoint.

**NOTE**: Many debug statements were used to figure out the key value pair, no animals were harmed.

```yaml
{%raw%}
- name: GRAB ORGANIZATION'S LIST OF NETWORKS
  uri:
    url: https://api.meraki.com/api/v0/organizations/{{meraki_org_id}}/networks
    method: GET
    headers:
      X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
  register: meraki_network

- name: GRAB ID OF TYPE WIRELESS AND NAME OF US AP DEPOT
  set_fact:
    meraki_network_id: "{{ item.id }}"
  with_items: "{{ meraki_network.json }}"
  when: item.type == 'wireless' and item.name == 'US AP Depot'
{%endraw%}
```

Now, an organization contains a list of networks.  For our example, we want to grab a network that is of type "wireless," but also having the name of "US AP Depot."  The type and name used in this example is specific to the Cisco DevNet Sandbox, but could easily be applied to any Cisco Meraki environment.

We now have the network ID of a specific wireless network named "US AP Depot," which we will use to get the SSID of a specific Access Point (AP).

```yaml
{%raw%}
- name: GRAB SSID OF NETWORK
  uri:
    url: https://api.meraki.com/api/v0/networks/{{meraki_network_id}}/ssids
    method: GET
    headers:
      X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
  register: meraki_ssid

- name: GRAB ID OF US AP DEPOT WIFI
  set_fact:
    meraki_ssid_number: "{{item.number}}"
  with_items: "{{ meraki_ssid.json}}"
  when: item.name == 'US AP Depot WiFi'
{%endraw%}
```

Next, we use the `meraki_network_id` from the previous task to grab the SSID's for the `US AP Depot` network.  We also add a conditional `when` statement to ensure that we are grabbing the correct SSID, with the conditional being that the name equals `US AP Depot WiFi`

```yaml
{%raw%}
- name: GRAB L3 FIREWALL POLICIES
  uri:
    url: https://api.meraki.com/api/v0/networks/{{meraki_network_id}}/ssids/{{meraki_ssid_number}}/l3FirewallRules
    method: GET
    headers:
      X-Cisco-Meraki-API-Key: "{{MERAKI_API_KEY}}"
  register: meraki_l3firewallrules

- debug:
    msg: "{{meraki_l3firewallrules}}"
{%endraw%}
```

Lastly, we use the SSID number from the previous task to grab the L3 Firewall Policies from our Cisco Meraki Infrastructure.  In summary, we were able to dive down from the organization to the access point and retrieve a security policy.
As an extra step, if we change the URI request to a 'PUT' method and add a body, we can push/change the security policies.


Hopefully this was helpful and it opens your imagination to other things that you can accomplish via Ansible!
