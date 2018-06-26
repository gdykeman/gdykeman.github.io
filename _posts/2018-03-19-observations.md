---
layout: post
title: Please reverse mentor your managers
comments: true
---

_"Everybody automate everything, now!"_ - Enterprise CTO (who just got back from a industry networking event)

_"I wish my management would help bring in some of this cool network automation stuff I am learning from my peers online"_ - Network engineer


I've heard these comments from numerous organizations; often from CTOs and engineers within the same organization. So if the topmost level of the management is laying down goals to automate "everything" and the engineers themselves are excited to make it happen, why is there a gap?

![](/assets/middle_mgmt.png)


So are middle managers the problem? Across organizations, middle mangers could be senior network engineers who have advanced to a point where there is no technical growth path. They have become comfortable managing a team of network engineers and know how to hire network talent. The automation space is filled with technical jargon and buzzwords that are new (and therefore scary). Coming from a technical background fear of the unknown can be unnerving. 

<!--more-->

From my experience, looking at this objectively, some ofthe common questions/pitfalls a network organization faces with respect to taking on automation are as follows:

1. Aversion to risk

2. Prioritizing automation

3. Fear of ownership

4. Uncharted hiring waters

Note that I did not call out **"Skill Gaps"**. In my opinion, the lack of automation tech-skills is the easiest risk that can be resolved. So much so that I refuse to even list it above.

### 1. Aversion to risk

Network engineers have an OCD streak in them. It was a survival technique. No one wants to be troubleshooting production issues at 2 in the morning, so we are trained to obsessively check and double-check any changes to the network. In the past this has been a necessary trait. With the advancement in virtualization technologies, it is time for us to overcome this behavior. Easier said than done, but I want to lay out some practical steps for network engineers to get there:

- If you are a major vendor shop (say Cisco/Juniper/Arista etc), use them. I always see the twitverse and other social media outlets with every vendor encouraging and trying to help engineers build automation skill sets. Use your SE as your partner to build PoCs and present them to your management. It is hard to fight data.

- Recruit your manager. What is important to her this quarter or year? Take it on as an automation undertaking. If you are waiting for your manager to tell you to automate something, you are doing something wrong.

- Recruit business stakeholders. Start small, educate using PoCs. Demonstrate due-diligence. In spite of all this, if something goes wrong, the organization will learn from the experience. By starting small, organizations will embrace failure.

### 2. Prioritizing

This is a top conversation item when I talk to engineers trying to bring in automation to their organization. _"My manager is fine with me picking up automation as long as I continue to execute my day job. Where do I find the time when my day is already loaded!"_

- Have you tried talking to your manager about how spending 2 days to automate VLAN creation and deletion in your datacenter is going to free everyone's time by 10%? The time gained can then be used to automate even more repeatable tasks building upon itself. 

- Build a case for automation. Have verifiable and achievable goals to automate. 

- Use external peers or vendor resources to justify your case. 

- Empathize with your manager. There might be pressure from her superiors. Partner with her to make her successful

### 3. Fear of ownership

Network IT organizations have forgotten that switches and routers are nothing but computers running specialized applications on them. We have become accustomed to getting a "box" from a vendor, turning it on and tuning it. With downtime translating to revenue loss, network changes were/are always classified "high risk" and subject to excruciating peer-reviews and change control procedures. How many times has improperly configured "redundancy" mechanisms been the root cause for outages? But, there was always a "get out of jail free card" we could use - open a support ticket with the vendor and count on their magic incantations to fix our problems. 


If we use automation one of the biggest fear is not having that "throat to choke". Meaning, if you wrote a piece of automation and executing it caused an outage, then you are to blame. How do we get around this one? Remember the CTO who wanted to automate all the things? This really means building a culture of trust and moving away from the blame game. When automation causes an outage, that should be a learning opportunity for the organization. 


Another cultural aspect is getting over the idea of buying a box and turning it on - expecting it to automate all the things. Automation within an organization can be very subjective. There might be generalizations but the business objectives will usually determine the automation fingerprint.


### 4. Uncharted hiring waters

I alluded to this earlier. Middle managers are comfortable hiring network talent and managing them (they've done it before and know what it takes to administer the network). With pressure from superiors, managers are in a dilemma - Should I hire a developer and teach her networks or should I teach my current network resources programming?!

While I have nothing against hiring of developers to manage a network, I think there is no reason that a current network resource will not be able to pick up programming skills to automate the network. Here are my reasons:

1. Network engineers are declarative programmers (Configuring BGP is declarative programming)

2. Network engineers understand distributed databases (OSPF anyone?)

3. At a high level, network engineers understand concepts of OOPs and functional logic (QoS policies, PBR?)


For managers to be successful, IMO, they have to take a crash course to get familiar with the technology space. I don't see another way around it. It does not have to be deeply technical but something like a 3 day workshop to understand version control systems, DevOps methodologies and tools will be extremely beneficial. This could even be entirely a reverse mentoring process; maybe using weekly meetings. You could even bring your team along by organizing internal "hackathons" to solve particular use cases. Turn this around and use it as a recruiting tool to attract talented engineers.


### Conclusion

In conclusion, I am convinced that the most resistance organizations are facing in the adoption of network automation is in the middle. I think this can be solved through education and partnering. If you've had experiences that were successful or otherwise do share. The community will benefit from our collective understanding of challenges across the board.


