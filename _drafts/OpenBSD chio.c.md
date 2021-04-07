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
### ioctl.h
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
