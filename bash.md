# Minimally competent bash scripting

Bash is the worst shell scripting language except for all the others.  For many of the analyses you'll write, all you will need is a simple bash script, so let's figure out how to write a decent one.  I'll share with you what I've found to be the minimal amount of bash I use.

# Statements

All programming language have a grammar where "statements" \(like "sentences"\) are built up from other terms.  Some languages like Python and Haskell use whitespace to figure out the end of a "statement," which is usually just the right side of the window.  C-like languages such as bash and Perl define the end of a statement with a colon `;`.  Bash is interesting because it uses both.  If you hit &lt;Enter&gt; or type a newline in your code, Bash will execute that statement.  If you want to put several commands on one line, you can separate each with a semicolon.  If you want to stretch a command over more than one line, you can use a backslash `\` to continue the line:

```
$ echo Hi
Hi
$ echo Hello
Hello
$ echo Hi; echo Hello
Hi
Hello
$ echo \
> Hi
Hi
```

# Comments

Every language has a way to indicate text in the source code that should not be executed by the program.  Many Unix/c-style languages use the `#` \(hash\) sign to indicate that any text to the right should be ignored by the language, but some languages use other characters or character combinations like `//` in Javascript, Java, and Rust.  Programmers may use comments to explain what some particularly bit of code is doing, or they may use the characters to temporarily disable some section of code.  Here is an example of what you might see:

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

It's worth investing time in an editor that can easily comment/uncomment whole sections of code.  For instance, in vim, I have a function that will add or removed the appropriate comment character\(s\) \(depending on the filetype\) from the beginning of the selected section.  If your editor can't do that \(e.g., nano\), then I suggest you find something more powerful.

# Shebang

Scripting languages \(sh, bash, Perl, Python, Ruby, etc.\) are generally distinguished by the fact that the "program" is a regular file containing plain text that is interpreted into machine code at the time you run it.  Other languages \(c, C++, Java, Haskell, Rust\) have a separate compilation step to turn their regular text source files into a binary executable.  If you view a compiled file with an editor/pager, you'll see a mess that might even lock up your window.  \(If that happens, refer back to "Make it stop!" to kill it or just close the window and start over.\)

So, basically a "script" is a plain text file that is often executable by virtue of having the executable bit\(s\) turned on \(cf. "Permissions"\).  It does not have to be executable, however.  It's acceptable to put some commands in a file and simply tell the appropriate program to interpret the file:

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

    $ echo 'print("Hello, World")' > hello.py
    $ chmod +x hello.py
    $ ./hello.py
    ./hello.py: line 1: syntax error near unexpected token `"Hello, World"'
    ./hello.py: line 1: `print("Hello, World")'

We put some Python code in a file and then asked our shell \(which is bash\) to interpret it.  That didn't work.  If we ask Python to run it, everything is fine:

```
$ python hello.py
Hello, World
```

So we just need to let the shell know that this is Python code, and that is what the "shebang" \(see "Pronunciations"\) line is for.  It looks like a comment, but it's special line that the shell uses to interpret the script.  I'll use an editor to add a shebang to the "hello.py" script, then I'll `cat` the file so you can see what it looks like.

```
$ cat hello.py
#!/usr/bin/env python
print("Hello, World")
$ ./hello.py
Hello, World
```

Often the shebang line will indicate the absolute path to a program like "/bin/bash" or "/usr/local/bin/gawk," but here I used an absolute path not to Python but to the "env" program which I then passed "python" as the argument.  Why did I do that?  To make this script "portable" \(for certain values of "portable," cf. "It's easier to port a shell than a shell script." – Larry Wall\), I prefer to use the "python" that is found by the environment.  There are two major versions of Python \(2.x and 3.x\), and I can't actually be sure which is the user's default or where they are installed.  In fact, just from my laptop to the two HPC systems I use, my "python" can be:

