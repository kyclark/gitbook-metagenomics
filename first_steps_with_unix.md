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

* <TAB>: hit the Tab key for command completion
* **!!**: execute the last command again
* **!$**: the last argument from your previous command line (think of the $ as the right anchor in a regex)
* CTRL-R: reverse search of your history
* Up/down cursor keys: go backwards/forwards in your history
* CTRL-A, CTRL-E: jump to the start, end of the command line when in emacs mode (default)

Protip: If you are on a Mac, it's easy to remap your (useless) CAPSLOCK key to CTRL.  Much less strain on your hand as you will find you need CTRL quite a bit, even more so if you choose emacs for your $EDITOR.

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

Side note: Dotfiles (https://en.wikipedia.org/wiki/Hidden_file_and_hidden_directory) are files with names that begin with a dot "." and include:

* **.**: the current directory
* **..**: the parent directory

They are normally hidden from view unless you use "ls -a" to list "all" files.

Your ".bashrc" (or maybe ".profile" or maybe ".bash_profile" depending on your system) file is read every time you login to your system, so you can remember your customizations.

# Aliases

Sometimes you'll find you're using a particular command quite often and want to create a shortcut.  You can assign any command to a single "alias" like so:

```
alias cx='chmod +x'
alias up2='cd ../../'
alias up3='cd ../../../'
```

If you execute this on the command line, the alias will be saved until you log out.  Adding this to your .bashrc will 
make it available every time you log in.  When you make a change and want the shell to bring those into the current environment, you need to "source" the file.  The command "." is an alias for "source":

```
$ source ~/.bashrc
```

# File system layout

The top level of a Unix file system is "/" which is called "root."  Confusingly, there is also an account named "root" which is basically the super-user/sysadmin (systems administrator).  Unix has always been a multi-tenant system

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

## Count "oo" words

On almost every Unix system, you can find "/usr/share/dict/words."  Let's use "grep" to find how many have the "oo" vowel combination.  It's a long list, so I'll pipe it into "head" to see just the first five:

```
$ grep 'oo' /usr/share/dict/words | head -5
abloom
aboon
aboveproof
abrood
abrook
```

Yes, that works, so redirect those words into a file and count them:

```
$ grep 'oo' /usr/share/dict/words > oo-words
$ wc -l !$
10460 oo-words
```

Let's count them directly out of "grep":

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

## Something with sequences

Now we will get some sequence data from the iMicrobe FTP site.  Both "wget" and "ncftpget" will do the trick:

```
$ mkdir -p ~/work/abe487/contigs
$ cd !$
$ wget ftp://ftp.imicrobe.us/abe487/contigs/contigs.zip
```

Side track: How do we know we got the correct data?  Go back and look at that FTP site, and you will see that there is a "contigs.zip.md5" file that we can "less" on the server to view the contents:

```
$ ncftp ftp://ftp.imicrobe.us/abe487/contigs
NcFTP 3.2.5 (Feb 02, 2011) by Mike Gleason (http://www.NcFTP.com/contact/).
Connecting to 128.196.131.100...
Welcome to the imicrobe.us repository
Logging in...
Login successful.
Logged in to ftp.imicrobe.us.
Current remote directory is /abe487/contigs.
ncftp /abe487/contigs > ls
contigs.zip        contigs.zip.md5
ncftp /abe487/contigs > less contigs.zip.md5

1b7e58177edea28e6441843ddc3a68ab  contigs.zip
ncftp /abe487/contigs > exit
```

You can read up on MD5 (https://en.wikipedia.org/wiki/Md5sum) to understand that this is a signature of the file.  If we calculate the MD5 of the file we dowloaded and it matches what we see on the server, then we can be sure that we have the exact file that is on the FTP site:

```
$ md5sum contigs.zip
1b7e58177edea28e6441843ddc3a68ab  contigs.zip
```

Yes, those two sums match.  Note that sometimes the program is just called "md5."

So, back to the exercise.  Let's unpack the contigs:

```
$ unzip contigs.zip
Archive:  contigs.zip
  inflating: group12_contigs.fasta
  inflating: group20_contigs.fasta
  inflating: group24_contigs.fasta
$ rm contigs.zip
```

These files are in FASTA format (https://en.wikipedia.org/wiki/FASTA_format), which basically looks like this:

```
>MCHU - Calmodulin - Human, rabbit, bovine, rat, and chicken
ADQLTEEQIAEFKEAFSLFDKDGDGTITTKELGTVMRSLGQNPTEAELQDMINEVDADGNGTID
FPEFLTMMARKMKDTDSEEEIREAFRVFDKDGNGYISAAELRHVMTNLGEKLTDEEVDEMIREA
DIDGDGQVNYEEFVQMMTAK*
>gi|5524211|gb|AAD44166.1| cytochrome b [Elephas maximus maximus]
LCLYTHIGRNIYYGSYLYSETWNTGIMLLLITMATAFMGYVLPWGQMSFWGATVITNLFSAIPYIGTNLV
EWIWGGFSVDKATLNRFFAFHFILPFTMVALAGVHLTFLHETGSNNPLGLTSDSDKIPFHPYYTIKDFLG
LLILILLLLLLALLSPDMLGDPDNHMPADPLNTPLHIKPEWYFLFAYAILRSVPNKLGGVLALFLSIVIL
GLMPFLHTSKHRSMMLRPLSQALFWTLTMDLLTLTWIGSQPVEYPYTIIGQMASILYFSIILAFLPIAGX
IENY
```

Header lines start with ">", then the sequence follows.  Sequences may be broken up over several lines of 50 or 80 characters, but it's just as common to see the sequences take only one (sometimes very long) line.  Sequences may be nucleotides, proteins, very short DNA/RNA, longer contigs (shorter strands assembled into contiguous regions), or entire chromosomes or even genomes.

So, how many sequences are in "group12_contigs.fasta"?  To answer, we just need to count how many times we see ">".  We can do that with "grep":

```
$ grep > group12_contigs.fasta
Usage: grep [OPTION]... PATTERN [FILE]...
Try 'grep --help' for more information.
```

What is going on?  Remember when we captured the "oo" words that we used the ">" symbol to tell Unix to *redirect* the output of "grep" into a file.  We need to tell Unix that we mean a literal greater-than sign by placing it in single or double quotes or putting a backslash in front of it:

```
$ grep '>' group12_contigs.fasta
$ grep \> group12_contigs.fasta
```

You should actually see nothing because something quite insidious happened with that first "grep" statement -- it overwrote our original "group12_contigs.fasta" with the result of "grep"ing for nothing, which is nothing:

```
$ ls -l group12_contigs.fasta
-rw-rw---- 1 kyclark staff 0 Aug 10 15:08 group12_contigs.fasta
```

Ugh, OK, I have to go back and "wget" the "contigs.zip" file to restore it.  That's OK.  Things like this happen all the time.  

```
$ ls -lh group12_contigs.fasta
-rw-rw---- 1 kyclark staff 2.9M Aug 10 14:38 group12_contigs.fasta
```

Now that I have restored my data, I want to count how many greater-than signs in the file:

```
$ grep '>' group12_contigs.fasta | wc -l
132
```

Hey, I could see doing that often.  Maybe we should make this into an "alias" (see above).  The problem is that the "argument" to the function (the filename) is stuck in the middle of the chain of commands, so it would make it tricky to use an alias for this.  We can create a bash function that we add to our .bashrc:

```
function countseqs() {
  grep '>' $1 | wc -l
}
```

After you add this, remember to source this file to make it available:

```
$ source ~/.bashrc
$ countseqs group12_contigs.fasta
132
```

Same answer.  Good.  However, someone beat us to the punch.  There is a powerful tool called "seqmagick" (https://github.com/fhcrc/seqmagick) that will do this (and much, much more).  It's installed into the "hurwitzlab/bin" directory, or you can install it locally:

```
$ seqmagick info group12_contigs.fasta
name                  alignment    min_len   max_len   avg_len  num_seqs
group12_contigs.fasta FALSE           5136    116409  22974.30       132
```

Run "seqmagick -h" to see everything it can do.

Moving on, let's find how many contig IDs in "group12_contigs.fasta" contain the number "47":

```
$ grep 47 group12_contigs.fasta > group12_ids_with_47
[login3@~/work/sequences]$ cat !$
cat group12_ids_with_47
>Contig_247
>Contig_447
>Contig_476
>Contig_1947
>Contig_4764
>Contig_4767
>Contig_13471
```

Here we did a little "useless use of cat," but it's OK.  We also could have used "less" to view the file.  Here's another useless use of cat to copy a file:

```
$ cat group12_ids_with_47 > temp1_ids
```

Additionally, we want to copy the file again to make duplicates:

```
$ cp group12_ids_with_47 temp2_ids
```

How can we be sure these files are the same?  Let's use "diff":

```
$ diff temp1_ids temp2_ids
```

You should see nothing, which is a case of "no news is good news."  The don't differ in any way.  We can verify this with "md5sum":

```
$ md5sum temp*
957390ab4c31db9500d148854f542eee  temp1_ids
957390ab4c31db9500d148854f542eee  temp2_ids
```

They are the same file.  If there were even one character difference, they would generate different hashes.

Now we will create a file with duplicate IDs:

```
$ cat temp1_ids temp2_ids > duplicate_ids
```

Check contents of "duplicate_ids" using "less" or "cat."  Now grab all of the contigs IDs from "group20_contigs.fasta" that contain the number "51."  Concatenate the new IDs to the duplicate_ids file in a file called "multiple_ids":

```
$ cp duplicate_ids multiple_ids
$ grep 51 group20_contigs.fasta >> !$
grep 51 group20_contigs.fasta >> multiple_ids
```

Notice the ">>" arrows to indicate that we are *appending* to the existing "multiple_ids" file.

Remove the existing "temp" files using a "*" wildcard:

```
$ rm temp*
```

Now let's explore more of what "sort" and "uniq" can do for us.  We want to find which IDs are unique and which are duplicated.  If we read the manpage ("man uniq"), we see that there are "-d" and "-u" flags for doing just that.  However, we've already seen that input to "uniq" needs to be sorted, so we need to remember to do that:

```
$ sort multiple_ids | uniq -d > temp1_ids
$ sort multiple_ids | uniq -u > temp2_ids
$ diff temp*
1,7c1,11
< >Contig_13471
< >Contig_1947
< >Contig_247
< >Contig_447
< >Contig_476
< >Contig_4764
< >Contig_4767
---
> >Contig_10051
> >Contig_1651
> >Contig_4851
> >Contig_5141
> >Contig_5143
> >Contig_5164
> >Contig_5170
> >Contig_5188
> >Contig_6351
> >Contig_9651
> >Contig_9851
```

Let's remove our temp files again and make a "clean_ids" file:

```
$ rm temp*
$ sort multiple_ids | uniq > clean_ids
```

We can use "sed" to alter the IDs.  The "s//" command say to "substitute" the first thing with the second thing, e.g., to replace all occurences of "foo" with "bar", use "s/foo/bar" (http://stackoverflow.com/questions/4868904/what-is-the-origin-of-foo-and-bar).

```
$ sed 's/C/c/' clean_ids
$ sed 's/_/./' clean_ids
# sed 's/>//' clean_ids > newclean_ids

That last one removes the FASTA file artifact that identifies the beginning of an ID but is not part of the ID.  We can use this with "seqmagick" now to extract those sequences and find out how many were found:

```
seqmagick convert --include-from-file newclean_ids group12_contigs.fasta newgroup12_contigs.fasta
$ seqmagick info !$
seqmagick info newgroup12_contigs.fasta
name                     alignment    min_len   max_len   avg_len  num_seqs
newgroup12_contigs.fasta FALSE           5587     30751  16768.14         7
```

We can get stats on all our files:

```
$ seqmagick info *fasta > fasta-info
$ cat !$
name                     alignment    min_len   max_len   avg_len  num_seqs
group12_contigs.fasta    FALSE           5136    116409  22974.30       132
group20_contigs.fasta    FALSE           5029     22601   7624.38       203
group24_contigs.fasta    FALSE           5024     81329  12115.70       139
newgroup12_contigs.fasta FALSE           5587     30751  16768.14         7
```

We can use "cut" to view various columns:

```
$ cut -f 2 fasta-info
$ cut -f 2,4 fasta-info
$ cut -f 2-4 fasta-info
```

But it does not line up very nicely.  We can use "column" to fix this:

```
$ cut -f 2-4 fasta-info | column -t
alignment  min_len  max_len
FALSE      5136     116409
FALSE      5029     22601
FALSE      5024     81329
FALSE      5587     30751
```

## Abandoned idea

We will now use the NCBI SRA Toolkit to download a small data set to play with.  If you are on the UA HPC, then you should have "/rsgrps/bhurwitz/hurwitzlab/bin" in your $PATH so that ```which fastq-dump``` should return "/rsgrps/bhurwitz/hurwitzlab/bin/fastq-dump".  If you do not have access to this directory, then you can install the program following the directions https://ncbi.github.io/sra-tools/.  If you are on another shared system like stampede, you might be able to load pre-built binaries with "module load."  You can first "module spider sra" or "module keyword sra" to see if it is available and then "module load sratoolkit."  You can always go to the NCBI SRA website at "https://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?view=run_browser&run=SRR390728" and directly download the FASTQ file.  Alternately, you could use "wget" or "ncftpget" to directly download the file from the NCBI FTP site:

```
$ wget ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR304/SRR304976/SRR304976.sra
```

But then you will still need to use "fastq-dump" to extract the FASTQ from the SRA file.  

We will make a directory for our work:

```
$ mkdir -p ~/work/SRR390728
$ cd !$
$ fastq-dump SRR390728
Read 7178576 spots for SRR390728
Written 7178576 spots for SRR390728
```

If all goes well, then you should have the file "SRR390728.fastq" in your directory.  How big is the file?

```
$ ls -lh SRR390728.fastq
-rw-rw-r-- 1 kyclark staff 1.5G Aug 10 13:30 SRR390728.fastq
```

Very large.  We *do not* want to open this in a text editor!  Let's peek:

```
$ head SRR390728.fastq
@SRR390728.1 1 length=72
CATTCTTCACGTAGTTCTCGAGCCTTGGTTTTCAGCGATGGAGAATGACTTTGACAAGCTGAGAGAAGNTNC
+SRR390728.1 1 length=72
;;;;;;;;;;;;;;;;;;;;;;;;;;;9;;665142;;;;;;;;;;;;;;;;;;;;;;;;;;;;;96&&&&(
@SRR390728.2 2 length=72
AAGTAGGTCTCGTCTGTGTTTTCTACGAGCTTGTGTTCCAGCTGACCCACTCCCTGGGTGGGGGGACTGGGT
+SRR390728.2 2 length=72
;;;;;;;;;;;;;;;;;4;;;;3;393.1+4&&5&&;;;;;;;;;;;;;;;;;;;;;<9;<;;;;;464262
@SRR390728.3 3 length=72
CCAGCCTGGCCAACAGAGTGTTACCCCGTTTTTACTTATTTATTATTATTATTTTGAGACAGAGCATTGGTC
```

You can go read about https://en.wikipedia.org/wiki/FASTQ_format in your copious free time, but basically we're dealing with a really terrible file format (bioinformatics is full of them!).  The only sane FASTQ format sticks to strictly 4 lines per sequence:

* header
* sequence
* header/spacer
* quality scores

We could find out how many sequences are present by counting the number of lines and dividing by four:

```
$ wc -l SRR390728.fastq
28714304 SRR390728.fastq
$ bc <<< 28714304/4
7178576
```

Or we could just convert this to FASTA format.  There is a handy program called "fastq_to_fasta" from the Fastx Toolkit (http://hannonlab.cshl.edu/fastx_toolkit/) that we have installed. 

```
$ fastq_to_fasta SRR390728.fastq
```

Protip: Want to know how long a command takes?  Put "time" in front of it.