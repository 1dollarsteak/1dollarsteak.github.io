---
layout: post
title: vLab 3 - LDAP
categories: [showcase]
tags: [virtualisation, linux, vlab, kvm, openldap, letsencrypt]
description: Setting up two OpenLDAP server in CentOS 7
date: 2020-07-14
lang: en
---
It's done. Finally. I really hope that my next tasks will be a little bit less
time consuming. The learning curve is currently a little bit flat. While working
on this, I was once reminded of my everyday favourite errors. SELinux and
firewall rules. I just wasted too much time on these two.

## Overview
Task #3 doesn't solely exists of creating an LDAP service. In detail it looks
like this:
- Create two virtual machines from Spacewalk (without touching them)
- Install and configure a basic OpenLDAP service
- Set up a master/master replication between these two hosts
- Disable anonymous bind
- Allow only TLS secured connections (replication & queries)

The errors I experienced where numerous. I was also on the brink of giving up
and just creating the servers by hand. Needless to say, after my motivation was
completely diminished, something started to work and I was able to moved on.

## Planning
As with the DNS server in the last post, I'll equip the LDAP server with 1 GB
RAM, 20 GB disk space and 1 core each. As there are two servers I think the
scaling I chose is sufficient.

I neither set up (or kickstarted) a server automatically, nor did I install an
OpenLDAP server. So for the planning part, best I could do was vague guessing of
the needed steps:
- Create new kickstart profile
- Startup two vms and finish the configuration
- Install and configure OpenLDAP
- Get and install a certificate
- Configure master/master replication



## Implementation
The system is going to be CentOS 7 as before. I'll not go into detail as in the
last post, because I think the installation process should be clear. Apart from
that, this is where I encountered my first issue (more on that in the section
Problems).

