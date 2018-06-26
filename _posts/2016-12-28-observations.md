---
layout: post
title: The need for a stateful variable tracker and an implementation example
comments: true
---
When it comes to automation role models, network engineers have often looked up,
to our compute brethren. For decades, compute admins have had tools that allowed
them to execute scripts on systems at particular times\: typically backups,
rsync etc. More recently, in the VM universe,  DevOps tools like Chef/Puppet/
Ansible, have empowered 'Developer Administrators', to stand up the entire app
stack, automatically.

In my network automation journey, I realized early on that, a big gap/obstacle
for network automation is the need for a backend, to track positive
integers; a source of truth, that knows what numeric value was last

<!--more-->

assigned to a particular network *service*(viz. firewall contexts,
vlans etc  for a 3 tier app in the DMZ. Traditional networks are
relatively static(from a configuration
standpoint). We typically make incremental changes to *variables* in a production
network configuration. For instance, once a port-channel
is created, say Po101, we typically have some internal standard as to how the
next port-channel will be numbered (could be Po102, Po201 etc). For a given
'network service', we have to track many similar variables: Vlan numbers, Vrf numbers,
HSRP group numbers, subinterface numbers, AS numbers.... The list goes on.
Now, in quick contrast, the compute folks have almost never had this problem.
Most automation in that space is\:

1. Stand up the VM
2. Install necessary middleware
3. Clone the app repo
4. Fire up the app
5. Ensure compliance

## What about IP addresses:
Obviously, I am oversimplifying a bit, but the point remains that, there really
isn't much state tracking, when it comes to application/OS admin automation.
Back in the day (actually, less than 10 yrs ago), I remember when
compute/application admins were very fond of static IP addresses.
That used to be 'stateful' variable they needed tracked. Not any more.
Unfortunately, on the network side, we are still very dependent (depending on the use
case) on static IP addressing for our devices. Needless to say a solid IPAM
is an extremely important stateful variable tracker for network
automation. The reason for this blog post is, however, to address the
"other" variables (vlan numbers, vif numbers et al), we need for
network automation, that doesn't really come built into standard,
'network focused' software, like IPAMs/CMDB

## An implementation example using NSoT:
[NSoT](https://github.com/dropbox/nsot) is an opensource
IPAM(primarily) from the folks at dropbox. Last year at the NetDevOps
workshop at Interop,  [Jathan](https://twitter.com/jathanism) demo'ed
the product. It had 2 things that caught my attention\:

1. It was written with an API first approach
2. It was written in python (a language that I am least uncomfortable
   in :) )

I forked the repo and implemented the "Iterables" API, with a lot of
guidance and support from Jathan. We have since, internally, used my
implementation of NSoT, to prove out quite a few automation scenarios.

## Iterables - A visualization:
My implementation of the stateful variables, involves 2 tables. One
table tracks the variable that needs to be iterated (vlans numbers,
vif numbers etc, along with the increment value). The other table
tracks all allocated values for a given variable.
My good friend [Bobby Outlaw](https://www.linkedin.com/in/bobbyoutlaw)
helped visualize the idea as follows:


[![](/assets/iterables.png)](/assets/iterables.png)


Using NSoT as a backend, the above drawing visualizes various network
automation services, within a service provider. *Service A* requires the
network automation system to keep track of a cryptomap sequence
number, tenant VRF number and a portchannel interface
number. Alternatively *Service B* requires the automation system to
track the cryptomap sequence number and the tenant vrf number.
Each time we invoke the "playbook" for service A, we can now make API
calls to our NSoT backend, that will give us the next available unused
integer for the given variable.
This particular implementation, also gives you an unique *"Service
Key"* each time you invoke the playbook. The idea behind it is; as a
service provider, you hand over the unique key to your customer as a
reference to the service request. If the customer wants the service
revoked or modified (pretty much any CRUD operation)  at a later date,
the service key can be used to identify all the values associated with
a particular service, and thus modified.

## Demo Playbook:
If you'd like to play around with the implementation, you can download
a copy [here](https://github.com/termlen0/nsot). Please keep in mind
that you will have to follow
the
[developer guide](https://nsot.readthedocs.io/en/latest/development.html)
to compile from source. (Hopefully once the PR is approved, it should
be available for general consumption, directly from pip). Once you have NSoT running, you
should be able to grab
the [iterable test playbook](https://github.com/termlen0/nsot-tester)
to get an idea of how to use the stateful backend for your automation
playbooks.
