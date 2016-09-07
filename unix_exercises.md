# Unix exercises

> Note: When you see a "$" given in the example prompts, it is a metacharacter indicating that this is the prompt for a normal (not super-user) account.  Your default prompt may be different, and it is highly configurable (search for "PS1 unix prompt" to learn more).  Anyway, point is that you should type (copy/paste) all the stuff *after* the $.  If you ever see a prompt with "#," it's indicating a command that should be run as the super-user/root account, e.g., installing some software into a system-wide directory so it can be shared by all users.

## Number of unique users

Find the number of unique users on a shared system

We know that "w" will tell us the users logged in.  Try it now on a system that has many users (i.e., not your laptop) and see the output.  We'll connect the output of "w" to "head" using a pipe "|", but we only want the first five lines:

```
$ w | head -5
 14:36:01 up 21 days, 21:51, 176 users,  load average: 3.83, 4.31, 4.47
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
antontre pts/1    149.165.156.129  Sat14    3days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    4:56m  0.15s  0.15s -bash
```

We want to see the first five *users*, not the first five lines of output.  To skip the first two lines of headers from "w," we first pipe "w" into "awk" and tell it we only want to see output when the Number of Records (NR) is greater than 2:

```
$ w | awk 'NR>2' | head -5
antontre pts/1    149.165.156.129  Sat14    4days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    5:13m  0.15s  0.15s -bash
antontre pts/5    149.165.156.129  Sun19    2days  0.14s  0.14s /bin/sh -i
minyard  pts/8    129.114.64.18    29Jul16  4:24m  3:46m  3:46m top
antontre pts/11   149.165.156.129  Sun23    2days  0.24s  0.24s /bin/sh -i
```

Let's "cut" out just the first column of data.  The manpage for "cut" says that it defaults to using the tab character to determine columns, so we'll need to tell it to use spaces:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1
antontre
huddack
antontre
minyard
antontre
```

We can see right away that the some users like "antontre" are logged in multiple times, so let's "uniq" that output:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | uniq
antontre
huddack
antontre
minyard
antontre
```

Hmm, that's not right.  Remember I said earlier that "uniq" only works *on sorted input*?  So let's sort those names first:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | sort | uniq
antontre
huddack
minyard
```

OK, that is correct.  Now let's remove the "head -5" and use "wc" to count all the lines (-l) of input:

```
$ w | awk 'NR>2' | cut -d ' ' -f 1 | sort | uniq | wc -l
138
```

So what you see is that we're connecting small, well-defined programs together using pipes to connect the "standard input" (STDIN) and "standard output (STDOUT) streams.  There's a third basic file handle in Unix called "standard error" (STDERR) that we'll come across later.  It's a way for programs to report problems without simply dying.  You can redirect errors into a file like so:

```
$ program 2>err
$ program 1>out 2>err
```

The first example puts STDERR into a file called "err" and lets STDOUT print to the terminal.  The second example captures STDOUT into a file called "out" while STDERR goes to "err."  

> Protip: Sometimes a program will complain about things that you cannot fix, e.g., "find" may complain about file permissions that you don't care about.  In those cases, you can redirect STDERR to a special filehandle called "/dev/null" where they are forgotten forever.  Kind of like the "memory hole" in 1984.

```
find / -name my-file.txt 2>/dev/null
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

> Protip: How do we know we got the correct data?  Go back and look at that FTP site, and you will see that there is a "contigs.zip.md5" file that we can "less" on the server to view the contents:

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