* /Users/kyclark/anaconda3/bin/python
* /work/03137/kyclark/local/bin/python
* /rsgrps/bhurwitz/hurwitzlab/bin/python

Go to your command line and type and type `env python`, and you should be dropped into the Python REPL \(Read-Evaluate-Print-Loop\):

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

# Let's Make A Script!

Let's make our script say "Hello" to someone:

```
$ cat -n hello2.sh
     1	#!/bin/bash
     2
     3	NAME="Newman"
     4	echo "Hello, $NAME!"
$ ./hello2.sh
Hello, Newman!
```

I've created a variable called `NAME` to hold the string "Newman."  Because all the variables from the environment \(see `env`\) are uppercase \(e.g., `$HOME` and `$USER`\), I tend to use all-caps myself, but this is not a requirement.  Just remember that everything in Unix is case-sensitive, so `$Name` is an entirely different variable from `$name`.  When assigning a variable, you can have NO SPACES and you never use the `$` in front of the variable:

```
$ NAME1="Doge"
$ echo "Such $NAME1"
Such Doge
$ NAME2 = "Doge"
-bash: NAME2: command not found
$ echo "Such $NAME2"
Such
```

We would like to get the NAME from the user rather than having it hardcoded in the script.  The arguments to a bash script are available through a few variables:

* `$#`: The number \(think "\#" == number\) of arguments
* `$@`: All the arguments in a single string
* `$0`: The name of the script
* `$1, $2`: The first argument, the second argument, etc.

A la:

```
$ cat -n args.sh
     1	#!/bin/bash
     2
     3	echo "Num of args    : \"$#\""
     4	echo "String of args : \"$@\""
     5	echo "Name of program: \"$0\""
     6	echo "First arg      : \"$1\""
     7	echo "Second arg     : \"$2\""
$ ./args.sh
Num of args    : "0"
String of args : ""
Name of program: "./args.sh"
First arg      : ""
Second arg     : ""
$ ./args.sh foo
Num of args    : "1"
String of args : "foo"
Name of program: "./args.sh"
First arg      : "foo"
Second arg     : ""
$ ./args.sh foo bar
Num of args    : "2"
String of args : "foo bar"
Name of program: "./args.sh"
First arg      : "foo"
Second arg     : "bar"
```

If you would like to iterate over all the arguments, you can use `$@` like so:

```
$ cat -n args2.sh
     1	#!/bin/bash
     2
     3	if [[ $# -lt 1 ]]; then
     4	    echo "There are no arguments"
     5	else
     6	    i=0
     7	    for ARG in "$@"; do
     8	        let i++
     9	        echo "$i: $ARG"
    10	    done
    11	fi
$ ./args2.sh
There are no arguments
$ ./args2.sh foo
1: foo
$ ./args2.sh foo bar "baz quux"
1: foo
2: bar
3: baz quux
```

Here I'm throwing in a conditional at line 3 to check if the script has any arguments.  If the number of arguments \(`$#`\) is less than \(`-lt`\) 1, then let the user know there is nothing to show; otherwise \(`else`\) do the next block of code.  The `for` loop on line 7 works by splitting the argument string \(`$@`\) on spaces just like the command line does.  Both `for` and `while` loops require the `do/done` pair to delineate the block of code \(some languages use `{}`, Haskell and Python use only indentation\).  Along those lines, line 11 is the close of the `if` -- "if" spell backwards; the close of a `case` statement in bash is `esac`.  

The other bit of magic I threw in was a counter variable \(which I always use lowercase `i` \["integer"\], `j` if I needed an inner-counter and so on\) which is initialized to "0" on line 6.  I increment it, I could have written `$i=$(($i + 1))`, but it's easier to use the `let i++` shorthand.  Lastly, notice that "baz quux" seen as a single argument because it was placed in quotes; otherwise arguments are separated by spaces.

Note that indentation doesn't matter as the program below works, but, honestly, which one is easier for you to read?

