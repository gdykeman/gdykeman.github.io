---
layout: post
title: Of state, idempotency, and CI/CD in the brownfield network
comments: true
---
This post is a consequence of an interesting conversation around
declarative control systems with a colleague. Throughout the
conversation, we kept coming back to the 'popularity' of automation
and specifically, how revolutionary, it appears for legacy, closed
systems(aka 80% of network gear in enterprises). When you think about it,
in a traditional control system (a bread toaster, in it's simplest
form), we interact with the system as follows:

    1. We tell it (declare) what our desired outcome is (how brown do you
    want your toast)
    2. Inputs (the slices of bread)

The system then figures out how to achieve your end state. In other
words..

<!--more-->

```
    * EndState -> System
    * System determines current state and calculates diff to achieve end
    state
    * System then does: Current State +/- diffs, to achieve end state
```

## Imperative and declarative systems
Lately I have been working a lot with ansible, with a specific focus
on network devices that natively do not have good/any api: Cisco
routers and switches, in particular. If you are familiar, you already
know, that the core ansible modules are idempotent. Ansible does this,
by getting the current state of the device before execution,
validating whether the differences to be applied, are already present,
and if not, applying them.  Going back to the earlier discussion on
control systems, it is pretty obvious that the way we achieve the "end
state" in ansible is quite different. Here, no desired/endstate is given
to the system. The operator calculates the differences that need to be
applied and gives it to the system. The system then assures that, the
requested differences are currently not present in the devices and
applies them.

```
    * Differences -> System.
    * System collects the current state
    * System validates that differences do not exist in the current
    state
    * System adds the differences
```

I specifically say that the system **adds** the differences. What I am
trying to point out, is the non-declarative nature here. For a real
example, if I have to remove a bgp neigbor through ansible, I have to
say

``` no ip bgp neighbor x.x.x.x```

Rather than being able to define the end state, where I tell the
system how I would like the device configuration to look like, I have
to tell it specifically, what needs to happen. This is not a
flaw/drawback. I am just pointing out the nature of closed systems.
Contrast this with a system dealing with configuration files. You can
define your end state, simply by writing the configuration file, the
way you want it to be on the system. The automation tool's job is to
identify the differences and then decide whether configs need to be
applied or removed to the end host - *declarative*

## So what and are there declarative n/w automation options?
Declarative systems are IMO, elegant, and embody
automation/abstraction better than imperative systems. However you
have lesser control over the how (your toaster makes the call about
the temperature to brown your bread).
In the past, I have built a few automated services
using
[tail-f systems](http://www.cisco.com/c/en/us/solutions/service-provider/solutions-cloud-providers/network-services-orchestrator-solutions.html)
This product helps the operator build network services, in a
quasi-declarative fashion. The way it achieves this, is that the tool,
caches/persists the current state of devices locally. When the
operator drives a change, the tool, validates the following:

    1. Does the current state on the devices match the cached configs
    2. If yes; calculate the difference, stage the config locally,
       push to end device
    3. If no; the operator has the option to treat the cached config
       as the source of truth *OR* the current device state as the
       source of truth

This brings me to the concept of state. In a
[previous](https://termlen0.github.io/2016/12/28/observations/)
post, I talked about stateful variable tracking for network
equipment. However, how do we track configuration state?

## CRUD and the database analogy
With tailf, having a cached copy of the "state" of the device, allows
the operator to treat the end devices as rows in a database. So you
have the ability to "commit" your changes and "rollback" when errors
are encountered. However from an automation tool standpoint, I want to
decouple the "state" from the tool.
I believe such a declarative system can be
achieved using a version control system + an imperative tool. For
example:

    1. The state of a topology is saved in say, github.
    2. A change to the system is requested via a pull request. Note
       that this is a declaration of the desired end state, which is
       calculated by some imperative system, like the ansible cisco
       core modules.
    3. The CI/CD tool chain evaluates the requested endstate validity,
       allowing the operator the opportunity to effect corrections.
    4. The desired end state is achieved and subsequently stored at
       the "Source of Truth" - github in this case.

I am not sure how exactly, to handle out of band changes. The
possibility exists, for writing playbooks, that can potentially
overwrite the device configs with the configs stored in github or
update github, with the current device state.

Another point of consideration, comparing network devices to
databases, makes one think of how tightly coupled the "data" and the
"configuration" is on network devices. I had not thought about a
firewall rule in such terms, until this conversation with my
colleague. The configuration of a database is very much decoupled from
the data, whereas the configuration (permit traffic) and data
(source/destination IP/PORTS) are extremely tightly coupled on network
devices - food for thought!

I would love to hear your thoughts on the state of network automation
for closed systems like routers and switches. Also feedback on this
thought exercise would be awesome.
