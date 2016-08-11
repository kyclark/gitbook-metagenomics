# Minimally competent bash scripting

Bash is the worst shell scripting language except for all the others.  Much of the time, all you need is a simple bash script, so let's figure out how to write a decent one.  Though there are plenty of entire books you can read, I'll share with you what I've found to be the minimal amount I use.

# Comments

Every language has a way to indicate non-executable text in the source code.  Many Unix/c-style languages use the "#" (hash) sign to indicate that any text to the right should be ignored by the language, but some languages use other characters or character combinations.  Programmers may actually "comment" on their code to explain what some particularly hairy bit is doing, or they may use the characters to temporarily disable some section of code.  Here is what you might see:

```
# cf. https://en.wikipedia.org/wiki/Factorial
sub fac(n) { 
  # first check terminal condition
  if (n <= 1) {
    return 1
  }
  # no? let's recurse!
  else {
    n * fac(n - 1) # the number times one less the number
  }
}
```

It's worth investing time in an editor that can easily comment/uncomment whole sections of code.  For instance, in vim, I have the "#" sign mapped to a function that will add or removed the appropriate comment character(s) from the beginning of the selected section.  If your editor can't do that (e.g., nano), then I suggest you find something more powerful.

# Shebang

Scripting languages (sh, bash, Perl, Python, Ruby, etc.) are generally distinguished by the fact that the "program" is a regular file containing plain text that is interpreted into machine code at the time you run it.  Other languages (c, C++, Haskell) have a separate compilation step to turn their regular text source files into binary executable.  If you "less" a compiled file, you'll see a mess that might even lock up your window.  (If that happens, refer back to "Make it stop!" to kill it or just close the window and start over.)

So, basically a "script" is a plain text file that is often executable by virtue of having the executable bit(s) turned on (cf. "Permissions").  It does not have to be executable, however.  It's acceptable to put some commands in a file and simply tell the appropriate program to interpret the file as a series of commands:

```
$ echo "echo Hello, World" > hello.sh
$ sh hello.sh
Hello, World
```

But it looks cooler to do this:

```
$ chmod +x hello.sh
$ ./hello.sh
Hello, World
```

But what's going on here?

```
$ echo 'print("Hello, World")' > hello.py
$ chmod +x hello.py
$ ./hello.py
./hello.py: line 1: syntax error near unexpected token `"Hello, World"'
./hello.py: line 1: `print("Hello, World")'
```

We put some Python code in a file and then asked our shell (which is bash) to interpret it.  That didn't work.  If we ask Python to run it, everything is fine:

```
$ python hello.py
Hello, World
```

So we just need to let the shell know that this is Python code, and that is what the "shebang" (see "Pronuciations") line is for.  It looks like a comment, but it's special line that the shell uses to interpret the script.  I'll use an editor to add this to the "hello.py" script, then I'll ```cat``` the file so you can see what it looks like.  

```
$ cat hello.py
#!/usr/bin/env python
print("Hello, World")
$ ./hello.py
Hello, World
```

Often the shebang line will indicate the absolute path to a program like "/bin/bash" or "/usr/local/bin/gawk," but here I used an absolute path not to Python but to the "env" program which I then passed "python" as the argument.  Why did I do that?  To make this script "portable" (for certain values of "portable," cf. "It's easier to port a shell than a shell script." – Larry Wall), I prefer to use the "python" that is found by the environment.  There are two major versions of Python (2.x and 3.x), and I can't actually be sure which is the user's default or where they are installed.  In fact, just from my laptop to the two HPC systems I use, my "python" can be:

* /Users/kyclark/anaconda3/bin/python
* /work/03137/kyclark/local/bin/python
* /rsgrps/bhurwitz/hurwitzlab/bin/python

Go to your command line and type and type ```env python```, and you should be dropped into the Python REPL (Read-Evaluate-Print-Loop):

```
$ env python
Python 2.7.11 (default, Apr 22 2016, 18:12:30)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-16)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hi there.")
Hi there.
>>> exit
Use exit() or Ctrl-D (i.e. EOF) to exit
```

