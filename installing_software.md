# Installing software

Much of the time, "bioinformatics" seems like little more than installing software and chaining them together with scripts.  Sometimes you may be lucky enough to have a "sysadmin" (systems administrator) who can assist you, but most of the time you'll find yourself needing to take care of business yourself.  

My suggestions for installing software are (in order):

## Package managers

There are several package management systems for Linux and OSX including apt-get, yum, homebrew, macports, and more.  These usually relieve the problems of software compatibility and shared libraries.  

## Binary installations

Quite often you'll be happy to find that the maintainers of the software you need have gone to the trouble to build binary distributions for your system, which is likely to be a generic 64-bit Linux platform.  Often you can just download the binaries and put them into your $PATH.  There is usually a "README" or "INSTALL" file that will explain exactly what to do.

## Source installations

Installing from source usually means downloading a "tarball" ("tar" = "tape archive," a container of files, that is then compressed with a program like "gzip" to create a ".tar.gz" or ".tgz" file extension), running "configure" to figure out how it can build on your system, and then "make" to build the binaries.  Usually you will run "make install" to put the binaries into their proper directory, but sometimes you just "make" and copy the files yourself.

The basic steps are usually:

```
$ tar xvf package.tgz
$ ./configure [--prefix=$HOME/local]
$ make && make install
```

When I'm in an environment with a directory I can share with my team (like the UA HPC), I'll configure the package to install into that shared space so that others can use the program.  When I'm on a system like "stampede" where I cannot share with others, I'll usually install into my "$HOME/local" directory.

