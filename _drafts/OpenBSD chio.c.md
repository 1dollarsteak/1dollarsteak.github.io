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
My work will base on Theo de Raadt's version (1.26) from 2019. The Source can be
found
[here](https://github.com/openbsd/src/blob/master/bin/chio/chio.c). This program
is used to control medium changers (such as tape drives). And was initially
written by Jason R. Thorpe. Different operations (for example moving or
exchanging) can be performed.
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
[mtio.h](https://man.openbsd.org/mtio) is a header file, that defines structures
and operations for _typical tape block devices_. There is one location in the
code, where this interface is used. It's in the function check_source_drive as
a measure of troubleshooting, if the drive in the tape device is full but not
accessible.
{% highlight c %}
static void
check_source_drive(int unit)
{
	struct mtop mtoffl =  { MTOFFL, 1 };
  // ...
  if (ioctl(mtfd, MTIOCTOP, &mtoffl) == -1)
		err(1, "%s drive %d (%s): rewoffl", changer_name, unit, tapedev);
	close(mtfd);
}
{% endhighlight %}
The appropriate definition can be looked up in the source code of the
corresponding
[header file](https://github.com/openbsd/src/blob/master/sys/sys/mtio.h):
{% highlight c %}
/* structure for MTIOCTOP - mag tape op command */
struct mtop {
	short	mt_op;		/* operations defined below */
	int	mt_count;	/* how many of them */
};

// ...
#define MTOFFL		6	/* rewind and put the drive offline */
// ...

/* mag tape io control commands */
#define	MTIOCTOP	_IOW('m', 1, struct mtop)	/* do a mag tape op */
{% endhighlight %}
The resolved function call would be: `_IOW('m', 1, {MTOFFL, 1});`. Note that
this is syntactically incorrect C. I've decided to pass the struct anonymously
to make the call more readable.
### chio.h
Based on the name of the next header file, it should be clear, that it can be
counted towards the more important files in this program. Everything is in use.
The file is separable in 4 sections:
- Element types and flags
- Data structures
- Return values
- IOCTL macros for controller / device communication
#### Element types and flags
This defines the ids for the elements of a tape device:
{% highlight c %}
/*
 * Element types.  Used as "to" and "from" type indicators in move
 * and exchange operations.
 * ...
*/
#define CHET_MT		0	/* medium transport (picker) */
#define CHET_ST		1	/* storage transport (slot) */
#define CHET_IE		2	/* import/export (portal) */
#define CHET_DT		3	/* data transfer (drive) */
{% endhighlight %}

Usage of elements[] in chio.c:
{% highlight c %}
const struct element_type elements[] = {
	{ "drive",		CHET_DT },
	{ "picker",		CHET_MT },
	{ "portal",		CHET_IE },
	{ "slot",		CHET_ST },
	{ NULL,			0 },
};
{% endhighlight %}

Declaration of element_type in local header file defs.h:
{% highlight c %}
struct element_type {
char  *et_name; /* name; i.e. "picker, "slot", etc. */
  int et_type;  /* type number */
};
{% endhighlight %}

The preceeding code shows the way in which different options get managed inside
chio.c. Through the struct element_type the connection between the id as a
number and a more distinguishable name for the element type is made.  
Finally inside chio.c the struct is parsed by this function:
{% highlight c %}
static int
parse_element_type(char *cp)
{
	int i;

	for (i = 0; elements[i].et_name != NULL; ++i)
		if (strcmp(elements[i].et_name, cp) == 0)
			return (elements[i].et_type);

	errx(1, "invalid element type `%s'", cp);
}
{% endhighlight %}
There the struct is traversed until the matching name can be found. Afterwards
the internal id is returned.

### err.h
There are 11 occurences of the warnx function, 17 of err and finally 11
times the errx function was called. The difference between err and the
errx/warnx variations are, that when err is called, also the error
message is looked up in [errno.2](https://man.openbsd.org/errno.2) and
appended to the fmt string (if not NULL).
### errno.h
These are the error message definitions that are looked up when err is
called.
### fcntl.h
Because chio works with files, this header is also clearly needed. The
creation of the changer file descriptor is achieved through open:
{% highlight c %}
if ((changer_fd = open(changer_name, O_RDWR, 0600)) == -1)
    err(1, "%s: open", changer_name);
{% endhighlight %}
### limits.h
This header file defines the size range that the different data types
can store. Nevertheless it is not used in this file. The only occurences
I came across are two checking for the range of a long in parse.y.
Though, there it's also added to the list of header files.
### stdio.h
This is used to get access to the streams `stderr` and `stdout` for
various usage in the functions warn, err and (f)printf. User input is
not expected / handled, because this program is solely based on a
configuration file.
### stdlib.h
The most interesting usage of functions in stdlib is calloc and free.
Next to these the following are used: exit, getenv, strtol. I'll not go
into deeper details with the latter. Memory allocation is used to store
the status info that gets queried from the changer device with
`ioctl(changer_fd, CHIOGSTATUS, &cmd)`. Here `&cmd` is the address of a
specific struct where volume labels (primary and alternative) can be
stored. They are later printed out with their corresponding serial
strings.
### string.h
No functions or other resources of this header file are actually in use.
### unistd.h
Also (check)
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
