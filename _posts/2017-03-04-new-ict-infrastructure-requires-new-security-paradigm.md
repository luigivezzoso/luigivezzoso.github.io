---
title: "New ICT infrastructure requires new security paradigm"
date: 2017-03-04
author: Luigi Vezzoso
layout: post
tags: 
    - general
category: posts
---

The title of this post may appear trivial but..... I know many security "expert", coming from the classic security architecture based on physical security, physical perimeter protection (firewalls, IPS/IDS, etc.) which continue to belive that only well defined boundary between security zones (LAN, DMZ, WAN, etc.) can protect your environment. The purpose of this post is to introduce some ideas about SDN and in particular the VMware NSX capability. Is not very technical and I'm not going deeper in the description because is out of scope. For all the technical details, please refer to the vendors documentation or ask me a question!

![virtualization-history](assets/postimages/virtualization-history.jpg)

The virtualizzation history stars many years ago.... I wasn't yet born. The great moment of the virtualizzation is when it could be used for the  standard server hardware virtualizzation. With that way you can run many VMs (wit standard OS) in a single piece of hardware maximizing the usage of the bare metal and optimizing costs.
Nowadays with the complete virtualizzation of datacenters and their cloud-izzation (the movement in the cloud) you need a new approach to securing that environment because the boundary are not well-defined.... and the cloud might be you LAN extension! Thinking at your datacenter only.... the introduction of virtualizzation makes it difficult to put a physical firewall between VMs. Those physical wire are not scalable and every change in the infrastructure require more hardware, more cabling, more effort..... and actually the security level of that solution (usually....) in not increased by those investments.

 I belong to those people which always prefer hardware/physical security solution instead of logical or virtual technology for many reasons:

- hardware appliances are more powerful
- physical boundary are more secure
- ...
- lots of other stuff

Recently I changed my point of view. With the introduction of SDN and new security solution based on that paradigm (Checkpoint vSec for Vmware NSX, Trendmicro DeepSecurity, etc.) you can protect your virtual and/or cloud datacenter in a more effective and efficient way.
In the "old/classic" physical security paradigm, or more in general in the centralized security architecture, you rely on a central gateway or routing engine on which all the firewalling and security "filtering" is applied.
My experience is based on the VMware NSX architecture but I think the idea behind that is general and is can be applied on all technology.


The main points of VMware NSX are:

- introduce distributed network capability (distributed logical switch, distributed logical routing, distributed firewalling)
- decoupling the physical network from the logical network (introducing an higher level of segmentation with the VX-LAN (https://en.wikipedia.org/wiki/Virtual_Extensible_LAN)
- create a framework to extend the security of NSX with the integration of leader security vendor products (checkpoint, paloalto, trendmicro, etc.)

Thank to these main points you can achieve different benefits. With the distributed network capability you can have a better network usage in the virtual datacenter, restricting the traffic in the hosts and network that should communicate. There is no need to transmit traffic outside the hosts if not required.
The VX-LAN is a powerful method decoupling the physical topology from the logical topology.... in this way you can for example, extend the same network (like a L2 stretched LAN)  across different datacenter with complete different networks defined! In addition the VX-LAN is a great tool for the scalability and for the multi-tenant datacenter.
Finally, and for me is a key-point, the integration of VMware NSX with strong security vendor add a greater level of security to the NSX architecture relying to security partner the application security capability.... VMware is a great virtualizzation company but have not the focus on security....
Just an example taken from the NSX documentation: the picture below shows some examples of benefits (in terms of routing and firewalling) using the NSX architecture.
Look at the two examples below: without network virtualizzation (VMware NSX in this case) the traffic between two VMs is always firewalled by the centralized firewall and should traverse the complete stack of access/distribution/core network to finally return to the VMs.... in past we always did in this way...

![image1](assets/postimages/663077621.PNG) 
![image2](assets/postimages/354208960.PNG)

With the introduction of distributed networking we can have routing and firewalling capability at the hypervisor level the routing and firewall decision could be taken near the VMs with better routing and network usage and more security.
But lets talk about security: with the possibility to restrict the VMs inside the host (micro segmentation) you can isolate automatically the VM in case of incident (malware, ransomware. virus, compromising, etc)
With the standard network and security architecture just few company have in place very restricted policy to separate VM.... they usually haven't a firewall inside the virtual datacenter. In addition the distributed nature of the NSX Logical Firewall is scalable and every hypervisor you add more power you have to process data.

Last but not least: you can apply firewall rules also between two VMs belong to the same L2 segment like a transparent/bridge firewall. This feature with the strong capability of IPS, Antivirus/Antimalware, etc. provided by security vendor can help you to have a deeper security.

## Some conclusion.

The rush to the cloud services introduce new service paradigm and new security services (cloud based 2FA, cloud antispam services, cloud web-security-gateway, etc.). To follow this rush We need to change our security architecture vision also. If we continue to think in the classical LAN/DMZ/WAN logic....we cannot follow the business requirements and we are out of market. But is not only a thing related to business....

The cloud is changing our life: Google drive, Outlook 365, and many other cloud services are used by us every day. The security community should alway find new way to contrast cyber crime and help the humanity to live better in safe.