# Positional Arguments

You've now seen that a "script" is just a series of commands that are interpreted from top to bottom by a program like bash or Python.  You can automate your workflows simply by putting them into a text file, making the script executable, and running it; however, this would mean that everything is "hard-coded."  That is, if you wrote a script to clean and trim "mouse-reads.fastq" into "mouse-reads.fasta," then you need to edit the script when you get a new "fungi-reads.fastq."  Now you need to let some parts of your script be "variable" depending on the input from the user, and that's where "arguments" to your script come into play.

Here is a simple bash script that looks for a couple of arguments, "greeting" and "name":

```
$ cat -n positional01.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	GREETING=${1:-Hello}
     6	NAME=${2:-Stranger}
     7
     8	echo "$GREETING, $NAME"
$ ./positional01.sh
Hello, Stranger
$ ./positional01.sh Howdy
Howdy, Stranger
$ ./positional01.sh Howdy Ken
Howdy, Ken
```

I've used "cat -n" to number the lines so we can break this down.  Line 1 is our shebang to confirm that we have a bash script.  This is good because we can't be sure that others are using bash -- they might be using csh, tcsh, zsh, or even good olde sh (pronounced "shuh" or "ess-ach").  It's pretty much a given that any "shell" program is found in "/bin" where the most important commands are found.  As you get into other languages that can vary from system to system, you'll find those are usually installed into "/usr/local/bin," and so that's where I start to use "env" to find the program.

Line 3 tells bash to complain if I try to use a variable that was never initialized to some value.  This is putting on your helmet.  It's not a requirement (depending on where you live), but you absolutely should do this because there might come a day when you misspell a variable, and it's really a very Good Thing to have the shell catch this mistake.

Lines 5 and 6 I'm assigning two new variables to the first and second arguments to the script.  Variables in bash are plain words like ```GREETING``` when you assign to them but have sigils ("$") when you use them.  Also, assignment to a variable requires *no spaces*, so this wouldn't work:

```
GREETING = $1 # this will not work!
```

When making my variable assignments, I'm taking into account that maybe the user ran the program with no arguments, so I say "use $1 (the first argument) or this other thing (Hello)."  

If I want to *require* at least the greeting, then I can check the number of arguments with the ```$#``` variable:

```
$ cat -n positional02.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	if [[ $# -lt 1 ]]; then
     6	  printf "Usage: %s GREETING [NAME]\n" $(basename $0)
     7	  exit 1
     8	fi
     9
    10	GREETING=$1
    11	NAME=${2:-Stranger}
    12
    13	echo "$GREETING, $NAME"
$ ./positional02.sh "Good Day"
Good Day, Stranger
$ ./positional02.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

On line 5, I'm using a conditional wrapped in the double square brackets.  As you look at other's bash scripts, you'll often see the use of single brackets.  My limited understanding is that double brackets are safer, so I use them.  (Remember, this isn't called "minimally competent bash scripting" for nothing.)  The conditional is comparing ```$#``` to the integer "1" using the ```-lt``` (less than) operator.  There are plenty of web page that will describe all the conditions you can write in bash.  I won't belabor the point.  You can probably find plenty of other examples using common sense and a search engine.

Line 6 is using ```printf``` ("print format") instead of ```echo``` because I want to use the built-in function ```basename``` to extract just the filename of the currently running program ```$0```.  Without ```basename```, the "Usage" statement might have the full path to the program depending on where the user was when the program was launched, and that dog won't hunt:

```
$ ~/work/abe487/book/bash/positional02.sh
Usage: /Users/kyclark/work/abe487/book/bash/positional02.sh GREETING [NAME]
```

