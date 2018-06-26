---
layout: post
title: NETCONF Ain't YANG
comments: true
---

Obvious!

Recently I was working on  YANG device models, using NETCONF to
interact with the actual device (NXOS 7.0.3.I6.1). I was using
Cisco's
[NX-OS Programmability Guide](http://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/programmability/guide/b_Cisco_Nexus_9000_Series_NX-OS_Programmability_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_Programmability_Guide_7x_chapter_010100.html) for
the 7.x version. I have had previous experience working with
YANG and Cisco's NEDs through the tail-f/NSO product and was feeling
quite confident in my understanding of how YANG is being used for
modeling (Either a service or a device) and how NETCONF was one of the
protocols used to interact with the model. 
However, I was getting lost trying to navigate the NXOS's YANG model
over the NETCONF client. 
<!--more-->
I first attempted to collect the device
capabilities offered by the NETCONF `HELLO` and then collect the vlans
configured on the device. Here is a breakdown of my flawed attempt.

1. I constructed the connectivity parameters as follows:

``` python
from ncclient import manager

nc_params = {'allow_agent': False,
 'device_params': {'name': 'nexus'},
 'host': '172.16.30.100',
 'hostkey_verify': False,
 'look_for_keys': False,
 'password': 'admin',
 'port': '22',
 'username': 'admin'}

``` 

2. Established connectivity to the device and collected the capabilities

``` python
ncm = manager.connect(**nc_params)

In [81]: for capability in ncm.server_capabilities:
   ....:     print capability
   ....:     
urn:ietf:params:netconf:capability:writable-running:1.0
urn:ietf:params:netconf:capability:rollback-on-error:1.0
urn:ietf:params:netconf:capability:validate:1.0
urn:ietf:params:netconf:capability:url:1.0?scheme=file
urn:ietf:params:netconf:base:1.0
urn:ietf:params:netconf:capability:candidate:1.0
urn:ietf:params:netconf:capability:confirmed-commit:1.0
urn:ietf:params:xml:ns:netconf:base:1.0

```

3. Devised a filter to collect the vlans configured on the device as
   follows:
   
``` python
 vlans_filter = '''
                   <show>
                     <vlan>
                         <brief\>
                     </vlan>
                   </show>
               '''
```

4. And collected the output as follows:

``` python
In [85]: ncm.get(('subtree', vlans_filter))
Out[85]: 
<?xml version="1.0" encoding="ISO-8859-1"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:if="http://www.cisco.com/nxos:1.0:if_manager" xmlns:nfcli="http://www.cisco.com/nxos:1.0:nfcli" xmlns:nxos="http://www.cisco.com/nxos:1.0" xmlns:vlan_mgr_cli="http://www.cisco.com/nxos:1.0:vlan_mgr_cli" message-id="urn:uuid:b1fec116-f94b-4a1f-8c42-964da7c54f26">
 <data>
  <vlan_mgr_cli:show>
   <vlan_mgr_cli:vlan>
    <vlan_mgr_cli:brief>
     <vlan_mgr_cli:__XML__OPT_Cmd_show_vlan_brief___readonly__>
      <vlan_mgr_cli:__readonly__>
       <vlan_mgr_cli:TABLE_vlanbriefxbrief>
        <vlan_mgr_cli:ROW_vlanbriefxbrief>
         <vlan_mgr_cli:vlanshowbr-vlanid>1</vlan_mgr_cli:vlanshowbr-vlanid>
         <vlan_mgr_cli:vlanshowbr-vlanid-utf>1</vlan_mgr_cli:vlanshowbr-vlanid-utf>
         <vlan_mgr_cli:vlanshowbr-vlanname>default</vlan_mgr_cli:vlanshowbr-vlanname>
         <vlan_mgr_cli:vlanshowbr-vlanstate>active</vlan_mgr_cli:vlanshowbr-vlanstate>
         <vlan_mgr_cli:vlanshowbr-shutstate>noshutdown</vlan_mgr_cli:vlanshowbr-shutstate>
         <vlan_mgr_cli:vlanshowplist-ifidx>Ethernet1/1,Ethernet1/2,Ethernet1/3,Ethernet1/4,Ethernet1/5,Ethernet1/6,Ethernet1/7,Ethernet1/8,Ethernet1/9,Ethernet1/10,Ethernet1/11,Ethernet1/12,Ethernet1/13,Ethernet1/14,Ethernet1/15,Ethernet1/16,Ethernet1/17,Ethernet1/18,Ethernet1/19,Ethernet1/20,Ethernet1/21,Ethernet1/22,Ethernet1/23,Ethernet1/24,Ethernet1/25,Ethernet1/26,Ethernet1/27,Ethernet1/28,Ethernet1/29,Ethernet1/30,Ethernet1/31,Ethernet1/32,Ethernet1/33,Ethernet1/34,Ethernet1/35,Ethernet1/36,Ethernet1/37,Ethernet1/38,Ethernet1/39,Ethernet1/40,Ethernet1/41,Ethernet1/42,Ethernet1/43,Ethernet1/44,Ethernet1/45,Ethernet1/46,Ethernet1/47,Ethernet1/48,Ethernet1/49,Ethernet1/50,Ethernet1/51,Ethernet1/52,Ethernet1/53,Ethernet1/54,Ethernet1/55,Ethernet1/56,Ethernet1/57,Ethernet1/58,Ethernet1/59,Ethernet1/60,Ethernet1/61,Ethernet1/62,Ethernet1/63,Ethernet1/64</vlan_mgr_cli:vlanshowplist-ifidx>
        </vlan_mgr_cli:ROW_vlanbriefxbrief>
        <vlan_mgr_cli:ROW_vlanbriefxbrief>
         <vlan_mgr_cli:vlanshowbr-vlanid>100</vlan_mgr_cli:vlanshowbr-vlanid>
         <vlan_mgr_cli:vlanshowbr-vlanid-utf>100</vlan_mgr_cli:vlanshowbr-vlanid-utf>
         <vlan_mgr_cli:vlanshowbr-vlanname>TEST100</vlan_mgr_cli:vlanshowbr-vlanname>
         <vlan_mgr_cli:vlanshowbr-vlanstate>active</vlan_mgr_cli:vlanshowbr-vlanstate>
         <vlan_mgr_cli:vlanshowbr-shutstate>noshutdown</vlan_mgr_cli:vlanshowbr-shutstate>
        </vlan_mgr_cli:ROW_vlanbriefxbrief>
        <vlan_mgr_cli:ROW_vlanbriefxbrief>
         <vlan_mgr_cli:vlanshowbr-vlanid>101</vlan_mgr_cli:vlanshowbr-vlanid>
         <vlan_mgr_cli:vlanshowbr-vlanid-utf>101</vlan_mgr_cli:vlanshowbr-vlanid-utf>
         <vlan_mgr_cli:vlanshowbr-vlanname>TEST101</vlan_mgr_cli:vlanshowbr-vlanname>
         <vlan_mgr_cli:vlanshowbr-vlanstate>active</vlan_mgr_cli:vlanshowbr-vlanstate>
         <vlan_mgr_cli:vlanshowbr-shutstate>noshutdown</vlan_mgr_cli:vlanshowbr-shutstate>
        </vlan_mgr_cli:ROW_vlanbriefxbrief>
        <vlan_mgr_cli:ROW_vlanbriefxbrief>
         <vlan_mgr_cli:vlanshowbr-vlanid>102</vlan_mgr_cli:vlanshowbr-vlanid>
         <vlan_mgr_cli:vlanshowbr-vlanid-utf>102</vlan_mgr_cli:vlanshowbr-vlanid-utf>
         <vlan_mgr_cli:vlanshowbr-vlanname>TEST102</vlan_mgr_cli:vlanshowbr-vlanname>
         <vlan_mgr_cli:vlanshowbr-vlanstate>active</vlan_mgr_cli:vlanshowbr-vlanstate>
         <vlan_mgr_cli:vlanshowbr-shutstate>noshutdown</vlan_mgr_cli:vlanshowbr-shutstate>
        </vlan_mgr_cli:ROW_vlanbriefxbrief>
       </vlan_mgr_cli:TABLE_vlanbriefxbrief>
      </vlan_mgr_cli:__readonly__>
     </vlan_mgr_cli:__XML__OPT_Cmd_show_vlan_brief___readonly__>
    </vlan_mgr_cli:brief>
   </vlan_mgr_cli:vlan>
  </vlan_mgr_cli:show>
 </data>
</rpc-reply>


```

As you can see from the output, I am getting the vlan information back
correctly (you can see details for vlan 1, 100, 101 & 102). However I
was expecting this to conform to the yang model from the [published Model](https://github.com/YangModels/yang/blob/master/vendor/cisco/nx/7.0-3-I6-1/Cisco-NX-OS-device.yang)
and the output was nowhere close.

After much searching on the web I ran into two of my colleagues'
blogs :
1. [Jason's](https://twitter.com/jedelman8) [blog from over 2 years ago! ](http://jedelman.com/home/netconf-and-the-ncclient/) and
2. A [more recent post from](https://projectme10.wordpress.com/2017/03/06/cisco-wants-you-to-use-apis-and-it-shows/) [Gabriele](https://twitter.com/GabrieleGerbino) ) 

(This is truly one of the best parts of my job really; working with folks
much smarter than yourself)

After a quick chat with Jason I realized the source of my confusion. I
was simply interacting with a NETCONF enabled XML API (exposing a XML
schema) on NXOS and not
the YANG model. I was getting somewhere. Revisiting the Cisco
documentation, I noticed that I had overlooked the connectivity port
parameter needed to interact with the nexus switch, for YANG!
So to appears that once the NETCONF agent is enabled on the switch,
Cisco has provided us with an option of programmatically interacting
with the device over NETCONF either over port 22 or port 830. If you
connect over port 22, you can use the XML Schema Database (XSD) over NETCONF
whereas over port 830, you can use the YANG Model.

So here is the correct way to interact with the NXOS using NETCONF and
YANG:

1. Establish the connectivity over port 830

``` python
params = {'allow_agent': False,
 'device_params': {'name': 'nexus'},
 'host': '172.16.30.100',
 'hostkey_verify': False,
 'look_for_keys': False,
 'password': 'admin',
 'port': '830',
 'username': 'admin'}

m = manager.connect(**params)
```

2. Look at the offered capabilities (contrast it with the previous
   section)
   
``` python
In [88]: for capability in m.server_capabilities:
   ....:     print capability
   ....:     
urn:ietf:params:netconf:capability:writable-running:1.0
http://openconfig.net/yang/vlan?revision=2016-05-26&module=openconfig-vlan&deviations=openconfig-vlan-deviations
urn:ietf:params:netconf:capability:rollback-on-error:1.0
urn:ietf:params:netconf:capability:confirmed-commit:1.1
urn:ietf:params:netconf:capability:validate:1.1
http://openconfig.net/yang/local-routing?revision=2016-05-11&module=openconfig-local-routing&deviations=openconfig-local-routing-deviations
http://openconfig.net/yang/bgp-multiprotocol?revision=2016-06-06&module=openconfig-bgp-multiprotocol&deviations=openconfig-bgp-multiprotocol-deviations
urn:ietf:params:netconf:base:1.0
urn:ietf:params:netconf:base:1.1
urn:ietf:params:netconf:capability:candidate:1.0
http://openconfig.net/yang/routing-policy?revision=2016-05-12&module=openconfig-routing-policy&deviations=openconfig-routing-policy-deviations
http://openconfig.net/yang/bgp?revision=2016-06-06&module=openconfig-bgp&deviations=openconfig-bgp-deviations
http://openconfig.net/yang/interfaces?revision=2016-05-26&module=openconfig-interfaces&deviations=openconfig-interfaces-deviations
http://openconfig.net/yang/interfaces/ip?revision=2016-05-26&module=openconfig-if-ip&deviations=openconfig-if-ip-deviations
http://cisco.com/ns/yang/cisco-nx-os-device?revision=2017-05-16&module=cisco-nx-os-device&deviations=cisco-nx-os-device-deviations

```
3. Use the appropriate namespace to collect the vlan information:

``` python
vf = """<filter type="subtree">
                <System xmlns="http://cisco.com/ns/yang/cisco-nx-os-device">
                        <stp-items>
                            <inst-items>
                              <vlan-items>
                                  <Vlan-list/>
                              </vlan-items>
                            </inst-items>
                        </stp-items>
                </System> 
            </filter>
 """
m.get(vf)

<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns:if="http://www.cisco.com/nxos:1.0:if_manager" xmlns:nfcli="http://www.cisco.com/nxos:1.0:nfcli" xmlns:nxos="http://www.cisco.com/nxos:1.0" xmlns:vlan_mgr_cli="http://www.cisco.com/nxos:1.0:vlan_mgr_cli" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:2bdcccfb-e1b7-4d5c-a56d-081410edf08a">
    <data>
        <System xmlns="http://cisco.com/ns/yang/cisco-nx-os-device">
            <stp-items>
                <inst-items>
                    <vlan-items>
                        <Vlan-list>
                            <id>101</id>
                            <adminSt>enabled</adminSt>
                            <bridgeAddress>00:00:00:00:00:00</bridgeAddress>
                            <diameter>2</diameter>
                            <fwdTime>15</fwdTime>
                            <helloTime>2</helloTime>
                            <maxAge>20</maxAge>
                            <operErr/>
                            <priority>32768</priority>
                            <protocol>unknown</protocol>
                            <root>primary</root>
                            <rootAddress>00:00:00:00:00:00</rootAddress>
                            <rootPathCost>0</rootPathCost>
                            <rootPort>unspecified</rootPort>
                            <rootPortNumber>0</rootPortNumber>
                        </Vlan-list>
                        <Vlan-list>
                            <id>1</id>
                            <adminSt>enabled</adminSt>
                            <bridgeAddress>08:00:27:88:8B:E1</bridgeAddress>
                            <bridgePriority>32769</bridgePriority>
                            <diameter>2</diameter>
                            <fwdTime>15</fwdTime>
                            <helloTime>2</helloTime>
                            <if-items>
                                <VlanIf-list>
                                    <id>eth1/1</id>
                                    <designatedBridgeAddress>08:00:27:88:8B:E1</designatedBridgeAddress>
                                    <designatedBridgePriority>32769</designatedBridgePriority>
                                    <designatedPortNumber>1</designatedPortNumber>
                                    <designatedPortPriority>128</designatedPortPriority>
                                    <designatedRootAddress>08:00:27:88:8B:E1</designatedRootAddress>
                                    <designatedRootCost>0</designatedRootCost>
                                    <designatedRootPriority>32769</designatedRootPriority>
                                    <portNumber>1</portNumber>
                                    <portPathCost>4</portPathCost>
                                    <portPriority>128</portPriority>
                                    <portRole>designated</portRole>
                                    <portState>disabled</portState>
                                </VlanIf-list>
                            </if-items>
                            <maxAge>20</maxAge>
                            <operErr/>
                            <priority>32768</priority>
                            <protocol>rstp</protocol>
                            <root>primary</root>
                            <rootAddress>08:00:27:88:8B:E1</rootAddress>
                            <rootPathCost>0</rootPathCost>
                            <rootPort>unspecified</rootPort>
                            <rootPortNumber>0</rootPortNumber>
                            <rootPortPriority>0</rootPortPriority>
                            <rootPriority>32769</rootPriority>
                        </Vlan-list>
                        <Vlan-list>
                            <id>102</id>
                            <adminSt>enabled</adminSt>
                            <bridgeAddress>00:00:00:00:00:00</bridgeAddress>
                            <diameter>2</diameter>
                            <fwdTime>15</fwdTime>
                            <helloTime>2</helloTime>
                            <maxAge>20</maxAge>
                            <operErr/>
                            <priority>32768</priority>
                            <protocol>unknown</protocol>
                            <root>primary</root>
                            <rootAddress>00:00:00:00:00:00</rootAddress>
                            <rootPathCost>0</rootPathCost>
                            <rootPort>unspecified</rootPort>
                            <rootPortNumber>0</rootPortNumber>
                        </Vlan-list>
                        <Vlan-list>
                            <id>100</id>
                            <adminSt>enabled</adminSt>
                            <bridgeAddress>00:00:00:00:00:00</bridgeAddress>
                            <diameter>2</diameter>
                            <fwdTime>15</fwdTime>
                            <helloTime>2</helloTime>
                            <maxAge>20</maxAge>
                            <operErr/>
                            <priority>32768</priority>
                            <protocol>unknown</protocol>
                            <root>primary</root>
                            <rootAddress>00:00:00:00:00:00</rootAddress>
                            <rootPathCost>0</rootPathCost>
                            <rootPort>unspecified</rootPort>
                            <rootPortNumber>0</rootPortNumber>
                        </Vlan-list>
                    </vlan-items>
                </inst-items>
            </stp-items>
        </System>
    </data>
</rpc-reply>

```

As you can see from this output, the data conforms to the yang model
as follows (only a section shown)

``` shell
       +--rw stp-items
       |  +--rw name?         naming_Name
       |  +--rw adminSt?      nw_AdminSt
       |  +--ro operSt?       nw_EntOperSt
       |  +--ro operErr?      nw_OperErrQual
       |  +--rw inst-items
       |     +--rw vlan-items
       |        +--rw Vlan-list* [id]
       |           +--rw id                  stp_VlanId
       |           +--rw priority?           stp_Priority
       |           +--rw root?               stp_Root
       |           +--rw diameter?           stp_Diameter
       |           +--ro operErr?            nw_OperErrQual
       |           +--ro protocol?           stp_Protocol
       |           +--rw bridgePriority?     uint16
       |           +--ro bridgeAddress?      address_Mac
       |           +--rw rootPriority?       uint16
       |           +--ro rootAddress?        address_Mac
       |           +--rw rootPortPriority?   uint16
       |           +--ro rootPortNumber?     uint16
       |           +--ro rootPort?           nw_IfId
       |           +--ro rootPathCost?       uint32
       |           +--rw adminSt?            nw_AdminSt
       |           +--rw fwdTime?            stp_FwdTime
       |           +--rw helloTime?          stp_HelloTime
       |           +--rw maxAge?             stp_MaxAge
       |           +--rw if-items
       |              +--ro VlanIf-list* [id]
       |                 +--ro id                          nw_IfId
       |                 +--ro designatedRootPriority?     uint16

```
*Please Note: I have edited the output of pyang -f tree to highlight the significant portions for this post*
  
In summary, be vigilant about the API schema you are working with. It
is very easy (especially for most of us who are just getting started),
to not realize whether we are dealing with a YANG model or a XSD
schema. Hopefully this post highlights the difference and helps
further our collective understanding. Appreciate your feedback! 
