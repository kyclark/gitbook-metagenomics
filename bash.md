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

# set -u

If you type `echo $HOEM` on the command line, you'll get no output or warning that you misspelled the `$HOME` variable unless you `set -u`:

```
$ echo $HOEM

$ set -u
$ echo $HOEM
-bash: HOEM: unbound variable
```

This command tells bash to complain when you use a variable that was never initialized to some value.  This is like putting on your helmet.  It's not a requirement \(depending on which state you live in\), but you absolutely should do this because there might come a day when you misspell a variable.  One serious problem is that you can still get errors like this:

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

# Positional Arguments

You've now seen that a "script" is just a series of commands that are interpreted from top to bottom by a program like bash or Python.  You can automate your workflows simply by putting them into a text file, making the script executable, and running it; however, this would mean that everything is "hard-coded."  That is, if you wrote a script to clean and trim "mouse-reads.fastq" into "mouse-reads.fasta," then you need to edit the script when you get a new "fungi-reads.fastq."  You need to let some parts of your script be "variable" depending on the input from the user, and that's where "arguments" to your script come into play.

Here is a simple bash script that looks for a couple of arguments, "greeting" and "name":

```
$ cat -n positional01.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    GREETING=${1:-Hello}
     6    NAME=${2:-Stranger}
     7
     8    echo "$GREETING, $NAME"
$ ./positional01.sh
Hello, Stranger
$ ./positional01.sh Howdy
Howdy, Stranger
$ ./positional01.sh Howdy Ken
Howdy, Ken
```

I've used "cat -n" to number the lines so we can break this down.  Line 1 is our shebang to confirm that we have a bash script.  This is good because we can't be predict the user's shell -- they might be using csh, tcsh, zsh, bash, or even good olde sh \(pronounced "shuh" or "ess-ach"\).  It's pretty much a given that any "shell" program is found in "/bin" where the most important commands are found.  As you get into other languages that can vary from system to system, you'll find those are _often_ \(but not always\) installed into "/usr/local/bin," and so that's where I start to use "env" to find the program.

In lines 5 and 6, I'm assigning two new variables to the first and second arguments to the script.  Variables in bash are plain words like `GREETING` when you assign to them but have sigils \("$"\) when you use them \(line 8\).  Also, assignment to a variable requires _no spaces_, so this wouldn't work:

```
GREETING = $1 # this will not work!
```

When making my variable assignments, I'm taking into account that maybe the user ran the program with no arguments, so I say "use $1 \(the first argument\) or this other thing \(Hello\)."

If I want to _require_ at least the greeting, then I can check the number of arguments with the `$#` variable:

```
$ cat -n positional02.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    if [[ $# -lt 1 ]]; then
     6      printf "Usage: %s GREETING [NAME]\n" $(basename $0)
     7      exit 1
     8    fi
     9
    10    GREETING=$1
    11    NAME=${2:-Stranger}
    12
    13    echo "$GREETING, $NAME"
$ ./positional02.sh "Good Day"
Good Day, Stranger
$ ./positional02.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

On line 5, I'm using a conditional wrapped in the double square brackets.  As you look at bash scripts written by other \(lesser\) programmers, you'll may see the use of single brackets.  My limited understanding is that double brackets are safer, so I use them.  \(Remember, this isn't called "minimally competent bash scripting" for nothing.\)  The conditional is comparing `$#` \(the number of arguments to the script\) to the integer "1" using the `-lt` \(less than\) operator.  \(All of the arguments are contained in the variable `$@`.\)  There are plenty of web pages that will describe all the conditions you can write in bash -- I won't belabor the point.  You can find plenty of other examples using common sense and a search engine.

Line 6 is using `printf` \("print format"\) instead of `echo` because I want to use the built-in function `basename` to extract just the filename of the currently running program `$0`.

> Protip: Perl 5 and Python also think of $0 as the name of the script.  In Perl it's actually the variable `$0` as in bash, but in Python is `argv[0]` -- the zeroth argument to the script.

If I didn't call `basename`, the "Usage" statement might have the full path to the program depending on where the user was when the program was launched, and that dog won't hunt:

```
$ ~/work/abe487/book/bash/positional02.sh
Usage: /Users/kyclark/work/abe487/book/bash/positional02.sh GREETING [NAME]
```

