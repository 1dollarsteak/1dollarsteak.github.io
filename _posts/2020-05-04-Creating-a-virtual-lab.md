---
layout: post
title: Creating a virtual lab
categories: [showcase]
tags: [virtualisation, linux, vlab]
description: An initial showcase of setting up a small virtual lab
lang: en
---

As I want to advance my skills in managing Linux systems. I want to dive into
some software that's used quite regularly (I guess) in server environments.
On the internet, there exists a pretty good to do list for the curious. All
credit of this list goes to the User [IConrad](https://reddit.com/u/IConrad) on
Reddit. I've only made some minor modifications to it, so that it'll better fit
my needs and interests.

## Overview
Generally speaking, the whole project consists of 20 steps and contains the
following topics/software:
- KVM
- System Management (Spacewalk & Puppet)
- Infrastructure (DNS, DHCP & LDAP)
- Backup
- E-Mail
- Monitoring
- Documentation

Due to a current lack of server hardware I'll mainly create virtual machines,
with KVM, to host the software. I have already begun testing on CentOS 8 inside
a vm, but it seems that the software Spacewalk is only supported on CentOS 7
yet. Therefore I'm pretty sure, that every vm will revolve around CentOS 7.

This post exists to create a bare overview of the whole side project. In the
next upcoming weeks I'll create a new post for each new step, documenting my
process and lessons learned.

## Hypervisor specifications
The computing unit I'm working on has the following specs:
- RAM: 16 GB (2 x 8 GB DDR4-2400)
- Disk Size: 500 GB
- CPU: AMD Ryzen 5 1400 Quad-Core 3,2 GHz
- Mainboard: Asus Prime B350M-A

The following virtualisation solution is used:
- Infrastructure: KVM
- Software: QEMU

I'm aware that the available memory is low. I've decided to see how far I can go
with this capacity. After operation isn't possible anymore I'll perform an
upgrade and double the size (see below).

## Vm plan
In the following table, I've written down a roughly calculated usage of my
available hardware. If everything is operable at the given RAM size then I'll
end up with 8 GB of memory available for the host or as a buffer.
The disk sizes should be sufficient, as I don't see any big data in the used
software.

| Step | Hostname | RAM (GB) | Disk (GB) | CPU (Cores) | Use |
| ---- | -------- | -------- | --------- | ----------- | ----------------- |
| 1    | labsmt1  | 2        | 50        | 1           | Spacewalk         |
| 2    | labdns1  | 1        | 20        | 1           | named & dhcpd     |
| 3    | labldp1  | 1        | 20        | 1           | LDAP 1            |
| 4    | labldp2  | 1        | 20        | 1           | LDAP 2            |
| 5    | labsql1  | 2        | 20        | 1           | PostgreSQL 1      |
| 6    | labsql2  | 2        | 20        | 1           | PostgreSQL 2      |
| 7    | labcfg1  | 1        | 20        | 1           | Puppet            |
| 8    | labstg1  | 1        | 20        | 1           | iscsitgt & nfs    |
| 9    | labbkp1  | 1        | 20        | 1           | bakula Backup     |
| 10   | labweb1  | 1        | 20        | 1           | Apache httpd 1    |
| 11   | labweb2  | 1        | 20        | 1           | Apache httpd 2    |
| 12   | labwac1  | 1        | 20        | 1           | Apache Tomcat 1   |
| 13   | labwac2  | 1        | 20        | 1           | Apache Tomcat 2   |
| 14   | labldb1  | 1        | 20        | 1           | iptables          |
| 15   | labeml1  | 2        | 20        | 1           | postfix           |
| 16   | labmon1  | 1        | 20        | 1           | Nagios core       |
| 17   | lablog1  | 4        | 20        | 1           | ELK stack         |
| **&#8721;** | **-** | **24** | **430** | **17**      | **-**             |

In sum there are 17 active vms in my virtual lab when the end of this project is
reached. Step 5 and 6 is primarily used for inhabiting the database of vm #1.
Vm 10-13 is for hosting a wiki server and maybe more. Preferably the wiki 
[Confluence](https://www.atlassian.com/software/confluence) from Attlassian will
be used. Caching and loadbalancing is done with
[Infinispan](https://infinispan.org/). The postfix server is set up to receive
mails from every server inside the lab network. This is set up due to monitoring
reasons. Also the monitoring section is extended with #16 and #17. Where
[Nagios](https://www.nagios.com/products/nagios-core/) takes in place as the
monitoring server and the
[Elasticsearch stack](https://www.elastic.co/de/elastic-stack)
is responsible as a log server and data visualizer in the first place.

## Up next
The preparations for the first vm have already been started. Right now the
hypvervisor is all set up and I'll begin shortly with setting up Spacewalk. The
next post also serves as a template for the structure of the future posts of
this project.