> You can read up on MD5 (https://en.wikipedia.org/wiki/Md5sum) to understand that this is a signature of the file.  If we calculate the MD5 of the file we dowloaded and it matches what we see on the server, then we can be sure that we have the exact file that is on the FTP site:

```
$ md5sum contigs.zip
1b7e58177edea28e6441843ddc3a68ab  contigs.zip
```

> Yes, those two sums match.  Note that sometimes the program is just called "md5."

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
```

That last one removes the FASTA file artifact that identifies the beginning of an ID but is not part of the ID.  We can use this with "seqmagick" now to extract those sequences and find out how many were found:

```
$ seqmagick convert --include-from-file newclean_ids group12_contigs.fasta newgroup12_contigs.fasta
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

# Find unclustered protein sequences

For this exercise, use the UA HPC and do the following setup:

```
$ mkdir ~/work/abe487/unclustered-proteins
$ cd !$
$ wget ftp://ftp.imicrobe.us/abe487/exercises/unclustered-proteins.tgz
$ tar xvf unclustered-proteins.tgz
$ cd unclustered-proteins
```

The "README" contains our instructions:

```
$ cat README
# Find unclustered proteins

The file "cdhit60.3+.clstr" contains all of the GI numbers for
proteins that were clustered and put into hmm profiles.  The file
"proteins.fa" contains all proteins (the header is only the GI
number).  Extract the proteins from the "proteins.fa" file that were
not clustered.
```

If you type "make" in the directory, you will execute a small pipeline to do this job:

```
$ make
find . \( -name unclustered-proteins.fa -o -name clean-proteins.fa -o -name \*.o \) -exec rm {} \;
grep -ve '^>' cdhit60.3+.clstr | awk '{print $3}' | awk -F"|" '{print $2}' | sort > clustered-ids.o
sed "s/|.*//" proteins.fa > clean-proteins.fa
grep -e '^>' clean-proteins.fa | sed "s/^>//" | sort > protein-ids.o
comm -23 protein-ids.o clustered-ids.o > unclustered-ids.o
seqmagick convert --include-from-file unclustered-ids.o clean-proteins.fa unclustered-proteins.fa
seqmagick info unclustered-proteins.fa
name                    alignment    min_len   max_len   avg_len  num_seqs
unclustered-proteins.fa FALSE              0      7391    293.74    204264
```

Let's break this down step-by-step to understand what is happening by looking at the contents of the "Makefile":

```
$ cat -n Makefile
     1 	all: clean info
     2
     3 	clean:
     4 		find . \( -name unclustered-proteins.fa -o -name clean-proteins.fa -o -name \*.o \) -exec rm {} \;
     5
     6 	clustered-ids:
     7 		grep -ve '^>' cdhit60.3+.clstr | awk '{print $$3}' | awk -F"|" '{print $$2}' | sort | uniq > clustered-ids.o
     8
     9 	clean-proteins:
    10 		sed "s/|.*//" proteins.fa > clean-proteins.fa
    11
    12 	protein-ids: clean-proteins
    13 		grep -e '^>' clean-proteins.fa | sed "s/^>//" | sort > protein-ids.o
    14
    15 	unclustered-ids: clustered-ids protein-ids
    16 		comm -23 protein-ids.o clustered-ids.o > unclustered-ids.o
    17
    18 	unclustered-proteins: unclustered-ids
    19 		seqmagick convert --include-from-file unclustered-ids.o clean-proteins.fa unclustered-proteins.fa
    20
    21 	info: unclustered-proteins
    22 		seqmagick info unclustered-proteins.fa
    23
    24 	dist: clean
    25 		(cd .. && tar czvf unclustered-proteins.tgz unclustered-proteins)
    26
    27 	check: unclustered-proteins
    28 		./check.sh
```

The first defined target in a Makefile will be the default if none is supplied to ```make```, and it's typical to call this "all."  I set this target to simply be a combination of "clean" (to get rid of all the intermediate files -- I have to use this complex ```find``` command so as not to encounter a failure if I were to ```rm``` a file that does not exist) and "info."  Notice that most of the targets indicate a dependency, so executing "info" requires that "unclustered-proteins" be run which in turn needs "unclustered-ids" and so on such that the first action that must be run is "clustered-ids." 

The first task in "clustered-ids" is to find those protein IDs in the "cdhit60.3+.clstr" file that were clustered by ```cd-hit```.  Let's look at that file:

```
$ head -5 cdhit60.3+.clstr
>Cluster_5086
0	358aa, >gi|317183610|gb|ADV... at 66.76%
1	361aa, >gi|315661179|gb|ADU... at 70.36%
2	118aa, >gi|375968555|gb|AFB... at 70.34%
3	208aa, >gi|194307477|gb|ACF... at 61.54%
```

The format of the file is similar to a FASTA file where the ">" sign at the left-most column identifies a cluster with the following lines showing the IDs of the sequences in the cluster.  To extract just the clustered IDs, we cannot just do ```grep '>'``` as we'll get both the cluster IDs and the protein IDs.  

```
$ grep '>' cdhit60.3+.clstr | head -5
>Cluster_5086
0	358aa, >gi|317183610|gb|ADV... at 66.76%
1	361aa, >gi|315661179|gb|ADU... at 70.36%
2	118aa, >gi|375968555|gb|AFB... at 70.34%
3	208aa, >gi|194307477|gb|ACF... at 61.54%
```

We'll need to use a regular expression (the ```-e``` for "extended" on most greps, but sometimes not required):

```
$ grep -e '^>' cdhit60.3+.clstr | head -5
>Cluster_5086
>Cluster_10030
>Cluster_8374
>Cluster_13356
>Cluster_7732
```

and then invert that with "-v":

```
$ grep -v '^>' cdhit60.3+.clstr | head -5
0	358aa, >gi|317183610|gb|ADV... at 66.76%
1	361aa, >gi|315661179|gb|ADU... at 70.36%
2	118aa, >gi|375968555|gb|AFB... at 70.34%
3	208aa, >gi|194307477|gb|ACF... at 61.54%
4	358aa, >gi|291292536|gb|ADD... at 68.99%
```

From this output, we'd like to extract the ">gi|317183610|gb|ADV..." bit, which is the third column when split on whitespace.  The tool ```awk``` is perfect for this, and whitespace is the default split point (as opposed to ```cut``` which uses tabs):

```
$ grep -ve '^>' cdhit60.3+.clstr | awk '{print $3}' | head -5
>gi|317183610|gb|ADV...
>gi|315661179|gb|ADU...
>gi|375968555|gb|AFB...
>gi|194307477|gb|ACF...
>gi|291292536|gb|ADD...
```

If we look at the IDs in the proteins file, we'll see they are integers:

```
$ grep '>' proteins.fa | head -5
>388548806
>388548807
>388548808
>388548809
>388548810
```

So we see that from "gi|317183610|gb|ADV..." we need to extract just the second field when splitting on the vertical bar.  Again, ```awk``` is perfect, but we need to tell it to split on something other than the default by using the "-F" flag:

```
$ grep -ve '^>' cdhit60.3+.clstr | awk '{print $3}' | awk -F'|' '{print $2}' | head -5
317183610
315661179
375968555
194307477
291292536
```

  These are the protein IDs for those that were successfully clustered, so we just need to capture these to a file which we can do with a redirect ```>```.  Looking ahead, I know that these will need to be sorted for another tool to work, so I'll add that to the pipeline.  Also, each protein might be clustered more than once, so I should ```uniq``` the list, and you'll recall that input must be sorted first. You'll notice that this is now the "clustered-ids" target.  Because the dollar sign is used by ```make``` to indicate variables, we need to escape them with another dollar sign to make them literals:

```
clustered-ids:
       	grep -ve '^>' cdhit60.3+.clstr | awk '{print  $$3}' | awk -F"|" '{print $$2}' | sort | uniq > clustered-ids.o
```

Now we need to extract the protein IDs which we can do with ```grep```, but we also need to remove the ">" sign.  Additionally, some of the IDs have characters other than digits that we will need to remove.  To demonstrate this, I'll use ```grep -P``` to indicate a Perl regular expression and combine with "-v" to invert it:

```
$ grep -e '^>' proteins.fa | sed "s/^>//" | grep -v -P '^\d+$' | head -5
26788002|emb|CAD19173.1| putative RNA helicase, partial [Agaricus bisporus virus X]
26788000|emb|CAD19172.1| putative RNA helicase, partial [Agaricus bisporus virus X]
985757046|ref|YP_009222010.1| hypothetical protein [Alternaria brassicicola fusarivirus 1]
985757045|ref|YP_009222011.1| hypothetical protein [Alternaria brassicicola fusarivirus 1]
985757044|ref|YP_009222009.1| polyprotein [Alternaria brassicicola fusarivirus 1]
```

Let's break down that regex:

```
^ \d + $
1 2  3 4
```

1. start of the line
2. a digit (0-9)
3. one or more
4. end of the line

Looking at the above output, we can see that it would be pretty easy to get rid of everything starting with the vertical bar.  Since that is not a part of the sequence, we should be safe using ```sed``` to clean up the proteins file:

```
clean-protein-ids:
    sed "s/|.*//" proteins.fa > proteins-clean.fa
```

Also, I'll sort the output for use by a later step and redirect to a file, and this now becomes the "protein-ids" target:

```
protein-ids: clean-proteins
       	grep -e '^>' clean-proteins.fa | sed "s/^>//" | sort > protein-ids.o
```

To find the lines in "protein-ids" that are not in "clustered-ids," I can use the ```comm``` (common) command.  Notice that this target has two dependencies:

```
unclustered-ids: clustered-ids protein-ids
       	comm -23 protein-ids.o clustered-ids.o > unclustered-ids.o
```

To extract the actual sequences from the "proteins.fa" file using the IDs in "unclustered-ids," ```seqmagick convert``` program is a great choice:

```
unclustered-proteins: unclustered-ids
       	seqmagick convert --include-from-file unclustered-ids.o clean-proteins.fa unclustered-proteins.fa
```

Lastly, we'd like to see confirmation that we got a reasonable output, so we can examine the resulting FASTA file with ```seqmagick info```:

```
info: unclustered-proteins
    seqmagick info unclustered-proteins.fa
```

In the end, we need to verify that we have a reasonable answer.  Let's add the number of clustered and unclustered protein IDs and see if they total the original number of proteins:

```
$ seqmagick info unclustered-proteins.fa
name                    alignment    min_len   max_len   avg_len  num_seqs
unclustered-proteins.fa FALSE              0      7391    293.74    204264
```

So we found 204,264 unclustered proteins sequences.  Is that the right number?

```
$ wc -l clustered-ids.o
16257 clustered-ids.o
$ wc -l unclustered-ids.o
204263 unclustered-ids.o
$ bc <<< 16257+204263
220520
$ seqmagick info proteins.fa
name        alignment    min_len   max_len   avg_len  num_seqs
proteins.fa FALSE              0      7391    298.18    220520
```

I can put that into a shell script:

```
$ cat -n check.sh
     1 	#!/bin/bash
     2
     3 	set -u
     4
     5 	function lc() {
     6 	  wc -l $1 | cut -d ' ' -f 1
     7 	}
     8
     9 	echo $(lc clustered-ids.o)   > count-clustered.o
    10 	echo $(lc unclustered-ids.o) > count-unclustered.o
    11
    12 	bc <<< "$(cat count-clustered.o)+$(cat count-unclustered.o)" > count.o
    13
    14 	grep -e '^>' proteins.fa | cut -d ' ' -f 1 | wc -l > proteins-count.o
    15
    16 	MYCOUNT=$(cat count.o)
    17 	PROTCOUNT=$(cat proteins-count.o)
    18
    19 	if [[ "$MYCOUNT" -eq "$PROTCOUNT" ]]; then
    20 	  echo "Counts match, all's good."
    21 	else
    22 	  echo "Not OK (MYCOUNT='$MYCOUNT', PROTCOUNT='$PROTCOUNT')";
    23 	fi
```

And run it as a target:

```
$ make check
...
Counts match, all's good.
```

When I create this pipeline, I worked out each step and then added it to the "Makefile."  In doing so, I completely documented my work while also creating a way to execute the entire workflow in one command, ```make```.  Huzzah for reproducibility!