---
layout: post
title: vLab 1 - Spacewalk
categories: [showcase]
tags: [virtualisation, linux, vlab, spacewalk, kvm]
description: Setting up a spacewalk server in CentOS 7
lang: en
---
The first two steps of creating a virtual lab are:
1. Create a hypervisor
2. Install a [Spacewalk](https://spacewalkproject.github.io/) server

I'm using [KVM](https://www.linux-kvm.org/page/Main_Page) as my supplier for
the virtualisation infrastructure in combination with
[QEMU](https://www.qemu.org/). The Spacewalk server is set up as a new CentOS 7
vm. As the basic installation and activation of KVM is simple, this post is
solely focusing on the Spacewalk topic.

## Overview
Spacewalk is a software for managing Fedora and RHEL-like hosts. Management is
related to the basic environment overview (inventory), software installation /
updates and configuration management of hosts.

## Planning
To start I looked up the minimum system requirements. I then created the virtual
machine to install CentOS 7. The following specs where used:
- RAM: 2048 MB
- vCPUs: 1
- Disk space: 20 GB

You maybe notice, that the disk space differs from what I've initially planned.
This was a mistake and I needed to resize the disk later on (see Problems
below).
These are the steps for the implementation:
- Install CentOS 7
- Install Spacewalk 2.10
- Configure Spacewalk
  - Configure channels
  - Configure repositories
  - Set up errata importation

As the Spacewalk configuration will be altered in the future steps, the planning
done here, might seem not complete. I just wanted to install a basic server to
continue with the next steps. I think the real focusing is needed when migrating
the database to a PostgreSQL cluster.

## Implementation
I've decided to not invest too much time in configuring CentOS 7, as my main
goal is to get to know the software better. With this in mind, the CentOS
installation is kind of standard. Including the suggested partitioning scheme.
For the sake of completeness, this is the full system documentation of
non-standard changes to the os:
- OS: CentOS 7 (minimal)
- Language: English (United States)
- Keyboard Layout: German (no dead keys)
- Time & Date:
  - Region: Europe
  - City: Berlin
- Software Selection: Server
- Installation Destination: Automatic partitioning
- Root password: lab-Virt-02
- User:
  - Full name: Lab Admin
  - User name: labmin
  - Password: lab-Virt-02
- IP address: 192.168.122.15
- Gateway: 192.168.122.1
- DNS: 9.9.9.9 (for now)
- Seach domain: lab

The installation and configuration of the Spacewalk server was done with the
help of the
[user wiki](https://github.com/spacewalkproject/spacewalk/wiki/HowToInstall)
page. In the same manner as the system documentation, this is the Spacewalk
part:
- Admin email address: labmin@intramail.lab
- Automatically configure Apache's default SSL server
- CA certificate password: fn;sF.9M\_-
- CNAME alias: none
- Organization: Lab
- Organization unit: Spacewalk Server
- Email address: labmin@intramail.lab
- City: Waltrop
- State: NRW
- Country Code: DE
- Enable TFTP and XINETD (for PXE provisioning functionality)

This got me up to the point as I was able to access the main site and
login:

![Spacewalk overview](/assets/img/posts/vl1/spacewalk_overview.png#center)

On the web interface I was able to configure a few more details about the
Spacewalk installation.
- Organization Name: Lab
- Desired Login: labmin
- Desired Password: lab-Virt-02
- Email: labmin@intramail.lab
- First name: Mr. Lab
- Last name: Min

I then proceeded to configure automatic errata importation, with the help of a
[common perl script](https://cefs.steve-meier.de/). The automation was done by
creating a
[daily cronjob script](http://www.stankowic-development.net/?p=8661&lang=en).
At this moment I encountered fault #2. The script wasn't able to connect the
errata information with neither any channels nor packages. So I needed to set
them up too. This was a rather straightforward task (when done over the web
ui). The following channels were created (with fitting repositories linked to
each of them):
- centos7-base-x86\_64
- centos7-extras-x86\_64
- centos7-updates-x86\_64
- epel-el7-x86\_64

After all packages were downloaded, an errata test importation was also
successfully:

![Spacewalk errata](/assets/img/posts/vl1/spacewalk_errata.png#center)

## Problems
- 20 GB but need 50 GB
- CentOS 8 but can't really install it (no manual found)
- no errata imported

## Lessons learned
- Need to read more attentive.
