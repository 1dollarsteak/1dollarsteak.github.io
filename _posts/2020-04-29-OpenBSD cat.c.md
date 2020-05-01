---
layout: post
title: OpenBSD - cat.c
categories: [explanation]
tags: [programming, c, flowchart]
description: A thorough explanation of the OpenBSD version of the cat.c program
lang: en
---

A few years ago I've read about looking at and understanding one source
file of the OpenBSD operating system a day. And since then I've always
wanted to start with it. Though I've only got to the first file, since
then. Because I don't have much free time to devote to this activity, I
highly doubt it's possible to satisfy the given interval at reading and
understanding. Nevertheless, I want to share my discoveries with you and
show you some (not all, hopefully) files I deemed as interesting and
worthy to describe in depth. So I thought why don't give this a little
motivational bump and write about it in my blog.


To begin with I want to focus my view on cat.c.
## Overview
Cat is a program everyone using a linux or mac operating system should
know about, after a few days of working with the os. It's derived from
the word concatenate and is used to add the contents of files together
and output it on the standart output. As simple as it's function sounds,
the source file is (license included) 249 SLOC long. Also it's making
use of helper functions out of ten header files.
## Header files
This is the list of the header files I mentioned above:
```c
#include <sys/types.h>
#include <sys/stat.h>

#include <ctype.h>
#include <err.h>
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
```
### types.h
The file types.h defines extra types for variables. The only usage of
types out of this header file is `size_t` and `ssize_t`. According to
the NetBSD-8.1 man page for
[types](https://man.openbsd.org/NetBSD-8.1/types), `size_t` is typically
used for size declaration of objects and `ssize_t` is used for counting
of bytes. Usually these two are defined like this (depending on your
architecture):
```c
typedef unsigned long __size_t;
typedef long __ssize_t;
```
### stat.h
With the functions, variables and structs defined in stat.h you are able
to retrieve information from various files. In cat.c `int fstat(int,
struct stat *);` is used to copy information about the standard
outstream (passed as a file descriptor) into a stat struct. The stat
struct (big surprise) is also defined in stat.h and inhabits all
important data about files. Out of this struct only `st_blksize` is
used. This variable states _"the optimal I/O block size for the file"_
([stat(2)](https://man.openbsd.org/fstat.2)).
### ctype.h
The source code uses functions out of this header file to check if the given
char is a printable, or an invisible control character. You can achieve this
behaviour with the v flag ([cat(1)](https://man.openbsd.org/cat)).
The two functions used are `__only_inline int isascii(int _c)` and 
`__only_inline int iscntrl(int _c)`. Furthermore there is
`__only_inline int toascii(int _c)` to convert any given char into an ascii 
value. Before I take a look at both definitions of these functions, I want to
shine some (really not much) light on the declaration. You may notice that 
there is a special keyword before int. Defined in 
[cdefs.h](https://github.com/openbsd/src/blob/master/sys/sys/cdefs.h) it
manages inlining of functions. The comment in the source file describes it
best:
```c
/*
 * __only_inline makes the compiler only use this function definition
 * for inlining; references that can't be inlined will be left as
 * external references instead of generating a local copy. The
 * matching library should include a simple extern definition for
 * the function to handle those references. c.f. ctype.h
 */
```
In general, inlining of extern or static functions has something to do with
optimization. The definition of all three mentioned functions are very simple
oneliners. In my opinion the less interesting one is
`__only_inline int toascii(int _c)`.
```c
__only_inline int toascii(int _c)
{
  return (_c & 0177);
}
```
As you pass any char value to this function it converts it to an integer value
(because of `int _c`). Afterwards it applies the variable with a bitwise AND
operation to the octal value 0177 (that's the last value of the ascii table).
With this you can expect that the given parameter is always in the range of the
ascii table. `__only_inline int toascii(int_c)` works with the same method in
mind. _"The isascii() function returns zero if the character tests false or 
non-zero if the character tests true."_
([isascii(3)](https://man.openbsd.org/isascii.3)). The man page also states: _"
The isascii() function tests for an ASCII character, wich is any character with
a value less than or equal to 0177."_.
The function definition of iscntrl makes an interesting use of a pointer
assignment to a char array with the field size of the integer value of it.
```c
//...
#define _C 0x20
extern const char *_ctype_;
//...
__only_inline int iscntrl(int _c)
{
  return (_c == -1 ? 0 : ((_ctype_ + 1)[(unsigned char)_c] & _C));
}
```
The most interesting part is `(_ctype_ + 1)[(unsigned char)_c]`. As I've
already dug deeper into this header file than I initially wanted, I'll not go
into it any further and let this function abide marked as interesting. (Also
because I don't understand how this works yet, after a few days of puzzling.)
### err.h
This header file declares functions that print out warning or error messages on
the related output stream. In the cat.c source code I found usages of
`__dead void err(int, const char *, ...)` and `void warn(const char *, ..)`.
The `__dead` keyword defines a behaviour that the given function does not
return. Instead it exits the program with the global variable `errno` as the 
return value.
### errno.h
The global variable mentioned above is defined in this header. The declaration
is simple:
```c
#ifndef errno
int *__errno(void);
#define errno (*__errno())
#endif /* errno */
```
So, `errno` returns an integer value. A list of different possible values for
this error value can be extracted from
[this](https://github.com/openbsd/src/blob/master/sys/sys/errno.h) source file.
Also further information about the usage of `errno` can be acquired by reading
through [errno(2)](https://man.openbsd.org/errno.2).

I didn't dug deep on these
two files, because I think that error handling doesn't need to be as thoroughly
explained as checking char values in ctype.h. At least when it comes to
understanding the source code of cat.c.
### fcntl.h
This header file defines several functions to work with files in different
modes (read only, write, ...). The `int open(const char *, int, ...);` function
is used in cat.c to check if a file, that was passed as an argument, can be
opened for reading (and open it afterwards).
```c
else if ((fd = open(*argv, 0_RDONLY, 0)) == -1) {
  warn("%s", *argv);
  rval = 1;
  ++argv;
  continue;
}
```
This happens in the cat.c function `void raw_args(char **argv)`. If opening the
file fails, it prints out a warning, jumps to the next argument (file) and
continues the loop the if clause resides in, The file descriptor is in the next
step passed to the function `void raw_cat(int rfd)` to print it on the standard
output, as already mentioned above.
### stdio.h
The stdio.h library is, as the name already states the standard for input / 
output in terms of streams. An excerpt from the manual sums it up like this:
_"The standard I/O library provides a simple and efficient buffered stream I/O
interface. Input and output is mapped into logical data streams and the
physical I/O characteristics are concealed._
[stdio(3)](https://man.openbsd.org/stdio.3).
In general that means you have functions declared that print out something
somewhere and functions that get input data from the user. Surely this header
file is way more complex, but for cat.c this (kind of) sums it up. An important
last thing to note is that the famous I/O streams (stdin, stdout and stderr)
are declared in stdio.h.

With these things in mind we can move on and focus on the usages of this header
in cat.c.

As this header file defines many functions for general I/O and cat.c nearly
only task is to receive stuff and print it out, the usage of functions and
variables out of this file is very high. To access the standard I/O of the
operating system, cat.c makes use of the common streams `stdout, stdin, stderr`.
These streams get opened at program startup. In the header file they are defined
like this:
```c
extern FILE __sF[];

#define stdin (&__sF[0])
#define stdout (&__sF[1])
#define stdout (&__sF[2])
```
`FILE` is a struct with different flags, options and methods for reading and
writing to files. `FILE` is frequently used throughout stdio.h as a return value
or as a type for arguments to a function.
Next `int fprintf(FILE *, const char *, ...);` is used to write output to the
supplied stream pointer stream
[printf.3](https://man.openbsd.org/OpenBSD-5.2/printf.3). The argument list is
the content that has to be written using the first argument. It easily 
recognizable as the standard
[printf format string](https://en.wikipedia.org/wiki/Printf_format_string<Paste>)
. I was startled that there is a page on Wikipedia that describes this type of
format in a mostly adequate depth.

As you maybe can guess from the names of the next functions,
`int fclose(FILE *);` and `FILE *fopen(const char *, const char *);` are used
for closing and opening streams. The 2 arguments inside `fopen` represent the
path to a file, where a stream should be associated with and a mode flag to
specify how the file is going to be opened (reading / writing /
[etc](https://man.openbsd.org/OpenBSD-5.2/fopen.3#DESCRIPTION)).
`int putchar(int c);` is a function that converts the int parameter c to
an unsigned character and prints it to the stdout stream. `int
fileno(FILE *stream);` takes a stream as an argument and returns its
integer descriptor
[clearerr(3)](https://man.openbsd.org/NetBSD-7.0.1/clearerr.3).
The function `int setvbuf(FILE *stream, char *buf, int mode, size_t
size);` is a function I personally really like. With it you can control
the buffering behavior of a stream. You can set it to unbuffered, where
input is directly passed to the output stream, or line buffered where
content is passed after a newline character (\n) on the input side. The
last mode is block buffered. Characters are saved up and released after
the count reaches a specific value. With the pointer `buf` you can
reassign the standard buffer to a custom one along with the value size,
which states how big the buffer has to be.
As streams can include an error indicator that shows if any read or
write errors occured, a function to clear these errors is needed. `void
clearerr(FILE *);` is such a function, that has this effect on streams.
Also it clears the end-of-file indicator of the stream. In cat.c it's
used for example to clear the stream after an uncritical error is
identified, which is interpreted as a warning.
`int getc(FILE *);` simply gets the next character that's on the stream.
### stdlib.h
A definition of the stdlib.h header file on Wikipedia of what the header
contains is: _"Defines numeric conversion functions, pseudo-random
numbers generation functions, memory allocation, process control
functions"_ - [Wikipedia](https://en.wikipedia.org/wiki/C_standard_library)
The function that's used inside cat.c is for memory allocation. With
`void *malloc(size_t);` space for a buffer is allocated inside this
piece of code:
```c
if (buf == NULL) {
  if (fstat(wfd, &sbuf) == -1)
    err(1, "stdout");
  bsize = MAXIMUM(sbuf.st_blksize, BUFSIZ);
  if ((buf = malloc(bsize)) == NULL)
    err(1, "malloc");
}
```
There is a lot of error detection going on here and the only really
interesting line is malloc itself inside the third if clause. Speaking
about memory allocation, OpenBSD comes with functions that extend the
features of it and make memory allocation safer with preventing
overflows. See: [free(3)](https://man.openbsd.org/free.3).
### string.h
As the C standard doesn't come with a string datatype, the string header
provides many different functions to work with strings. The only
function used is `int strcmp(const char *, const char *);`, where two
strings are compared. The manual entry explains the return value of it
very well in a few words: _"The strcmp() and strncmp() functions return
an integer greater than, equal to, or less than 0, according to whether
the string s1 is greater than, equal to, or less than the string s2."_ -
[strcmp(3)](https://man.openbsd.org/strcmp.3)
In cat.c it is used to check if a dash is given as an argument. If
that's the case it sets the input _FILE_ pointer to the standard input
(stdin).
### unistd.h
unistd.h is the header file that provides interfaces to operate with
functions of UNIX or UNIX like operating systems. As in the previous
libraries, the use of this is also rather small in cat.c. Only `int
getopt(int, char * const *, const char *);` is used. With this function
you can search the next valid option character from the command line
argument list (*argv). The last argument specifies a list of available
arguments. In the following section I'll explain how this check is
implemented.
## Functions
Overall there a five functions in cat.c. In this chapter I'll describe the
content of each with the help of a diagram with standard flowchart like symbols.
### int main(...)
As the main function and the starting point, this function checks the passed
parameters to cat and sets additional flags accordingly. A function is run
depending on if options were passed or not.

![main flowchart](/assets/svg/cat_main.svg#center)

The first thing to note after the start node is the usage of 
[pledge](https://man.openbsd.org/pledge.2). Pledge gives the possibility to
restrict the permissions of the program to access system operations. After
pledge was successfully ran, main enters a loop in wich every passed option is
checked. Appropriate flags are set to identify the options later while the loop
is processed. The argv string is afterwards added with the number of options
(optind). This is done to handle passed files (behind the options) afterwards
more easily. If any flag is set, cook_args is run or else raw_args. In the end
the standard out is closed and if all went ok with no errors, rval is returned
and cat exits.
### void cook_args(...)
As noted in the description of the main function, if any flags were set when cat
was run, this function is called. In general cook_args opens every passed file
(including stdin -) and runs cook_buf on it. Also a bit of error checking is
done in this function. Let's get into some detail.

![cook_args flowchart](/assets/svg/cat_cook_args.svg#center)

After the assignment of stdin to the filepointer variable and the string "stdin"
to the filename variable cook_args enters a loop. I'd like to add here a small
comment about choices regarding the design of the flow chart. You may notice 
that there are two identical decision objects in this chart. It's where 
*argv != 0 is checked. I've added one at the start and one at the end because 
the source code just acts the same way. A small excerpt to visualize:
```c
//...
do {
  if (*argv) {
    if (!strcmp(*argv, "-"))
      fp = stdin;
    else if ((fp = fopen(*argv, "r")) == NULL) {
      warn("%s", *argv);
      rval = 1;
      ++argv;
      continue;
    }
    filename = *argv++;
  }
  //...
} while (*argv);
//...
```
Have a look at line three and 15. There you can see that the check is performed
twice. This is what I have tried to depict in my flow chart. You can see here,
that if cat is run with no "file" arguments passed (that means no arguments
after the options) it passes stdin as the fp to cook_buf. If it contains any
file arguments, the first check is if it contains "-" and if yes it sets the fp
to stdin.
### void raw_args(...)
This function is very similar to cook_args, described earlier. But because there
are no flags set (checked inside main), the function doesn't need to utilize a
file pointer like his "sibling". cook_args would create a file pointer (fp) and
point it to stdin or any passed file to then forward it to cook_buf. This is
because cook_buf needs access to higher level functions to fulfil the actions
requested by the flags.

![raw_args flowchart](/assets/svg/cat_raw_args.svg#center)

As mentioned above no flags have to be checked. This function forgoes the use of
fopen to open passed files to read the content of and just uses the function
open on it. The main difference is that with fopen you get a FILE * object to
work on. It also comes with a buffering I/O and paves the way for different 
stdio functions (more on that in section cook_buf).
### void raw_cat(int rfd)
raw_cat is responsible for putting out content passed to cat without any
parameter. The function can be divided in 3 parts.
- Allocate memory for the buffer
- Read file (or stdin) for a earlier set up number of bytes
- Write content to stdout

![raw_cat flowchart](/assets/svg/cat_raw_cat.svg#center)

At first raw_cat defines variables for accessing stdout and storing information
about it. This information is then used to define a fixed buffer size and later
also determine the number of bytes to read from the file descriptor that was
passed as a parameter to the function. The allocation of the buffer consists of
a single if clause and is accompanied by two error checks. After that the
function enters a rather simple read-file-print-file loop construct. To focus a
bit on how simple the loops are constructed, I have added the relevant snippet
below:
```c
while ((nr = read(rfd, buf, bsize)) != -1 && nr != 0)
  for (off = 0; nr; nr -= nw, off += nw)
    if ((nw = write(wfd, buf + off, (size_t)nr)) == 0 || nw == -1)
      err(1, "stdout");
```
As every action takes place inside the loops, no space is wasted. Just to take
a better shot at explaining it, I've allowed myself to expand this piece of code
a bit:
```c
nr = read(rfd, buf, bsize);
while (nr != -1 && nr != 0) {
  for (off = 0; nr; nr -= nw, off += nw) {
    nw = write(wfd, buf + off, (size_t)nr);
    if (nw == 0 || nw == -1) {
      err(1, "stdout");
    }
  }
  nr = read(rfd, buf, bsize);
}
```
First nr is set with bsize bytes out of the file that rfd links to. If
everything went well, the while loop is entered. The for loop is now responsible
to write from the buffer to the stdout stream. The variable off is used to
specify an offset for what was already printed to stdout. For every run inside 
the for loop, nr is reduced by the size of nw (already printed bytes). If nr
reaches 0 the next bytes is transfered in the buffer. This goes on until EOF is
reached. After that the program exits.

### void cook_buf(FILE *fp)
I've decided to place cook_buf at the end, because of it's size and because it's
the most complex function inside cat. The function is responsible for every
behavior that can be controlled with flags. To summarize (in short, for the full
description see: [cat(1)](https://man.openbsd.org/cat.1)), this covers:
1. -b: Number the lines, but don't count blank lines.
2. -e: Print a dollar sign (‘$’) at the end of each line.
3. -n: Number the output lines, starting at 1.
4. -s: Squeeze multiple adjacent empty lines.
5. -t: Print tab characters as ‘^I’.
6. -u: The output is guaranteed to be unbuffered.
7. -v: Displays non-printing characters so they are visible.

![cook_buf flowchart](/assets/svg/cat_cook_buf.svg#center)

To start the explanation of this function: A loop is entered at first
(at ch = getc(fp)), where ch is the next character from the passed stream
argument fp. It's then checked if it contains EOL. If yes it goes directly down
to ferror(fp) and the function (and (almost) cat itself) exits. If the previous
character was a newline, then #4 in the above list (_Squeeze multiple adjacent
empty lines_) is checked. If it's set and ch is also a newline char, then the
output of it is bypassed and the loop starts with reading in the next character.

Is the n flag set (_Number the output lines_), then it's checked if there is
also the b flag set (_don't count blank lines_) and then if the e flag is not
set (_print a $ sign at the end of each line_). The outcome is then depending on
the case of flags set. Bear in mind that if the n flag is set, there is also the
occurrence of the e flag (_dollar at end of line_) checked. If it's set, then
there is a tab after the line number (if the line is empty and counted). The
further checks for e, t and v are alike implemented. Inside the check for the
vflag, when the character is a control character, it's binary code is run
against the binary or of _0100_. This is just a commonly used technique to map
control characters to their suitable printable character.

After all these checks the character is outputted on the stream inside the if
clause (putchar(ch) == EOF). If this worked then the loop starts again. If not,
the fp parameter is checked for errors with ferror(fp).

### Bottom line
I know that the last flowchart is a bit confusing and overwhelming. But as I am
constantly improving my skills creating representive charts, I'm confident that
it'll get better over time. As the thorough explanation of cat is just the start
of a nice little side project, I'm eager awaiting the next challenges. Also to
be honest, this post took me a bit too long. Starting in December until the end
of April. I hope I can reduce this number in the next posts.
