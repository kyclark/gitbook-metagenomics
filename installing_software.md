# Installing software

Much of the time, "bioinformatics" seems like little more than installing software and chaining them together with scripts.  Sometimes you may be lucky enough to have a "sysadmin" (systems administrator) who can assist you, but most of the time you'll find yourself needing to take care of business yourself.  

My suggestions for installing software are (in order):

## Sysadmin

Go introduce yourself to your sysadmins.  Take them to lunch or order them some pizza or drop off some good beer or whiskey.  Whatever it takes to be on good terms because a good sysadmin who is responsive to your needs is an enormous help.  If they are willing to install software for you, that is the way to go.  Often, though, this is a task far beneath them, and they would expect you to be able to fend for yourself.  They may provide "sudo" (https://xkcd.com/149/) privileges to allow you to install software into shared locations (e.g., "/usr/local"), but it's more likely they would expect you to install into your $HOME.

## Package managers

There are several package management systems for Linux and OSX including apt-get, yum, homebrew, macports, and more.  These usually relieve the problems of software compatibility and shared libraries.  Unless you have "sudo" to install globally, you can configure to install into your $HOME.

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

