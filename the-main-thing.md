If you're coming to Perl 6 from Perl 5, the global variable &lt;a href="https://docs.perl6.org/language/variables\#index-entry-%40%2AARGS"&gt;@\*ARGS&lt;/a&gt; will be familiar to you as the place to get the command-line arguments to your program:



```
$ cat main1.pl6
#!/usr/bin/env perl6
```



put "ARGS = ", @\*ARGS.join\(', '\);

$ ./main1.pl6 foo bar baz

ARGS = foo, bar, baz

\`\`\`



The @ is the sigil that denotes the variable as an array, and the &lt;code&gt;\*&lt;/code&gt; is the "twigle" that denotes that the variable is a global.  If you follow the above link to the documentation, you'll find a whole host of other dynamic and environmental variables like user, hostname, PID, cwd, etc.



Dealing directly with @\*ARGS is fine your program accepts a few positional arguments where the first argument means one thing \(name\), the second argument another thing \(rank\), the third another \(serial number\), etc:



\`\`\`

$ cat main2.pl6

\#!/usr/bin/env perl6



my \($name, $rank, $serial-num\) = @\*ARGS;

put "name \($name\) rank \($rank\) serial number \($serial-num\)";

$ ./main2.pl6 Patch Private 1656401

name \(Patch\) rank \(Private\) serial number \(1656401\)

\`\`\`



Or if all the arguments are homogenous \(e.g., a list of files to process\), this works reasonably well:



\`\`\`

$ cat main3.pl6

\#!/usr/bin/env perl6



for @\*ARGS -&gt; $file {

    put "Processing '$file'";

}



put "Done."

$ ./main3.pl6 foo bar baz

Processing 'foo'

Processing 'bar'

Processing 'baz'

Done.

\`\`\`



If we declare a &lt;a href="https://docs.perl6.org/language/functions\#index-entry-MAIN"&gt;MAIN&lt;/a&gt; subroutine, we can automatically assign the variables instead of taking them from @\*ARGS:



\`\`\`

sub MAIN \($name, $rank, $serial-num\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

\`\`\`



Perl will enforce the number of arguments and generate help when supplied the wrong number or when requested:



\`\`\`

$ ./main4.pl6 Patch Private

Usage:

    ./main4.pl6 &lt;name&gt; &lt;rank&gt; &lt;serial-num&gt;

$ ./main4.pl6 Patch Private 1656401

name \(Patch\) rank \(Private\) serial number \(1656401\)

$ ./main4.pl6 -h

Usage:

  ./main4.pl6 &lt;name&gt; &lt;rank&gt; &lt;serial-num&gt;

\`\`\`



You can also add &lt;a href="https://docs.perl6.org/type.html"&gt;type&lt;/a&gt; constraints:



\`\`\`

$ cat main5.pl6

\#!/usr/bin/env perl6



sub MAIN \(Str $name, Str $rank, Int $serial-num\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

$ ./main5.pl6 foo bar baz

Usage:

  ./main5.pl6 &lt;name&gt; &lt;rank&gt; &lt;serial-num&gt;

$ ./main5.pl6 foo bar 123

name \(foo\) rank \(bar\) serial number \(123\)

\`\`\`



To make them named arguments, simply prefix with a : and then you can supply the arguments in any order:



\`\`\`

$ cat main6.pl6

\#!/usr/bin/env perl6



sub MAIN \(Str :$name, Str :$rank, Int :$serial-num\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

$ ./main6.pl6 -h

Usage:

  ./main6.pl6 \[--name=&lt;Str&gt;\] \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

$ ./main6.pl6 --name=Patch --serial-num=1656401 --rank=Private

name \(Patch\) rank \(Private\) serial number \(1656401\)

\`\`\`



There's a problem if I don't supply any arguments to this last version:



\`\`\`

$ ./main6.pl6

Use of uninitialized value of type Str in string context.

Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.

  in sub MAIN at ./main6.pl6 line 3

Use of uninitialized value of type Str in string context.

Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.

  in sub MAIN at ./main6.pl6 line 3

Use of uninitialized value of type Int in string context.

Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.

  in sub MAIN at ./main6.pl6 line 3

name \(\) rank \(\) serial number \(\)

\`\`\`



Perl is complaining that it can't print uninitialized values. I didn't declare that any of the arguments are required -- which I can do by post-fixing the variables with ! -- nor did I provide defaults for those that are not -- which I can do with &lt;code&gt;=&lt;/code&gt;:



\`\`\`

$ cat main7.pl6

\#!/usr/bin/env perl6



sub MAIN \(Str :$name!, Str :$rank='NA', Int :$serial-num=0\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

$ ./main7.pl6

Usage:

  ./main7.pl6 --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

\`\`\`



The absence of \[\] around the &lt;code&gt;--name&lt;/code&gt; indicates to the user that the argument is required, while &lt;code&gt;--rank&lt;/code&gt; and &lt;code&gt;--serial-num&lt;/code&gt; are optional:



\`\`\`

$ ./main7.pl6 --name=Patch

name \(Patch\) rank \(NA\) serial number \(0\)

$ ./main7.pl6 --name=Patch --serial-num=1656401 --rank=Private

name \(Patch\) rank \(Private\) serial number \(1656401\)

\`\`\`



Even though Perl is checking the types \(string, integer\) for our arguments, it's not enforcing any validation on the value. We can add a &lt;a href="https://docs.perl6.org/type/Signature\#index-entry-where\_clause\_%28Signature%29"&gt;where clause&lt;/a&gt; to ensure we only get a positive value for --serial-num:



\`\`\`

$ cat main8.pl6

\#!/usr/bin/env perl6



sub MAIN \(

    Str :$name!, 

    Str :$rank='NA', 

    Int :$serial-num where \* &gt; 0 = 1

\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

$ ./main8.pl6 --name=Patch --serial-num=-10

Usage:

  ./main8.pl6 --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

$ ./main8.pl6 --name=Patch

name \(Patch\) rank \(NA\) serial number \(1\)

\`\`\`



As it turns out, the built-in &lt;a href="https://docs.perl6.org/type/UInt"&gt;UInt&lt;/a&gt; type will work for ensuring positive, so maybe we would be better off ensuring that the serial number is of the correct length:



\`\`\`

$ cat main9.pl6

\#!/usr/bin/env perl6



sub MAIN \(

    Str :$name!,

    Str :$rank='NA',

    UInt :$serial-num where \*.Str.chars == 7 = 1111111

\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}

$ ./main9.pl6 --name=Patch --serial-num=12345

Usage:

  ./main9.pl6 --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

$ ./main9.pl6 --name=Patch --serial-num=1234567

name \(Patch\) rank \(NA\) serial number \(1234567\)

\`\`\`



Unfortunately the usage doesn't include information about the length, so we can override the default &lt;a href="https://docs.perl6.org/language/functions\#index-entry-USAGE"&gt;USAGE&lt;/a&gt;:



\`\`\`

$ cat main10.pl6

\#!/usr/bin/env perl6



sub MAIN \(

    Str :$name!,

    Str :$rank='NA',

    UInt :$serial-num where \*.Str.chars == 7 = 1111111

\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}



sub USAGE {

    printf "Usage:\n  %s --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]\n",

        $\*PROGRAM.basename;



    put "Note: --serial-num must be 7 digits in length."

}

$ ./main10.pl6

Usage:

  main10.pl6 --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

Note: --serial-num must be 7 digits in length.

\`\`\`



I can move the --serial-num code into a &lt;a href="https://docs.perl6.org/language/typesystem\#index-entry-subset-subset"&gt;subset&lt;/a&gt; and further constrain &lt;code&gt;--rank&lt;/code&gt; to a list of acceptable values:



\`\`\`

$ cat main11.pl6

\#!/usr/bin/env perl6



subset SerialNum of UInt where \*.Str.chars == 7;

my @ranks = &lt;NA Recruit Private PSC PFC Specialist&gt;;

subset Rank of Str where \* eq any\(@ranks\);



sub MAIN \(

    Str :$name!,

    Rank :$rank='NA',

    SerialNum :$serial-num=1111111

\) {

    put "name \($name\) rank \($rank\) serial number \($serial-num\)";

}



sub USAGE {

    printf "Usage:\n  %s --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]\n",

        $\*PROGRAM.basename;



    put "--rank must be one of: {@ranks.join\(', '\)}";

    put "--serial-num must be 7 digits in length."

}

$ ./main11.pl6 --name=Patch --serial-num=1234567 --rank=XXX

Usage:

  main11.pl6 --name=&lt;Str&gt; \[--rank=&lt;Str&gt;\] \[--serial-num=&lt;Int&gt;\]

--rank must be one of: NA, Recruit, Private, PSC, PFC, Specialist

--serial-num must be 7 digits in length.

$ ./main11.pl6 --name=Patch --serial-num=1234567 --rank=Private

name \(Patch\) rank \(Private\) serial number \(1234567\)

\`\`\`