```
$ cat -n args3.sh
     1	#!/bin/bash
     2
     3	if [[ $# -lt 1 ]]; then
     4	echo "There are no arguments"
     5	else
     6	i=0
     7	for ARG in "$@"; do
     8	let i++
     9	echo "$i: $ARG"
    10	done
    11	fi
$ ./args3.sh foo bar
1: foo
2: bar
```

bash is a notoriously easy language to write incorrectly.  One step you can take to ensure you don't misspell variables is to add `set -u` at the top of your script.  E.g., if you type `echo $HOEM` on the command line, you'll get no output or warning that you misspelled the `$HOME` variable unless you `set -u`:

```
$ echo $HOEM

$ set -u
$ echo $HOEM
-bash: HOEM: unbound variable
```

This command tells bash to complain when you use a variable that was never initialized to some value.  This is like putting on your helmet.  It's not a requirement \(depending on which state you live in\), but you absolutely should do this because there might come a day when you misspell a variable.  Note that this will not save you from as error like this:

```
$ cat -n set-u-bug1.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    if [[ $# -gt 0 ]]; then
     6      echo $THIS_IS_A_BUG; # never initialized
     7    fi
     8
     9    echo "OK";
$ ./set-u-bug1.sh
OK
$ ./set-u-bug1.sh foo
./set-u-bug1.sh: line 6: THIS_IS_A_BUG: unbound variable
```

You can see that the first execution of the script ran just fine.  There is a bug on line 6, but bash didn't catch it because that line did not execute.  On the second run, the error occurred, and the script blew up.  \(FWIW, this is a problem in Python, too.\)

Here's another pernicious error:

```
$ cat -n set-u-bug2.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    GREETING="Hi"
     6    if [[ $# -gt 0 ]]; then
     7      GRETING=$1 # misspelled
     8    fi
     9
    10    echo $GREETING
$ ./set-u-bug2.sh
Hi
$ ./set-u-bug2.sh Hello
Hi
```

We were foolishly hoping that `set -u` would prevent us from misspelling the `$GREETING`, but at line 7 we simple created a new variable called `$GRETING`.  Perhaps you were hoping for more help from your language?  This is why we try to limit how much bash we write.

AT LAST, let's return to our "hello" script!

```
$ cat -n hello3.sh
     1	#!/bin/bash
     2
     3	echo "Hello, $1!"
$ ./hello3.sh Captain
Hello, Captain!
```

This should make perfect sense now.  We are simply saying "hello" to the first argument, but what happens if we provide no arguments?

```
$ ./hello3.sh
Hello, !
```

Well, that looks bad.  We should check that the script has the proper number of arguments which is 1:

```
$ cat -n hello4.sh
     1	#!/bin/bash
     2
     3	if [[ $# -ne 1 ]]; then
     4	    printf "Usage: %s NAME\n" "$(basename "$0")"
     5	    exit 1
     6	fi
     7
     8	echo "Hello, $1!"
$ ./hello4.sh
Usage: hello4.sh NAME
$ ./hello4.sh Captain
Hello, Captain!
$ ./hello4.sh Captain Picard
Usage: hello4.sh NAME
```

Line 3 checks if the number of arguments is not equal \(`-ne`\) to 1 and prints a help message to indicate proper "usage."  Importantly, it also will `exit` the program with a value which is not zero to indicate that there was an error.  \(NB: An exit value of "0" indicates 0 errors.\)  Line 4 uses `printf` rather than `echo` so I can do some fancy substitution so that the results of calling the `basename` function on the `$0` \(name of the program\) is inserted at the location of the `%s` \(a string value, cf. man pages for "printf" and "basename"\).

To call a function in bash, we can use either backticks \(under the `~` on a US keyboard\) or `$()`.  I find backticks to be too similar to single quotes, so I prefer the latter.  To demonstrate:

    $ ls | head
    args.sh*
    args2.sh*
    args3.sh*
    basic.sh*
    hello.sh*
    hello2.sh*
    hello3.sh*
    hello4.sh*
    hello5.sh*
    hello6.sh*
    $ FILES=`ls | head`
    $ echo $FILES
    args.sh args2.sh args3.sh basic.sh hello.sh hello2.sh hello3.sh hello4.sh hello5.sh hello6.sh