Notice that the call to `basename` was wrapped in `$()` and that there are no commas separating the arguments -- just like on the command line.  \(That is, you don't say "cp foo, bar".\)  You can also call functions using backticks \(\`\`\), but they are too visually similar to single quotes for my taste.

It's also important to note the subtle hints given to the user in the "Usage" statement.  "\[NAME\]" has square brackets to indicate that it is an option, but "GREETING" does not to say it is required.

Line 7 says we need to `exit` the program using an exit value of "1" to indicate an error.  Unix programs use "0" to report "success" and anything else to report that something went wrong.  As you incorporate shell scripts into your pipelines, it will be important to distinguish to the operating system that a program finished with an error so that it can be propogated back to the user.  Just saying "exit" would use the default "0," so it might never be reported that the program exited because of bad input as opposed to just exiting because it finished.

Line 8 is the close of the "if" -- it's "if" spell backwards.  Isn't that clever?  Yes, we programming folk are very clever.  Guess what the close of a "case" statement in bash is?  It's "esac."  Guess what the close of a "do" statement is?  It's "done."  D'oh!

The rest of the script is essentially the same, but I'd like to point out how it was called this time.  I wanted to use the GREETING "Good Day," so I had to put it in quotes so that the shell would not interpret them as two arguments.  Same with the NAME "Kind Sir."

```
$ ./positional02.sh Good Day Kind Sir
Good, Day
```

Hmm, maybe we should detect that the script had too many arguments?

```
$ cat -n positional03.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    if [[ $# -lt 1 ]] || [[ $# -gt 2 ]]; then
     6      printf "Usage: %s GREETING [NAME]\n" $0
     7      exit 1
     8    fi
     9
    10    GREETING=$1
    11    NAME=${2:-Stranger}
    12
    13    printf "%s, %s\n" "$GREETING" "$NAME"
$ ./positional03.sh Good Day Kind Sir
Usage: ./positional03.sh GREETING [NAME]
$ ./positional03.sh "Good Day" "Kind Sir"
Good Day, Kind Sir
```

To check for too many arguments, I added an "OR" \(the double pipes `||`\) and another conditional \("AND" is `&&`\).  I also changed line 13 to use a `printf` command to highlight the importance of quoting the arguments _inside the script_ so that bash won't get confused.  Try it without those quotes and try to figure out why it's doing what it's doing.  I highly recommend using the program "shellcheck" \([https://github.com/koalaman/shellcheck](https://github.com/koalaman/shellcheck)\) to find mistakes like this.  Also, consider using more powerful/helpful/sane languages -- but that's for another discussion.

# Named arguments

It's great that we can make our script take arguments, some of which are required, but it gets to be pretty icky once we go beyond approximately two arguments.  After that, we really need to have named arguments and/or flags to indicate how we want to run the program.  A named argument might be "-f mouse.fa" to indicate the value for the "-f" \("file," probably\) argument is "mouse.fa," whereas a flag like "-v" might be a yes/no \("Boolean," if you like\) indicator that we do or do not want "verbose" mode.  You've encountered these with programs like `ls -l` to indicate you want the "long" directory listing or `ps -u $USER` to indicate the value for "-u" is the $USER.

Here is a version that has named arguments:

```
$ cat -n named01.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    GREETING=""
     6    NAME="Stranger"
     7
     8    function USAGE() {
     9      printf "Usage:\n  %s -g GREETING [-n NAME]\n\n" $(basename $0)
    10      echo "Required arguments:"
    11      echo " -g GREETING"
    12      echo
    13      echo "Options:"
    14      echo " -n NAME ($NAME)"
    15      echo
    16      exit ${1:-0}
    17    }
    18
    19    if [[ $# -eq 0 ]]; then
    20      USAGE 1
    21    fi
    22
    23    while getopts :g:n:h OPT; do
    24      case $OPT in
    25        h)
    26          USAGE
    27          ;;
    28        g)
    29          GREETING="$OPTARG"
    30          ;;
    31        n)
    32          NAME="$OPTARG"
    33          ;;
    34        :)
    35          echo "Error: Option -$OPTARG requires an argument."
    36          exit 1
    37          ;;
    38        \?)
    39          echo "Error: Invalid option: -${OPTARG:-""}"
    40          exit 1
    41      esac
    42    done
    43
    44    if [[ ${#GREETING} -lt 1 ]]; then
    45      USAGE 1
    46    fi
    47
    48    echo "$GREETING, $NAME"
```

Our script just got much longer but also more flexible.  I've written a hundred shell scripts with just this as the template, so you can, too.  Go search for how `getopt` works and copy-paste this for your bash scripts.

I've introduced a new function called `USAGE` that prints out the "Usage" statement so that it can be called when:

* the script is run with no arguments \(line 19\)
* the script is run with the "-h" flag \(lines 25-27\)
* the script is run with bad input \(lines 44-46\)

I initialized the NAME to "Stranger" \(line 6\) and then let the user know in the "Usage" what the default value will be.  When checking the GREETING in line 44, I'm actually checking that the length of the value is greater than zero because it's possible to run the script like this:

```
$ ./named01.sh -g ""
```

Which would technically pass muster but does not actually meet our requirements.

# A few more tricks

Lastly I'm going to show you how to create some sane defaults, make missing directories, find user input, transform that input, and report back to the user.  Here's a script that takes an IN\_DIR, counts the lines of all the files therein, and reports said line counts into an optional OUT\_DIR.

```
$ cat -n basic.sh
     1    #!/bin/bash
     2
     3    set -u
     4
     5    IN_DIR=""
     6    OUT_DIR="$PWD/$(basename $0 '.sh')-out"
     7
     8    function lc() {
     9      wc -l "$1" | awk '{print $1}'
    10    }
    11
    12    function USAGE() {
    13      printf "Usage:\n  %s -i IN_DIR -o OUT_DIR\n\n" $(basename $0)
    14
    15      echo "Required arguments:"
    16      echo " -i IN_DIR"
    17      echo "Options:"
    18      echo " -o OUT_DIR"
    19      echo
    20      exit ${1:-0}
    21    }
    22
    23    if [[ $# -eq 0 ]]; then
    24      USAGE 1
    25    fi
    26
    27    while getopts :i:o:h OPT; do
    28      case $OPT in
    29        h)
    30          USAGE
    31          ;;
    32        i)
    33          IN_DIR="$OPTARG"
    34          ;;
    35        o)
    36          OUT_DIR="$OPTARG"
    37          ;;
    38        :)
    39          echo "Error: Option -$OPTARG requires an argument."
    40          exit 1
    41          ;;
    42        \?)
    43          echo "Error: Invalid option: -${OPTARG:-""}"
    44          exit 1
    45      esac
    46    done
    47
    48    if [[ ${#IN_DIR} -lt 1 ]]; then
    49      echo "IN_DIR is required"
    50      exit 1
    51    fi
    52
    53    if [[ ! -d $IN_DIR ]]; then
    54      echo "IN_DIR \"$IN_DIR\" is not a directory."
    55      exit 1
    56    fi
    57
    58    echo "Started $(date)"
    59
    60    FILES_LIST=$(mktemp)
    61    find "$IN_DIR" -type f -name \*.sh > "$FILES_LIST"
    62    NUM_FILES=$(lc $FILES_LIST)
    63
    64    if [[ $NUM_FILES -gt 0 ]]; then
    65      echo "Will process NUM_FILES \"$NUM_FILES\""
    66
    67      if [[ ! -d $OUT_DIR ]]; then
    68        mkdir -p "$OUT_DIR"
    69      fi
    70
    71      i=0
    72      while read FILE; do
    73        BASENAME=$(basename $FILE)
    74        let i++
    75        printf "%3d: %s\n" $i $BASENAME
    76        wc -l $FILE > $OUT_DIR/$BASENAME
    77      done < $FILES_LIST
    78
    79      rm $FILES_LIST
    80      echo "See results in OUT_DIR \"$OUT_DIR\""
    81    else
    82      echo "No files found in \"$IN_DIR\""
    83    fi
    84
    85    echo "Finished $(date)"
```

The IN\_DIR argument is required \(lines 48-51\), and it must be a directory \(lines 53-56\). If the user does not supply an OUT\_DIR, I will create a reasonable default \(line 6\).  One thing I love about bash is that I can call functions inside of strings, so OUT\_DIR is a string \(it's in double quotes\) of the variable $PWD, the character "/", and the result of the function call to `basename` where I'm giving the optional second argument ".sh" that I want removed from the first argument, and then the string "-out".

At line 60, I create a temporary file to hold the names of the files I need to process.  A line 61, I look for the files in IN\_DIR that need to be processed.  You can read the manpage for `find` and think about what your scirpt might need to find \(".fa" files greater than 0 bytes in size last modified since some date, etc.\).  At line 62, I call my `lc` \(line count\) function to see how many files I found.  If I found more than 0 files \(line 64\), then I move ahead with processing.  I check to see if the OUT\_DIR needs to be created \(lines 67-69\), and then create a counter variable \("i"\) that I'll use to number the files as I process them.  At line 72, I start a `while` loop to iterate over the input from redirecting _in_ from the temporary file holding the file names \(line 77, `< $FILES_LIST`\).  Line 74 is a "let" expression where I increment the "i" variable by one \(`i++`\).  Then a `printf` to let the user know which file we're processing, then a simple command \(`wc`\) but where you might choose to BLAST the sequence file to a database of pathogens to determine how deadly the sample is.  When I'm done, I clean up the temp file \(line 79\).

The alternate path when I find no input files \(line 81-83\) is to report that fact.  Bracketing the main processing logic are "Started/Finished" statements so I can see how long my script took.  When you start your coding career, you will usually sit and watch your code run before you, but eventually you'll submit the your jobs to an HPC queue where they will be run for you on a separate machine when the resources become available.

The above is, I would say, a minimally competent bash script.  If you can understand everything in there, then you know enough to be dangerous and should move on to learning more powerful languages -- like Perl!

# Exercises

Write a script that mimics "cat -n" for a given file.

