# First steps with Unix

I assume you are on a command line by now, so let's look at some commands.

* **whoami**: tells you your username
* **w**: shows who is currently on a system
* **man**: show the manual page for a command
* **echo**: say something
* **env**: print your environment
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
* **>**: redirecty the output of a command into a file
* **<**: redirect contents of a file into a command

# File system layout

The top level of a Unix file system is "/" which is called "root."  Confusingly, there is also an account named "root" which is basically the super-user/sysadmin (systems administrator).  Unix has always been a multi-tenant system

# Pronunciation

* **/**: slash; the thing leaing the other way is a "backslash"
* **etc**: et-see
* **usr**: user
* **$**: dollar
* **!**: bang
* **#!**: shebang

# Exercises

In all the following, when you see a "$" it is a metacharacter indicating that this is a command-line prompt for a normal (not super-user) account.  Your prompt may be anything you like (search for "PS1 unix prompt" to learn more).  Anyway, point is that you should type (copy/paste) all the stuff *after* the $.  If you ever see a prompt with "#," it's indicating a command that should be run as the super-user/root account, e.g., installing some software into a system-wide directory so it can be shared by all users.

## Number of unique users

Find the number of unique users on a shared system

We know that "w" will tell us the users logged in.  Let's connect the output of "w" to "head" using a pipe, but we only want the first five lines:

```
$ w | head -5
 14:36:01 up 21 days, 21:51, 176 users,  load average: 3.83, 4.31, 4.47
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
antontre pts/1    149.165.156.129  Sat14    3days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    4:56m  0.15s  0.15s -bash
```

We need to skip the first two lines of header, so we can pipe the output of "w" into "awk" and tell it we only want to see output when the Number of Records is greater than 2:

```
$ w | awk 'NR>2' | head -5
antontre pts/1    149.165.156.129  Sat14    4days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    5:13m  0.15s  0.15s -bash
antontre pts/5    149.165.156.129  Sun19    2days  0.14s  0.14s /bin/sh -i
minyard  pts/8    129.114.64.18    29Jul16  4:24m  3:46m  3:46m top
antontre pts/11   149.165.156.129  Sun23    2days  0.24s  0.24s /bin/sh -i
```

We can see right away that the some users like "antontre" are logged in multiple times.  Let's "cut" out just the first column of data.  The manpage for "cut" says that it defaults to using the tab character to determine columns, so we'll need to tell it to use spaces:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1
antontre
huddack
antontre
minyard
antontre
```

Great, let's "uniq" that output:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | uniq
antontre
huddack
antontre
minyard
antontre
```

That's not right.  Remember I said earlier that "uniq" only works *on sorted input*?  So let's sort those names first:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | sort | uniq
antontre
huddack
minyard
```

Yes, that is correct.  Now let's remove the "head -5" and use "wc" to count all the lines (-l) of input:

```
$ w | awk 'NR>2' | cut -d ' ' -f 1 | sort | uniq | wc -l
138
```

# Count "oo" words

On almost every Unix system, you can find "/usr/share/dict/words."  Let's use "grep" to find how many have the "oo" vowel combination.  It's a long list, so I'll pipe it into "head" to see just the first five:

```
$ grep 'oo' /usr/share/dict/words | head -5
abloom
aboon
aboveproof
abrood
abrook
```

Yes, that works, so now let's count the words:

```
$ grep 'oo' /usr/share/dict/words | wc -l
10460
```

How many of those words additionally contain the "ow" sequence?

```
$ grep 'oo' /usr/share/dict/words | grep 'ow' | wc -l
158
```

How many *do not* contain the "ow" sequence?

```
$ grep 'oo' /usr/share/dict/words | grep -v 'ow' | wc -l
10302
```

Do those numbers add up?

```
$ bc <<< 158+10302
10460
```

Excellent.  Smithers, massage my brain.