To continue, here is an alternate way to write this script:

```
$ cat -n hello5.sh
     1	#!/bin/bash
     2
     3	if [[ $# -eq 1 ]]; then
     4	    NAME=$1
     5	    echo "Hello, $NAME!"
     6	else
     7	    printf "Usage: %s NAME\n" "$(basename "$0")"
     8	    exit 1
     9	fi
```

Here I check on line 3 if there is just one argument, and the `else` is devoted to handling the error; however, I prefer to check for all possible errors at the beginning and `exit` the program quickly.  This also has the effect of keeping my code as far left on the page as possible.

Here is how you can provide a default value for an argument with `:-`:

```
$ cat -n hello6.sh
     1	#!/bin/bash
     2
     3	echo "Hello, ${1:-Stranger}!"
$ ./hello6.sh
Hello, Stranger!
$ ./hello6.sh Govnuh
Hello, Govnuh!
```

Now we're going to accept two arguments, "GREETING" and "NAME" while providing defaults for both:

```
$ cat -n positional.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	GREETING=${1:-Hello}
     6	NAME=${2:-Stranger}
     7
     8	echo "$GREETING, $NAME"
$ ./positional.sh
Hello, Stranger
$ ./positional.sh Howdy
Howdy, Stranger
$ ./positional.sh Howdy Padnuh
Howdy, Padnuh
```

What if I want to require at least one argument?

