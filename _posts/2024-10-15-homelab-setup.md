---
title: Homelab Setup
date: 2024-10-15 20:00:00 -0500
categories: [Education]
tags: [blog, homelab, pass the hash]
description: Starting a Homelab for Learning and Experimentation
toc: false
---
# Motivation

Last Thursday, Oct 10, I spoke with my school's cybersecurity club about my desire to get a job in the industry. I received the usual advice, with an extra emphasis on continuing to learn. This stood out to me, though. Someone said, "Not many newly grads can say they know what a pass the hash attack looks like on the logs, or how to tell if an actor is using mimikatz or evil winrm." A second person chimed in with, "Even if they do know, they often don't know how to stop it."

I'm going to learn how to spot a pass-the-hash attack in logs.

# Execution and Planning

So far, I have successfully:

* Installed a Windows Server VM and a Windows Pro VM to my hypervisor
* Set the Windows Server as a domain controller
* Joined the Windows Pro to the domain
* Added DHCP, DNS, and RRAS services to the Server

The client VM originally had trouble with connecting outside the local network, but I found that the problem was in the configuration of the network adapters on the Server VM. I had set up two bridged adapters, thinking I might need the second adapter for connecting to other devices on my local network, but this led to ambiguity in routing. When I realized the cause of the routing trouble, it immediately became apparent how I had overthought the routing needs of the Server VM.

Still to do:

* Install a SIEM for the sake of data gathering and future projects
* Actually do the PtH attack from a third machine
    * This will need to be on the same "network" in the hypervisor, and I know it can either be a Debian-based system or a Windows system.
    * For the first test, I'll assume I have stolen credentials for one user and will use Mimikatz to get another hash and use that for lateral movement
    * That will be a "successful" attack
* Observe the logs in Windows Event Manager and Kibana and note how it might differ from a legitimate login
