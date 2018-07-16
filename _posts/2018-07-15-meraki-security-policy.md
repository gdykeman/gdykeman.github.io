---
layout: post
title: Grabbing L3 Firewall Policy Information from Meraki
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

```shell
PLAY [GRAB MERAKI SECURITY POLICY OF US AP DEPOT IN CISCO DEVNET] *****************************************************************

TASK [CHECK CONNECTION] ***********************************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "cache_control": "no-cache, no-store, max-age=0, must-revalidate",
        "changed": false,
        "connection": "close",
        "content_type": "application/json; charset=utf-8",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 16 Jul 2018 12:50:41 GMT",
        "expires": "Fri, 01 Jan 1990 00:00:00 GMT",
        "failed": false,
        "json": [
            {
                "id": 537758,
                "name": "mCloud Demo",
                "samlConsumerUrl": "https://n149.meraki.com/saml/login/d029Cc/rtFn7bJtIRna",
                "samlConsumerUrls": [
                    "https://n149.meraki.com/saml/login/d029Cc/rtFn7bJtIRna"
                ]
            },
            {
                "id": 549236,
                "name": "DevNet Sandbox"
            },
            {
                "id": 646829496481089033,
                "name": "Test ENV"
            }
        ],
        "msg": "OK (unknown bytes)",
        "pragma": "no-cache",
        "redirected": true,
        "server": "nginx",
        "status": 200,
        "strict_transport_security": "max-age=15552000; includeSubDomains",
        "transfer_encoding": "chunked",
        "url": "https://n149.meraki.com/api/v0/organizations",
        "vary": "Accept-Encoding",
        "x_frame_options": "sameorigin",
        "x_request_id": "ce0f2dbfa4e1bf2a192191b4c652d6a2",
        "x_runtime": "0.206179",
        "x_ua_compatible": "IE=Edge,chrome=1"
    }
}

TASK [CHANGE MERAKI_ORG TO LOWER] *************************************************************************************************
ok: [localhost]

TASK [GRAB DEVNET SANDBOX ORGANIZATION ID] ****************************************************************************************
skipping: [localhost] => (item={u'samlconsumerurls': [u'https://n149.meraki.com/saml/login/d029cc/rtfn7bjtirna'], u'samlconsumerurl': u'https://n149.meraki.com/saml/login/d029cc/rtfn7bjtirna', u'name': u'mcloud demo', u'id': 537758})
ok: [localhost] => (item={u'id': 549236, u'name': u'devnet sandbox'})
skipping: [localhost] => (item={u'id': 646829496481089033, u'name': u'test env'})

TASK [GRAB ORGANIZATION'S LIST OF NETWORKS] ***************************************************************************************
ok: [localhost]

TASK [GRAB ID OF WIRELESS TYPE AND NAME OF US AP DEPOT] ***************************************************************************
skipping: [localhost] => (item={u'name': u'SM - Corp', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'systems manager', u'id': u'N_646829496481145308'})
skipping: [localhost] => (item={u'name': u'SM - Branch', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'systems manager', u'id': u'N_646829496481145309'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Always On Read Only', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099586'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Small Business Reservable 1', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099587'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Small Business Reservable 2', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099589'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Small Business Reservable 3', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099590'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Small Business Reservable 4', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099592'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'DevNet Small Business Reservable 5', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099593'})
skipping: [localhost] => (item={u'name': u'Camera Depot', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'camera', u'id': u'N_646829496481145361'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'EU AP Depot', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'wireless', u'id': u'N_646829496481145363'})
ok: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'US AP Depot', u'tags': u' Sandbox ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'wireless', u'id': u'N_646829496481145366'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'PythonChallenge', u'tags': u' tag1 tag2 ', u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'combined', u'id': u'L_646829496481099579'})
skipping: [localhost] => (item={u'disableMyMerakiCom': False, u'name': u'WW AP Depot', u'tags': None, u'organizationId': u'549236', u'timeZone': u'America/Los_Angeles', u'type': u'wireless', u'id': u'N_646829496481146258'})

TASK [GRAB SSID OF NETWORK] *******************************************************************************************************
ok: [localhost]

TASK [GRAB ID OF US AP DEPOT WIFI] ************************************************************************************************
ok: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': True, u'number': 0, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'US AP Depot WiFi'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 1, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 2'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 2, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 3'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 3, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 4'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 4, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 5'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 5, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 6'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 6, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 7'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 7, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 8'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 8, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 9'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 9, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 10'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 10, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 11'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 11, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 12'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 12, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 13'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 13, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 14'})
skipping: [localhost] => (item={u'ipAssignmentMode': u'NAT mode', u'splashPage': u'None', u'perClientBandwidthLimitDown': 0, u'enabled': False, u'number': 14, u'perClientBandwidthLimitUp': 0, u'ssidAdminAccessible': False, u'bandSelection': u'Dual band operation', u'minBitrate': 11, u'authMode': u'open', u'name': u'Unconfigured SSID 15'})

TASK [GRAB L3 FIREWALL POLICIES] **************************************************************************************************
ok: [localhost]

TASK [debug] **********************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "cache_control": "no-cache, no-store, max-age=0, must-revalidate",
        "changed": false,
        "connection": "close",
        "content_type": "application/json; charset=utf-8",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 16 Jul 2018 12:50:47 GMT",
        "expires": "Fri, 01 Jan 1990 00:00:00 GMT",
        "failed": false,
        "json": [
            {
                "comment": "Wireless clients accessing LAN",
                "destCidr": "Local LAN",
                "destPort": "Any",
                "policy": "deny",
                "protocol": "Any"
            },
            {
                "comment": "Default rule",
                "destCidr": "Any",
                "destPort": "Any",
                "policy": "allow",
                "protocol": "Any"
            }
        ],
        "msg": "OK (unknown bytes)",
        "pragma": "no-cache",
        "redirected": true,
        "server": "nginx",
        "status": 200,
        "strict_transport_security": "max-age=15552000; includeSubDomains",
        "transfer_encoding": "chunked",
        "url": "https://n149.meraki.com/api/v0/networks/N_646829496481145366/ssids/0/l3FirewallRules",
        "vary": "Accept-Encoding",
        "x_frame_options": "sameorigin",
        "x_request_id": "d9ce7b26523317d4eb7f5be62a06f3d0",
        "x_runtime": "0.354606",
        "x_ua_compatible": "IE=Edge,chrome=1"
    }
}

PLAY RECAP ************************************************************************************************************************
localhost                  : ok=10   changed=0    unreachable=0    failed=0

```
Hopefully this was helpful and it opens your imagination to other things that you can accomplish via Ansible!
