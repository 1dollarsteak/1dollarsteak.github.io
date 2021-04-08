---
layout: post
title: OpenBSD - chio.c
categories: [explanation]
tags: [programming, c, flowchart]
description: A thorough explanation of the OpenBSD version of the chio.c program
date: 2021-04-08
lang: en
---

Afterall, why not continue with the next file in line? In this post I will
describe the features of the chio program. According to
[the manpage](https://man.openbsd.org/chio) this program was written by Jason R.
Thorpe for And Communications in 1996 (unverified). Sadly, the actual home page
for this company doesn't exist anymore. But let's focus more on the program.

## Overview
My work will base on Theo de Raadt's version (1.26) from 2019. Source can be
found
[here](https://github.com/bluerise/openbsd-src/blob/master/bin/chio/chio.c).
This program is used to control medium changers (such as tape drives). Different
operations (for example moving or exchanging) can be performed.
## Header files
Here is the list of all headers in use:
{% highlight c %}
#include <sys/types.h>
#include <sys/ioctl.h>
#include <sys/mtio.h>
#include <sys/chio.h>
#include <err.h>
#include <errno.h>
#include <fcntl.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <util.h>

#include "defs.h"
#include "pathnames.h"
{% endhighlight %}
### Already specified header files
For the following headers: types.h, err.h, errno.h, fcntl.h, stdio.h, stdlib.h,
string.h, unistd.h, I'd like you to refer to the following site to get an
explanation of it. Nevertheless I'll shortly address the usage of functions in
this program for these headers.
### types.h
The only things used from the types header are a few counter variables declared
with the type size_t. This guarantees that it's an unsigned integer. For example
on my system the definition of size_t is `typedef long unsigned int size_t;`.
### ioctl.h
ioctl.h is used to access the function
`ioctl(int d, unsigned long request, ...);`. ioctl is used to control devices
over special device files. A file descriptor (d) is needed, following further
arguments (int or pointer to specific structure) for running the requested
function on the device.
In this utility the function is used in conjunction with definitions of the file
sys/chio.h (see below). The general execution of actions is realised with ioctl:
`ioctl(changer_fd, CHIOMOVE, &cmd)`. Here the file descriptor of the changer is
chosen. Also CHIOMOVE is used as the definition of the command to be executed:
`#define CHIOMOVE	_IOW('c', 0x41, struct changer_move)`. In here 'c' is used to
specify the driver
(see [here](https://01.org/linuxgraphics/gfx-docs/drm/ioctl/ioctl-number.html)).
The letter c seems to be shared with a bunch of different drivers. The second
parameter exists to identify the ioctl action and the last parameter defines the
expected datatype.
This aside we can expect that &cmd is an address to a struct of the type
changer_move. Wich is correct: `struct changer_move cmd;`. Further, this is the
definition of the struct (found
[here](https://github.com/bluerise/openbsd-src/blob/master/sys/sys/chio.h)):
{% highlight c %}
struct changer_move {
	int	cm_fromtype;	/* element type to move from */
	int	cm_fromunit;	/* logical unit of from element */
	int	cm_totype;	/* element type to move to */
	int	cm_tounit;	/* logical unit of to element */
	int	cm_flags;	/* misc. flags */
};
{% endhighlight %}
### mtio.h
### chio.h
### err.h
### errno.h
### fcntl.h
### limits.h
### stdio.h
### stdlib.h
### string.h
### unistd.h
### util.h
### defs.h (local)
### pathnames.h (local)

## Functions
### int main
### static int do_move
### static int do_exchange
### static int do_position
### static int do_params
### static int do_getpicker
### static int do_setpicker
### static int do_status
### static void check_source_drive
### void find_voltag
### static int parse_element_type
### static int parse_element_unit
### static int parse_special
### static int is_special
### static int bits_to_string
### static void usage

## Bottom line
