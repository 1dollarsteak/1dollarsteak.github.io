---
layout: post
title: Creating a virtual lab
categories: [showcase]
tags: [virtualisation, linux]
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

| Step | Hostname | RAM (GB) | Disk (GB) | CPU (Cores) |
| ---- | -------- | -------- | --------- | ----------- |
| 1    | labsm1   | 2        | 50        | 1           |

