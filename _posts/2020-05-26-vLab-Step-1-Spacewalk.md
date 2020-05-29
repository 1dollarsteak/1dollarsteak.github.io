---
layout: post
title: vLab 1 - Spacewalk
categories: [showcase]
tags: [virtualisation, linux, vlab, spacewalk, kvm]
description: Setting up a spacewalk server in CentOS 7
date: 2020-05-26
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
At this moment I encountered fault #3. The script wasn't able to connect the
errata information with neither any channels nor packages. So I needed to set
them up too. This was a rather straightforward task (when done over the web
ui). The following channels were created (with fitting repositories linked to
each of them):
- centos7-base-x86\_64
- centos7-extras-x86\_64
- centos7-updates-x86\_64
- epel-el7-x86\_64

After all packages were downloaded, an errata test importation was also
successfull:

![Spacewalk errata](/assets/img/posts/vl1/spacewalk_errata.png#center)

## Problems
### CentOS 8 installation was not possible
In the beginning I wanted to start with a somewhat newer version of CentOS than
6 or 7. I decided to go with version 8. As I've never got in touch with CentOS
before, I was not expecting that you can't just use a manual that was written
for CentOS 7. The manual im reffering to is the Spacewalk installation guide. I
quickly abandonded my plan to use CentOS 8 and went with 7.

Time spend: 4 hours.

### The initial disk size was too small
I didn't pay enough attention, that the disk size in the act of creating the
virtual machine was not 50 GB as it was planned. Instead I've created a disk
with a size of 20 GB.

So I had to read about how to resize disks with QEMU. I've resized many disks
with various formats for many different hypervisors and operating systems. So,
I was confident that it was not a big deal. To summarize, these are the steps
I've taken:
1. Stopped the VM
2. Ran `qemu-img resize <disk> +30G`
3. Started the VM
4. Backed up the current partition table
5. Recreated the partition with the new end of the disk
6. Rebooted the VM
7. Checked the state with lsblk
8. Resized the LVM PV with pvresize
9. Resized the LVM LV with lvextend
10. Resized the filesystem with xfs\_growfs

After these steps I've checked everything twice and moved on to download the
additional packages from the mirrors.

Sources that helped me:
- <https://serverfault.com/questions/324281/how-do-you-increase-a-kvm-guests-disk-space#324314>
- <https://www.rootusers.com/lvm-resize-how-to-increase-an-lvm-partition/>
- <https://serverfault.com/questions/861517/centos-7-extend-partition-with-unallocated-space>

Time spend: 2 hours

### No errata was imported
A rather small error, that I was able to solve quickly. At the start of the
configuration I was eager to import every errata information that I could get
a hold on. It turned out that without any downloaded packages, it's not possible
to download and catalog errata information. Also it was a misleading error
message that quite not described the cause. Only something in general.

Time spend: 30 minutes

## Lessons learned
I think on the bottom line I should read more attentively, because these errors
may never occurred, when I've strictly followed the guideline. On the other side
I am graceful that I ran into these things. So to sum up:
- KVM / QEMU basic operation
- CentOS 7 basic installation
- Spacewalk basic installation and configuration
- Spacewalk errata management on CentOS 7
- Basic QEMU / CentOS 7 / LVM disk resizing