Notice that the call to ```basename``` was wrapped in ```$()``` that there are no commas separating the arguments.  You can also call functions using backticks (\`\`), but they are too visually similar to single quotes for my taste.

It's also important to note the subtle hints given to the user in the "Usage" statement.  "[NAME]" has square brackets to indicate that it is an option, but "GREETING" does not to say it is required.  

Line 7 says we need to ```exit``` the program using an exit value of "1" to indicate an error.  Unix programs use "0" to report "success" and anything else to report that something went wrong.  As you incorporate shell scripts into your pipelines, it will be important to distinguish to the operating system that a program finished with an error so that it can be propogated back to the user.  Just saying "exit" would use the default "0," so it might never be reported that the program exited because of bad input as opposed to just exiting because it finished.

Line 8 is the close of the "if" -- it's "if" spell backwards.  Isn't that clever?  Yes, we programming folk are very clever.  Guess what the close of a "case" statement in bash is?  It's "esac."  Guess what the close of a "do" statement is?  It's "done."  Oops.

The rest of the script is essentially the same, but I'd like to point out how it was called this time.  I wanted to use the GREETING "Good Day," so I had to put it in quotes so that the shell would not interpret them as two arguments.  Same with the NAME "Kind Sir."  

```
$ ./positional02.sh Good Day Kind Sir
Good, Day
```

Hmm, maybe we should detect that the script had too many arguments?

```
$ cat -n positional03.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	if [[ $# -lt 1 ]] || [[ $# -gt 2 ]]; then
     6	  printf "Usage: %s GREETING [NAME]\n" $0
     7	  exit 1
     8	fi
     9
    10	GREETING=$1
    11	NAME=${2:-Stranger}
    12
    13	printf "%s, %s\n" "$GREETING" "$NAME"
$ ./positional03.sh Good Day Kind Sir
Usage: ./positional03.sh GREETING [NAME]
$ ./positional03.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

To check for too many arguments, I added the double pipes ("||") and another conditional.  I also changed line 13 to use a ```printf``` command to highlight the importance of quoting the arguments *inside the script* so that bash won't get confused.  Try it without those quotes and try to figure out why it's doing what it's doing.  I highly recommend using the program "shellcheck" (https://github.com/koalaman/shellcheck) to find mistakes like this.  Also, consider using more powerful/helpful/sane languages -- but that's for another discussion.

# Named arguments

It's great that we can make our script take arguments, so of which are required, but it gets to be pretty icky once we go beyond approximately two arguments.  After that, we really need to have named arguments and/or flags to indicate how we want to run the program.  A named argument might be "-f mouse.fa" to indicate the value for the "-f" (probably "file") argument is "mouse.fa."  A flag like "-v" might be a yes/no ("Boolean," if you like) indicator that we want or do not want "verbose" mode.  You've encountered these with programs like ```ls -l``` to indicate you want the "long" directory listing or ```ps -u $USER``` to indicate the value for "-u" is the $USER.

Here is a version that has named arguments:

```
$ cat -n named01.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	GREETING=""
     6	NAME="Stranger"
     7
     8	function HELP() {
     9	  printf "Usage:\n  %s -g GREETING [-n NAME]\n\n" $(basename $0)
    10	  echo "Required arguments:"
    11	  echo " -g GREETING"
    12	  echo
    13	  echo "Options:"
    14	  echo " -n NAME ($NAME)"
    15	  echo
    16	  exit ${1:-0}
    17	}
    18
    19	if [[ $# -eq 0 ]]; then
    20	  HELP 1
    21	fi
    22
    23	while getopts :g:n:h OPT; do
    24	  case $OPT in
    25	    h)
    26	      HELP
    27	      ;;
    28	    g)
    29	      GREETING="$OPTARG"
    30	      ;;
    31	    n)
    32	      NAME="$OPTARG"
    33	      ;;
    34	    :)
    35	      echo "Error: Option -$OPTARG requires an argument."
    36	      exit 1
    37	      ;;
    38	    \?)
    39	      echo "Error: Invalid option: -${OPTARG:-""}"
    40	      exit 1
    41	  esac
    42	done
    43
    44	if [[ ${#GREETING} -lt 1 ]]; then
    45	  HELP 1
    46	fi
    47
    48	echo "$GREETING, $NAME"
```

Our s