---
layout: post
title: vLab 2 - DNS
categories: [showcase]
tags: [virtualisation, linux, vlab, dhcp, dns, kvm]
description: Setting up a dns & dhcp server in CentOS 7
date: 2020-06-04
lang: en
---
To be honest I thought that this task was a rather simple one. I was wrong.
The initial task was to set up a single server that offers dns and dhcp. As I
guessed right, this was (because of the size of my virtual lab) the easy part.
The next step was to add the pxeboot feature to labsmt1 (Spacewalk) and to
advertise it via dhcp.

## Overview
To implement the requested features in task #2 I've chosen the bind 9 server as
the dns and the isc-dhcp as the dhcp server. I'm convinced that both of these
software packages are the most used ones on linux server systems.

While implementing I have encountered 4 issues. Some minor and others even major
. This step has already taught me more than I'd have expected to learn in 5 or 6
steps. This really motivates me to move on, so I figured to just let you know.

## Planning
Beginning with the specs, I've decided to bring them down to 1 GB of RAM, 20 GB
disk space and 1 core. I don't expect any problems in the long run, as this
server is not often used. Also the latency is not important.

Because I've already set up isc-dhcp and bind9 in the past, I kinda know what to
expect. The steps to install both services are:
- Install CentOS 7
- Install bind & dhcp
- Configure lookup zones (forward & reverse)
- Enter all DNS names
- Configure the DHCP range

Because the DHCP service is mostly used to initially assign servers only
temporary network information (before it gets static), I'll dispense configuring
dynamic dns entries.

I think: where a DHCP server is, there is also a need for an ip-address plan.
For the distribution of my servers, on my subnet, I have deemed these factors as
most important:
- meta information
- flexibility
- accessibility

What does this mean in specific. I need meta information for IP addresses,
because it's a neat feature to have some information at hand when you are just
pinging or searching servers. Though there is not much place for meta
information inside an address. I dont want to bloat this up. I tought it'd be
good to combine the importance and workspace of a server with it's address.
To get started I defined 4 groups:
1. Infrastructure (network, information and basic function)
2. Back-end (services accessible by front-end systems)
3. Front-end (services accessible by users)
4. Auxillary (assistive systems)

As I stick with a fixed size of servers for my lab project, flexibility is not
a big issue, but I still want to allow 20% of address space buffer for minor
changes. The accessibitlity as the last entry on my list is a nice to have
feature of my address plan. The address should be easy to remember and in a few
special cases it should also be easy to guess without much knowledge about the
network. That means a fixed scheme and mostly contiguous or otherwise logically
comprehensible addresses. These are the IP addresses with the respective
priority group assigned:

| Hostname | IP address | Group |
| -------- | ---------- | ----- |
| labdns1  | 192.168.122.10 | 1 |
| labldp1  | 192.168.122.11 | 1 |
| labldp2  | 192.168.122.12 | 1 |
| labstg1  | 192.168.122.13 | 1 |
| labldb1  | 192.168.122.14 | 1 |
| labsmt1  | 192.168.122.20 | 2 |
| labcfg1  | 192.168.122.21 | 2 |
| labsql1  | 192.168.122.22 | 2 |
| labsql2  | 192.168.122.23 | 2 |
| labweb1  | 192.168.122.30 | 3 |
| labweb2  | 192.168.122.31 | 3 |
| labwac1  | 192.168.122.32 | 3 |
| labwac2  | 192.168.122.33 | 3 |
| labbkp1  | 192.168.122.40 | 4 |
| labeml1  | 192.168.122.41 | 4 |
| labmon1  | 192.168.122.42 | 4 |
| lablog1  | 192.168.122.43 | 4 |

Changes to this list when proceding with the project are possible. Just typing
this list made me rethink my grouping decisions. I decided against putting the
system and configuration management servers inside group 4, because I see the
administrator also as a "backend system" accessing servers. It's important to
grant the system management server a high priority, because otherwise it would
slowly fade into insignificance and won't be used anymore. As I see the system
management as a first responder documentation (out of my experience). A loss in
integrity of managed systems would be crucial to the stability of the whole
network. The next question arrised after realising that the Spacewalk server
also serves as a configuration server. This could mean that labcfg1 is no longer
needed. I'll postpone that decision.

The DNS plan for the hosts is integrated in the upper table. As my zone I've
chosen to create a subdomain of my actual domain: lab.fabianbissmann.de. Despite
the duplicate information (lab) and the longer text in a FQDN, I think it's the
solution with the most advantages.
- I am the owner of my internal domain (I wouldn't if I chose any fantasy
domain)
- I can use officially signed certificates on services

## Implementation
The system is going to be CentOS 7 as before. I'll not go into detail as in the
last post, because I think the installation process should be clear. Despite
that, here is where I encountered my first issue (more on that in the section
Problems).

Despite that I already have installed DNS and DHCP servers I conducted several
manuals in hope of learning something new and to decrease the possibility to
create issues I'll notice later on.
These are the links to the manuals:
- [a](www.google.de)
- [b](www.google.de)

Installation of DHCP Server
-> Disabling of QEMU-supplied DHCP-Server

Show off some results

Now time to enable the Spacewalk server as a target for pxe boot.
errors and errors later...

And then short showing off of the finished spacewalk server

## Problems
### More RAM when installing
### No Gateway configured
### Spacewalk Distribution failed
### Authorization not available boot error

## Lessons learned
phew
