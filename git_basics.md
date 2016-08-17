# Git basics

Linus Torvalds (https://en.wikipedia.org/wiki/Linus_Torvalds) is well known for creating the Linux operating system.  Managing all the source code for Linux became onerous with existing systems, so he created "git" to do that, too.  We'll learn enough git to get some things done.

# Github

Github is a for-profit business that provides git hosting services.  They offer free accounts for individuals and groups.  When you have completed an assignment, you will "push" it to Github.  When assignments are due, I will "pull" your code down to check it.  When you are done with class, you will have a public repository of code you can point to as evidence of your skills in both coding and source code management.  So, the first step is that you create a Github account.  Go do that.

# Git commands

A binary called "git" should be installed on any HPC or Unix system you have.  Git takes as its first argument a command.  To see a list, enter ```git``` or ```git help```.  To see more information on a particular command such as "add," use ```git help add```.

# Create a repo

Using the Github web interface, create a new repository called "abe487."  It's best to have the repo initialized with a "README."  Copy the "Clone or Download" link and then on your machine do:

```
git clone <your repo>
git clone git@github.com:kyclark/abe487.git kyclark
```

Now you can ```cp -r kyclark/problems <your repo>```.

# Committing your work

There are three git commands you must use to put your files into Github so that they can be seen by others:

* add
* commit
* push

A basic workflow is:

```
$ echo hello > hello.txt
$ git add hello.txt
$ git commit -m 'added hello' !$
$ git push
```

The ```-m``` argument to ```commit``` is the commit message.  If you don't specify a message, you will be dropped into your $EDITOR to type one.  
