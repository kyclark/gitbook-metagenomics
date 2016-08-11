# First steps with Unix

I assume you are on a command line by now, so let's look at some commands.

* **whoami**: tells you your username
* **w**: shows who is currently on a system
* **man**: show the manual page for a command
* **echo**: say something
* **env**: print your environment
* **which/type**: tells you the location of a program
* **touch**: create an empty regular file
* **pwd**: print working directory, where you are right now
* **ls**: list files in current directory
* **cd**: change directory (with no arguments, cd $HOME)
* **cp**: copy a file (or "cp -r" to copy a directory)
* **mv**: move a file or diretory
* **mkdir**: create a directory
* **rmdir**: remove a directory
* **rm**: remove a file (or "rm -r" to remove a directory)
* **cat**: concatenate files (cf. http://porkmail.org/era/unix/award.html)
* **column**: arrange text into columns
* **sort**: sort text or numbers
* **uniq**: remove duplicates *from a sorted list*
* **sed**: stream editor for altering text
* **awk/gawk**: pattern scanning and processing language
* **grep**: global regular expression program (maybe?), cf. https://en.wikipedia.org/wiki/Regular_expression
* **history**: look at past commands, cf. CTRL-R for searching your history directly from the command line
* **head**: view the first few (10) lines of a file
* **tail**: view the last (10) lines of a file
* **top**: view the programs taking the most system resources (memory, I/O, time, CPUs, etc.), cf. "htop"
* **cut**: select columns from the output of a program
* **wc**: (character) word (and line) count
* **more/less**: pager programs that show you a "page" of text at a time; cf. https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less/81131
* **bc**: calculator
* **df**: report file system disk space usage; useful to find a place to land your data
* **du**: report disk usage; recommend "du -shc"; useful to identify large directories that need to be removed
* **ssh**: secure shell, like telnet only with encryption
* **scp**: secure copy a file from/to remote systems using ssh
* **rsync**: remote sync; uses scp but only copies data that is different
* **ftp**: use "file transfer protocol" to retrieve large data sets (better than HTTP/browsers, esp for getting data onto remote systems)
* **|**: pipe the output of a command into another command
* **>, >>**: redirect the output of a command into a file; the file will be created if it does not exist; the single arrow indicates that you wish to overwrite any existing file, while the double-arrows say to append to an existing file
* **<**: redirect contents of a file into a command
* **wget**: web get a file from an HTTP location, cf. "wget is not a crime" and Aaron Schwartz
* **ftp**: original FTP client, very clunky
* **ncftp**: more modern FTP client that automatically handles anonymous logins
* **nano**: a very simple text editor; until you're ready to commit to vim or emacs, start here
* **md5sum**: calculate the MD5 checksum of a file
* **diff**: find the differences between two files

# Pronunciation

* **/**: slash; the thing leaing the other way is a "backslash"
* **etc**: et-see
* **usr**: user
* **$**: dollar
* **!**: bang
* **#!**: shebang
* **~**: twiddle or tilde; shortcut to your home directory when alone, shortcut to another user's home directory when used like "~bhurwitz"

# Handy command line shortcuts

* Tab: hit the Tab key for command completion
* **!!**: execute the last command again
* **!$**: the last argument from your previous command line (think of the $ as the right anchor in a regex)
* CTRL-R: reverse search of your history
* Up/down cursor keys: go backwards/forwards in your history
* CTRL-A, CTRL-E: jump to the start, end of the command line when in emacs mode (default)

> Protip: If you are on a Mac, it's easy to remap your (useless) CAPSLOCK key to CTRL.  Much less strain on your hand as you will find you need CTRL quite a bit, even more so if you choose emacs for your $EDITOR.

# Altering your environment

As you see above, "env" will list all the key-value pairs defining your environment.  For instance, everyone has a $HOME directory that you can see with ```echo $HOME```.  The exact location of $HOME can vary among systems, e.g.:

* Mac: /Users/kyclark
* Ocelot: /home/u20/kyclark
* Stampede: /home1/03137/kyclark

Your $PATH setting is extremely important as it defines the directory locations that will be searched (in order) to find programs.  Here's my $PATH on the UA HPC:

```
$ echo $PATH
/rsgrps/bhurwitz/hurwitzlab/bin:/sbin:/bin:/usr/bin:/usr/sbin:/usr/lpp/mmfs/bin:/usr/pbs/bin:/var/spool/pas/repository/pas-appmaker:/opt/sgi/sbin:/opt/sgi/bin:/usr/local/bin:/home/u20/kyclark/bin:/home/u20/kyclark/perl5/bin:/rsgrps/bhurwitz/hurwitzlab/tools/bpipe-0.9.9/bin:/home/u20/kyclark/.local/bin:/home/u20/kyclark/.rakudobrew/bin:/home/u20/kyclark/bin
```

You'll notice the diretories are separated by colons, and I have the shared "hurwitzlab" directory first in my path.  Much of our work will require access to tools that are not installed by default on the HPC.  You could build them into your own $HOME directory, but it will be easier if you just add this shared diretory to your $PATH.  From the command line, you can do this:

```
PATH=/rsgrps/bhurwitz/hurwitzlab/bin:$PATH
```

You just told your shell (bash) to set the PATH variable to our "hurwitzlab" diretory and then whatever it was set to before.  Obviously we want this to happen each time we log in, so we can add this command to "$HOME/.bashrc":

```
echo "PATH=/rsgrps/bhurwitz/hurwitzlab/bin:$PATH" >> ~/.bashrc
```

> Side note: Dotfiles (https://en.wikipedia.org/wiki/Hidden_file_and_hidden_directory) are files with names that begin with a dot.  They are normally hidden from view unless you use "ls -a" to list "all" files.  A single dot "." means the current directory, and two dots ".." mean the parent directory.


Your ".bashrc" (or maybe ".profile" or maybe ".bash_profile" depending on your system) file is read every time you login to your system, so you can remember your customizations.

# Aliases

Sometimes you'll find you're using a particular command quite often and want to create a shortcut.  You can assign any command to a single "alias" like so:

```
alias cx='chmod +x'
alias up2='cd ../../'
alias up3='cd ../../../'
```

If you execute this on the command line, the alias will be saved until you log out.  Adding these lines to your .bashrc will make it available every time you log in.  When you make a change and want the shell to bring those into the current environment, you need to "source" the file.  The command "." is an alias for "source":

```
$ source ~/.bashrc
$ . ~/.bashrc
```

# File system layout

The top level of a Unix file system is "/" which is called "root."  Confusingly, there is also an account named "root" which is basically the super-user/sysadmin (systems administrator).  Unix has always been a multi-tenant system ...