Despite that I already have installed DNS and DHCP servers in the past I
conducted several manuals in hope of learning something new and to decrease the
possibility to create issues I'll stumble upon in the later process.
These are the links to the manuals:
- [DNS Server](https://www.itzgeek.com/how-tos/linux/centos-how-tos/configure-dns-bind-server-on-centos-7-rhel-7.html)
- [DHCP Server](https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-and-configure-dhcp-server-on-centos-7-ubuntu-14-04.html)

The software was installed by using yum. This is the named configuration:
{% highlight bash %}
#/etc/named.conf
zone "." IN {
        type hint;
        file "/var/named/named.ca";
};

zone "lab.fabianbissmann.de" IN {
        type master;
        file "/var/named/lab.fabianbissmann.de.db";
        allow-update { none; };
};

zone "122.168.192.in-addr.arpa" IN {
        type master;
        file "/var/named/192.168.122.db";
        allow-update { none; };
};
{% endhighlight %}

After setting up the zones I had to create the zone files, with all entrys of
the above table (that I planned). There is nothing of much interest. Just to
give you an idea of it:
{% highlight text %}
/var/named/lab.fabianbissmann.de.db
@ IN SOA labdns1.lab.fabianbissmann.de. hostmaster.lab.fabianbissmann.de. (
        2020053001 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        600 )      ; negative caching

; Name server information
@ IN NS labdns1.lab.fabianbissmann.de.

; IP address of name server
labdns1 IN A 192.168.122.10
...
{% endhighlight %}
One thing worth to mention, is that I numbered the serial according to the date
of creation (plus 2 digits for changes). Every change this number needs to get
bigger. To make it easier to guess, when the last change to the dns was, I've
chosen to make the serial look like a timestamp.

The configuration for the DHCP server looks like this:
{% highlight bash %}
#/etc/dhcp/dhcpd.conf
option domain-name "lab.fabianbissmann.de";
option domain-name-servers labdns1.lab.fabianbissmann.de;

default-lease-time 600;
max-lease-time 7200;

authoritative;
subnet 192.168.122.0 netmask 255.255.255.0 {
        range 192.168.122.100 192.168.122.199;
        option routers 192.168.122.1;
        option broadcast-address 192.168.122.255;
        next-server 192.168.122.20;
        filename "/pxelinux.0";
}
{% endhighlight %}
Also, nothing special here. The next-server is the PXE server (it's the
Spacewalk server). It's advised to disable the QEMU-internal dhcp server (if
started before). For this to achieve, I had to edit the network configuration
of the virtual network via the virsh application and delete the dhcp part.
Luckily I got this information pretty fast from
[here](https://unix.stackexchange.com/questions/35291/qemu-kvm-and-internal-dhcp-server).

The last step was to enable the Spacewalk server as a target for the PXE
feature. I went along the official CentOS
[manual](https://wiki.centos.org/HowTos/PackageManagement/Spacewalk#Populating_the_distribution_tree) for setting it up For it to set up, I had to download a CentOS ISO
and mount it inside the labsmt1 server. After that, I should be able to
configure a distribution with the mounted ISO as a path to the boot image
inside. It turned out that:
1. ISO files (loop devices) get mounted read only
2. SELinux doesn't allow Spacewalk to read or access it.

More on that in the problems section. After I overcame these *features*,
Spacewalk was able to access the files in the mounted iso and the distribution
was successfully set up.

![Spacewalk distribution settings](/assets/img/posts/vl2/spacewalk_distribution_settings.png#center)

![Spacewalk distribution list](/assets/img/posts/vl2/spacewalk_distributions.png#center)

## Problems
### More RAM needed when installing
As I've planned to assign this vm only 1 GB of RAM, the installation of the base
system lasted significantly longer, than with 2 GB. When I'm installing new
servers, the minimum RAM for installation will always be 2 GB. Later when
everything is running, the RAM can be lowered to the planned value.

Time spent: 30 minutes.
### No gateway was configured
This is just a common mistake I encounter relatively often. I categorize these
as careless mistakes, as they can be easily avoided. Just after I was finishing
the configuration of the dns server and after a reboot, I noticed, that it
wasn't possible to resolve any public domains. It happened after I set up a
manual network address, to turn of the QEMU-provided DHCP server. I forgot to
add a default gateway. At the time I figured this out, I was already inspecting
the traffic with tcpdump.

Time spent: 1 hour.
### Spacewalk distribution setup failed
At the stage where I needed to create a distribution in Spacewalk for providing
a PXE service, there first was no possibility where I could mount the iso with
write access. After I learned that this is not in any way sensible, I've tried
to figure out what is preventing Spacewalk to read the content of the mounted
iso.

I mounted the iso with configuring the following fstab entry:
{% highlight text %}
/var/iso-images/CentOS-7-x86_64-Minimal-2003.iso /var/distro-trees/CentOS-7-x86_64 iso9660 ro,loop 0 0
{% endhighlight %}
While searching for more information on the error message, the website of the
Spacewalk server gave me, I saw a thread where a user was experiencing the same
issue. The solution there was to change the SELinux security context. I've tried
different approaches but none of them worked:
{% highlight bash %}
# Adding spacewalk_data_t to the context to access the directory
semanage fcontext -a -t spacewalk_data_t "/var/distro-trees(/.*)?"
# Checking for the context
ls -lZ /var/distro-trees/CentOS-7-x86_64/
# Changing the context to the executing service
sudo chcon -Rt tomcat_t /var/distro-trees/
# Changing the context to a more general group (?)
sudo chcon -t httpd_sys_content_t /var/distro-trees/
{% endhighlight %}
I was conscious of the possibility to just shut off SELinux, but I wanted to
know how the issue can be resolved more elegantly. As I found out, there are
existing fedora based helper utilities, that give fix suggestions based on
listed access errors inside the audit.log. I went, installed and executed the
sealert software package. After executing I was able to fix the 2 errors with
the following commands (twice executed after a second error after the first
run):
{% highlight bash %}
ausearch -c 'ajp-bio-0:0:0:0' --raw | audit2allow -M my-ajpbio0000
semodule -i my-ajpbio0000.pp
{% endhighlight %}

For reference these are the two error messages I've had in audit.log:
{% highlight text %}
type=AVC msg=audit(1591090111.335:172): avc:  denied  { search } for  pid=1010 comm="ajp-bio-0:0:0:0" name="/" dev="loop0" ino=1856 scontext=system_u:system_r:tomcat_t:s0 tcontext=system_u:object_r:iso9660_t:s0 tclass=dir permissive=0
type=SYSCALL msg=audit(1591090111.335:172): arch=c000003e syscall=4 success=no exit=-13 a0=7f650800f3a0 a1=7f64f79f6d70 a2=7f64f79f6d70 a3=736567616d692f34 items=0 ppid=1 pid=1010 auid=4294967295 uid=53 gid=53 euid=53 suid=53 fsuid=53 egid=53 sgid=53 fsgid=53 tty=(none) ses=4294967295 comm="ajp-bio-0:0:0:0" exe="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/bin/java" subj=system_u:system_r:tomcat_t:s0 key=(null)

type=AVC msg=audit(1591090841.371:217): avc:  denied  { getattr } for  pid=1010 comm="ajp-bio-0:0:0:0" path="/var/distro-trees/CentOS-7-x86_64/images/pxeboot/initrd.img" dev="loop0" ino=4550 scontext=system_u:system_r:tomcat_t:s0 tcontext=system_u:object_r:iso9660_t:s0 tclass=file permissive=0
type=SYSCALL msg=audit(1591090841.371:217): arch=c000003e syscall=4 success=no exit=-13 a0=7f6514064860 a1=7f64f77f4d70 a2=7f64f77f4d70 a3=736567616d692f34 items=0 ppid=1 pid=1010 auid=4294967295 uid=53 gid=53 euid=53 suid=53 fsuid=53 egid=53 sgid=53 fsgid=53 tty=(none) ses=4294967295 comm="ajp-bio-0:0:0:0" exe="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/bin/java" subj=system_u:system_r:tomcat_t:s0 key=(null)
{% endhighlight %}

I also needed help solving this from these links:
- <https://www.redhat.com/archives/spacewalk-list/2018-April/msg00020.html>
- <https://danwalsh.livejournal.com/67007.html>
- <https://www.reddit.com/r/linuxadmin/comments/5cl5bs/configuring_cobbler_in_spacewalk_for_pxe_boot/>
- <https://spacewalk-list.redhat.narkive.com/Ifz8VwiT/issues-with-kickstart>
- <https://www.linuxquestions.org/questions/linux-security-4/selinux-how-to-list-all-type-enforcement-contexts-that-exist-on-the-system-843654/>
- <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/selinux_users_and_administrators_guide/index>

I hope that my next 2 vms will successfully boot.

Time spent: 5 hours.
### Authorization not available (boot error)
After I created a fstab entry for the iso file for the Spacewalk distribution, a
boot of the server wasn't anymore possible and It dropped me to a rescue shell,
with a note saying that the authorization was not available and a boot error
occurred. It was probably a misspelling. After I readded the entry, everything
worked.

Time spent: 15 minutes.

## Lessons learned
The SELinux error lasted longer than I could have imagined before, but that was
also the time where I learned the most. Otherwise this part wasn't a challenge,
but it was neat to have the possibility to setup bind and isc-dhcp once again.
Another summary for completeness:
- SELinux basic troubleshooting and function
- BIND, ISC-DHCP configuration and installation refreshed
- Spacewalk Kickstart basics