```
$ cat -n positional2.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	if [[ $# -lt 1 ]]; then
     6	    printf "Usage: %s GREETING [NAME]\n" "$(basename "$0")"
     7	    exit 1
     8	fi
     9
    10	GREETING=$1
    11	NAME=${2:-Stranger}
    12
    13	echo "$GREETING, $NAME"
$ ./positional2.sh "Good Day"
Good Day, Stranger
$ ./positional2.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

It's also important to note the subtle hints given to the user in the "Usage" statement.  `[NAME]` has square brackets to indicate that it is an option, but `GREETING` does not to say it is required.  As noted before I wanted to use the GREETING "Good Day," so I had to put it in quotes so that the shell would not interpret them as two arguments.  Same with the NAME "Kind Sir."

```
$ ./positional2.sh Good Day Kind Sir
Good, Day
```

Hmm, maybe we should detect that the script had too many arguments?

```
$ cat -n positional3.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	if [[ $# -lt 1 ]] || [[ $# -gt 2 ]]; then
     6	    printf "Usage: %s GREETING [NAME]\n" "$(basename "$0")"
     7	    exit 1
     8	fi
     9
    10	GREETING=$1
    11	NAME=${2:-Stranger}
    12
    13	printf "%s, %s\n" "$GREETING" "$NAME"
$ ./positional3.sh Good Day Kind Sir
Usage: positional3.sh GREETING [NAME]
$ ./positional3.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

To check for too many arguments, I added an "OR" \(the double pipes `||`\) and another conditional \("AND" is `&&`\).  I also changed line 13 to use a `printf` command to highlight the importance of quoting the arguments _inside the script_ so that bash won't get confused.  Try it without those quotes and try to figure out why it's doing what it's doing.  I highly recommend using the program "shellcheck" \([https://github.com/koalaman/shellcheck](https://github.com/koalaman/shellcheck)\) to find mistakes like this.  Also, consider using more powerful/helpful/sane languages -- but that's for another discussion.

# Named arguments

It's great that we can make our script take arguments, some of which are required, but it gets to be pretty icky once we go beyond approximately two arguments.  After that, we really need to have named arguments and/or flags to indicate how we want to run the program.  A named argument might be "-f mouse.fa" to indicate the value for the "-f" \("file," probably\) argument is "mouse.fa," whereas a flag like "-v" might be a yes/no \("Boolean," if you like\) indicator that we do or do not want "verbose" mode.  You've encountered these with programs like `ls -l` to indicate you want the "long" directory listing or `ps -u $USER` to indicate the value for "-u" is the $USER.

Here is a version that has named arguments:

```
$ cat -n named.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	GREETING=""
     6	NAME="Stranger"
     7
     8	function USAGE() {
     9	    printf "Usage:\n  %s -g GREETING [-n NAME]\n\n" $(basename $0)
    10	    echo "Required arguments:"
    11	    echo " -g GREETING"
    12	    echo
    13	    echo "Options:"
    14	    echo " -n NAME ($NAME)"
    15	    echo
    16	    exit ${1:-0}
    17	}
    18
    19	[[ $# -eq 0 ]] && USAGE 1
    20
    21	while getopts :g:n:h OPT; do
    22	  case $OPT in
    23	    h)
    24	      USAGE
    25	      ;;
    26	    g)
    27	      GREETING="$OPTARG"
    28	      ;;
    29	    n)
    30	      NAME="$OPTARG"
    31	      ;;
    32	    :)
    33	      echo "Error: Option -$OPTARG requires an argument."
    34	      exit 1
    35	      ;;
    36	    \?)
    37	      echo "Error: Invalid option: -${OPTARG:-""}"
    38	      exit 1
    39	  esac
    40	done
    41
    42	[[ -z "$GREETING" ]] && USAGE 1
    43
    44	echo "$GREETING, $NAME"
$ ./named.sh
Usage:
  named.sh -g GREETING [-n NAME]

Required arguments:
 -g GREETING

Options:
 -n NAME (Stranger)

$ ./named.sh -n Patch -g "Good Boy"
Good Boy, Patch
```

Our script just got much longer but also more flexible.  I've written a hundred shell scripts with just this as the template, so you can, too.  Go search for how `getopt` works and copy-paste this for your bash scripts, but the important thing to understand about `getopt` is that flags that take arguments have a `:` after them \(`g:` == "-g something"\) and ones that do not, well, do not \(`h` == "-h" == "please show me the help page\).

I've introduced a new function called `USAGE` that prints out the "Usage" statement so that it can be called when:

* the script is run with no arguments \(line 19\)
* the script is run with the "-h" flag \(lines 23-24\)
* the script is run with bad input \(line 44\)

I initialized the NAME to "Stranger" \(line 6\) and then let the user know in the "Usage" what the default value will be.  When checking the GREETING in line 44, I'm actually checking that the length of the value is greater than zero because it's possible to run the script like this:

```
$ ./named01.sh -g ""
```

Which would technically pass muster but does not actually meet our requirements.

# Loops

Often we want to do some set of actions for all the files in a directory or all the identifiers in a file:

```
$ for FILE in *.sh; do echo "FILE = $FILE"; done
FILE = args.sh
FILE = args2.sh
FILE = args3.sh
FILE = basic.sh
FILE = hello.sh
FILE = hello2.sh
FILE = hello3.sh
FILE = hello4.sh
FILE = hello5.sh
FILE = hello6.sh
FILE = named.sh
FILE = positional.sh
FILE = positional2.sh
FILE = positional3.sh
FILE = set-u-bug1.sh
FILE = set-u-bug2.sh
```

Here it is in a script:

```
$ cat -n for.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	DIR=${1:-$PWD}
     6
     7	if [[ ! -d "$DIR" ]]; then
     8	    echo "$DIR is not a directory"
     9	    exit 1
    10	fi
    11
    12	i=0
    13	for FILE in $DIR/*; do
    14	    let i++
    15	    printf "%3d: %s\n" $i "$FILE"
    16	done    
```

On line 5, I default `DIR` to the current working directory which I can find with the environmental variable `$PWD` \(print working directory\).  I check on line 7 that the argument is actually a directory with the `-d` test \(`man test`\).  The rest should look familiar.  Here it is in action:

```
$ ./for.sh
  1: /Users/kyclark/work/metagenomics-book/bash/args.sh
  2: /Users/kyclark/work/metagenomics-book/bash/args2.sh
  3: /Users/kyclark/work/metagenomics-book/bash/args3.sh
  4: /Users/kyclark/work/metagenomics-book/bash/basic.sh
  5: /Users/kyclark/work/metagenomics-book/bash/for.sh
  6: /Users/kyclark/work/metagenomics-book/bash/hello.sh
  7: /Users/kyclark/work/metagenomics-book/bash/hello2.sh
  8: /Users/kyclark/work/metagenomics-book/bash/hello3.sh
  9: /Users/kyclark/work/metagenomics-book/bash/hello4.sh
 10: /Users/kyclark/work/metagenomics-book/bash/hello5.sh
 11: /Users/kyclark/work/metagenomics-book/bash/hello6.sh
 12: /Users/kyclark/work/metagenomics-book/bash/named.sh
 13: /Users/kyclark/work/metagenomics-book/bash/positional.sh
 14: /Users/kyclark/work/metagenomics-book/bash/positional2.sh
 15: /Users/kyclark/work/metagenomics-book/bash/positional3.sh
 16: /Users/kyclark/work/metagenomics-book/bash/set-u-bug1.sh
 17: /Users/kyclark/work/metagenomics-book/bash/set-u-bug2.sh
 18: /Users/kyclark/work/metagenomics-book/bash/unclustered-proteins
$ ./for.sh ../problems
  1: ../problems/cat-n
  2: ../problems/common-words
  3: ../problems/dna
  4: ../problems/gapminder
  5: ../problems/gc
  6: ../problems/greeting
  7: ../problems/hamming
  8: ../problems/hello
  9: ../problems/proteins
 10: ../problems/tac
 11: ../problems/txt2fasta
 12: ../problems/wc
 13: ../problems/yeast
```

Often I want to iterate over the results of some calculation.  Here is an example of saving the results of an operation into a temporary file:

```
$ cat -n while.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	FILE="${1:-../problems/gc/anthrax.fa}"
     6	IDS=$(mktemp)
     7	grep '^>' "$FILE" | awk '{print $1}' | sed "s/^>//" > "$IDS"
     8	NUM=$(wc -l "$IDS" | awk '{print $1}')
     9
    10	if [[ $NUM -lt 1 ]]; then
    11	    echo "Found no ids in FILE \"$FILE\""
    12	    exit 1
    13	fi
    14
    15	i=0
    16	while read -r ID; do
    17	    let i++
    18	    printf "%3d: %s\n" $i "$ID"
    19	done < "$IDS"
    20
    21	rm "$IDS"
```

Line 6 uses the `mktemp` function to give us the name of a temporary file.  On line 7, I `grep` for the greater than sign at the beginning of a line \("^&gt;"\) in the given `$FILE`. The results of that are piped `|` into `awk` to give me just the first field \(as delimited by spaces\) which I then pipe into `sed` \(stream editor\) to substitute \(`s//`\) the "&gt;" at the beginning of the line for nothing.  The result of that pipeline is redirected \(`>`\) into the `$IDS` file.  Line 8 counts the lines of the file \(`wc -l`\) and gets the first field using `awk`.  Line 10 checks that we found some identifiers and bails if not.  Line 16 uses `while; do/done` to read a redirect in \(`<`\) from the `$IDS` file.   Line 21 removes the temporary file.  Here is how it works:

```
$ ./while.sh for.sh
Found no ids in FILE "for.sh"
$ ./while.sh
  1: ERR1596646.1
  2: ERR1596646.2
  3: ERR1596646.3
  4: ERR1596646.4
  5: ERR1596646.5
  6: ERR1596646.6
  7: ERR1596646.7
  8: ERR1596646.8
  9: ERR1596646.9
 10: ERR1596646.10
 11: ERR1596646.11
 12: ERR1596646.12
 13: ERR1596646.13
 14: ERR1596646.14
 15: ERR1596646.15
 16: ERR1596646.16
 17: ERR1596646.17
 18: ERR1596646.18
 19: ERR1596646.19
 20: ERR1596646.20
 21: ERR1596646.21
 22: ERR1596646.22
 23: ERR1596646.23
```

To understand the `read` better, see this:

```
$ cat -n read.sh
     1	#!/bin/bash
     2
     3	set -u
     4	echo -n "What is your name? "
     5	read NAME
     6	echo "Would you like to play a nice game of chess, $NAME?"
$ ./read.sh
What is your name? Joshua
Would you like to play a nice game of chess, Joshua?
```

So the `while` keeps succeeding as long as `read` can put a line of input into `ID`.  When it reaches the end of the file, it stops.

It's possible to write this with `cat`, too:

```
$ cat -n while2.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	FILE="${1:-../problems/gc/anthrax.fa}"
     6	IDS=$(mktemp)
     7	grep '^>' "$FILE" | awk '{print $1}' | sed "s/^>//" > "$IDS"
     8	NUM=$(wc -l "$IDS" | awk '{print $1}')
     9
    10	if [[ $NUM -lt 1 ]]; then
    11	    echo "Found no ids in FILE \"$FILE\""
    12	    exit 1
    13	fi
    14
    15	i=0
    16	for ID in $(cat "$IDS"); do
    17	    let i++
    18	    printf "%3d: %s\n" $i "$ID"
    19	done
    20
    21	rm "$IDS"
```

But `while` has another advantage in that it can split each line of input into separate variables:

```
$ cat -n while3.sh
     1	#!/bin/bash
     2
     3	set -u
     4
     5	FILE="${1:-../problems/gc/anthrax.fa}"
     6	TMP=$(mktemp)
     7	grep '^>' "$FILE" | sed "s/^>//" | awk '{print $1 " " $3}' | sed "s/length=//" > "$TMP"
     8	NUM=$(wc -l "$TMP" | awk '{print $1}')
     9
    10	if [[ $NUM -lt 1 ]]; then
    11	    echo "Found no ids in FILE \"$FILE\""
    12	    exit 1
    13	fi
    14
    15	while read -r ID LENGTH; do
    16	    printf "%3d: %s\n" "$LENGTH" "$ID"
    17	done < "$TMP"
    18
    19	rm "$TMP"
$ ./while3.sh ../problems/gc/burk.fa
300: SRR3943777.1
300: SRR3943777.2
300: SRR3943777.3
300: SRR3943777.4
300: SRR3943777.5
300: SRR3943777.6
300: SRR3943777.7
300: SRR3943777.8
300: SRR3943777.9
```

# A few more tricks

Lastly I'm going to show you how to create some sane defaults, make missing directories, find user input, transform that input, and report back to the user.  Here's a script that takes an IN\_DIR, counts the lines of all the files therein, and reports said line counts into an optional OUT\_DIR.

```
     1	#!/bin/bash
     2	
     3	set -u
     4	
     5	IN_DIR=""
     6	OUT_DIR="$PWD/$(basename "$0" '.sh')-out"
     7	
     8	function lc() {
     9	    wc -l "$1" | awk '{print $1}'
    10	}
    11	
    12	function USAGE() {
    13	    printf "Usage:\n  %s -i IN_DIR -o OUT_DIR\n\n" "$(basename "$0")"
    14	
    15	    echo "Required arguments:"
    16	    echo " -i IN_DIR"
    17	    echo "Options:"
    18	    echo " -o OUT_DIR"
    19	    echo 
    20	    exit "${1:-0}"
    21	}
    22	
    23	[[ $# -eq 0 ]] && USAGE 1
    24	
    25	while getopts :i:o:h OPT; do
    26	    case $OPT in
    27	        h)
    28	            USAGE
    29	            ;;
    30	        i)
    31	            IN_DIR="$OPTARG"
    32	            ;;
    33	        o)
    34	            OUT_DIR="$OPTARG"
    35	            ;;
    36	        :)
    37	            echo "Error: Option -$OPTARG requires an argument."
    38	            exit 1
    39	            ;;
    40	        \?)
    41	            echo "Error: Invalid option: -${OPTARG:-""}"
    42	            exit 1
    43	    esac
    44	done
    45	
    46	if [[ -z "$IN_DIR" ]]; then
    47	    echo "IN_DIR is required"
    48	    exit 1
    49	fi
    50	
    51	if [[ ! -d "$IN_DIR" ]]; then
    52	    echo "IN_DIR \"$IN_DIR\" is not a directory."
    53	    exit 1
    54	fi
    55	
    56	echo "Started $(date)"
    57	
    58	FILES_LIST=$(mktemp)
    59	find "$IN_DIR" -type f -name \*.sh > "$FILES_LIST"
    60	NUM_FILES=$(lc "$FILES_LIST")
    61	
    62	if [[ $NUM_FILES -gt 0 ]]; then
    63	    echo "Will process NUM_FILES \"$NUM_FILES\""
    64	
    65	    [[ ! -d $OUT_DIR ]] && mkdir -p "$OUT_DIR"
    66	
    67	    i=0
    68	    while read -r FILE; do
    69	        BASENAME=$(basename "$FILE")
    70	        let i++
    71	        printf "%3d: %s\n" $i "$BASENAME"
    72	        wc -l "$FILE" > "$OUT_DIR/$BASENAME"
    73	    done < "$FILES_LIST"
    74	
    75	    rm "$FILES_LIST"
    76	    echo "See results in OUT_DIR \"$OUT_DIR\""
    77	else
    78	    echo "No files found in \"$IN_DIR\""
    79	fi
    80	
    81	echo "Finished $(date)"

```

The IN\_DIR argument is required \(lines 46-49\), and it must be a directory \(lines 51-54\). If the user does not supply an OUT\_DIR, I will create a reasonable default using the current working directory and the name of the script plus "-out" \(line 6\).  One thing I love about bash is that I can call functions inside of strings, so OUT\_DIR is a string \(it's in double quotes\) of the variable $PWD, the character "/", and the result of the function call to `basename` where I'm giving the optional second argument ".sh" that I want removed from the first argument, and then the string "-out".

At line 58, I create a temporary file to hold the names of the files I need to process.  A line 59, I look for the files in IN\_DIR that need to be processed.  You can read the manpage for `find` and think about what your script might need to find \(".fa" files greater than 0 bytes in size last modified since some date, etc.\).  At line 60, I call my `lc` \(line count\) function to see how many files I found.  If I found more than 0 files \(line 62\), then I move ahead with processing.  I check to see if the OUT\_DIR needs to be created \(line 65\), and then create a counter variable \("i"\) that I'll use to number the files as I process them.  At line 68, I start a `while` loop to iterate over the input from redirecting _in_ from the temporary file holding the file names \(line 73, `< "$FILES_LIST"`\).  Then a `printf` to let the user know which file we're processing, then a simple command \(`wc`\) but where you might choose to BLAST the sequence file to a database of pathogens to determine how deadly the sample is.  When I'm done, I clean up the temp file \(line 75\).

The alternate path when I find no input files \(line 77-79\) is to report that fact.  Bracketing the main processing logic are "Started/Finished" statements so I can see how long my script took.  When you start your coding career, you will usually sit and watch your code run before you, but eventually you'll submit the your jobs to an HPC queue where they will be run for you on a separate machine when the resources become available.

The above is, I would say, a minimally competent bash script.  If you can understand everything in there, then you know enough to be dangerous and should move on to learning more powerful languages -- like Python!

# Exercises

Write a script that mimics "cat -n" for a given file.

