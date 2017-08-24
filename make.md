# Organizing with Makefiles

GNU "make" \([https://www.gnu.org/software/make/](https://www.gnu.org/software/make/)\) will help you organize your code and testing.  You only need to learn a few things about make to be productive.

First, you create text file called "Makefile" \(or "makefile"\).  Whenever you run something on the command line that you might want to run again, you should probably put it into your Makefile so you don't have to go searching through your history to find the magic incantation of some command that actually did what you wanted.  Don't be like me and think "It's totally OK to leave my sunglasses in the driver's seat because I definitely will not forget they are there and then sit on them when I come back to my car."  Don't trust your memory, just add it to you Makefile.

If you check out "github.com:kyclark/metagenomics-book.git", you will find a "problems/dna" directory with a finished version of the "dna.pl6" program along with sample data, a test script, and a Makefile that looks like this

```
$ cat -n Makefile
     1	.PHONY: run file test
     2
     3	run:
     4		./dna.py AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
     5
     6	file:
     7		./dna.py test.txt
     8
     9	test:
    10		python3 -m pytest -v test.py
```

If you type `make`, you should see this:

```
$ make
./dna.py AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
20 12 17 21
```

Without any arguments, `make` will execute the first "target" which is basically a word-ish thing followed by a colon, which in this case is "run."  If you `make run`, you will see the same thing.  The `make file` target executes the script with a file as the input rather than a string, and `make test` will run the "test.pl6" script:

```
$ make test
python3 -m pytest -v test.py
============================= test session starts ==============================
platform darwin -- Python 3.5.3, pytest-3.0.7, py-1.4.33, pluggy-0.4.0 -- /Users/kyclark/anaconda3/bin/python3
cachedir: .cache
rootdir: /Users/kyclark/work/metagenomics-book/problems/dna, inifile:
collected 4 items

test.py::test_script_exists PASSED
test.py::test_usage PASSED
test.py::test_arg PASSED
test.py::test_file PASSED

=========================== 4 passed in 0.13 seconds ===========================
```

# Find unclustered protein sequences

A labmate of mine wanted help finding the sequences of proteins that failed to cluster.  Rather than writing the solution as a shell script, I found myself writing a Makefile as it easily allowed me to re-run certain steps while I worked out the kinks in my logic.

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
     1     all: clean info
     2
     3     clean:
     4         find . \( -name unclustered-proteins.fa -o -name clean-proteins.fa -o -name \*.o \) -exec rm {} \;
     5
     6     clustered-ids:
     7         grep -ve '^>' cdhit60.3+.clstr | awk '{print $$3}' | awk -F"|" '{print $$2}' | sort | uniq > clustered-ids.o
     8
     9     clean-proteins:
    10         sed "s/|.*//" proteins.fa > clean-proteins.fa
    11
    12     protein-ids: clean-proteins
    13         grep -e '^>' clean-proteins.fa | sed "s/^>//" | sort > protein-ids.o
    14
    15     unclustered-ids: clustered-ids protein-ids
    16         comm -23 protein-ids.o clustered-ids.o > unclustered-ids.o
    17
    18     unclustered-proteins: unclustered-ids
    19         seqmagick convert --include-from-file unclustered-ids.o clean-proteins.fa unclustered-proteins.fa
    20
    21     info: unclustered-proteins
    22         seqmagick info unclustered-proteins.fa
    23
    24     dist: clean
    25         (cd .. && tar czvf unclustered-proteins.tgz unclustered-proteins)
    26
    27     check: unclustered-proteins
    28         ./check.sh
```

The first defined target in a Makefile will be the default if none is supplied to `make`, and it's typical to call this "all."  I set this target to simply be a combination of "clean" \(to get rid of all the intermediate files -- I have to use this complex `find` command so as not to encounter a failure if I were to `rm` a file that does not exist\) and "info."  Notice that most of the targets indicate a dependency, so executing "info" requires that "unclustered-proteins" be run which in turn needs "unclustered-ids" and so on such that the first action that must be run is "clustered-ids."

The first task in "clustered-ids" is to find those protein IDs in the "cdhit60.3+.clstr" file that were clustered by `cd-hit`.  Let's look at that file:

```
$ head -5 cdhit60.3+.clstr
>Cluster_5086
0    358aa, >gi|317183610|gb|ADV... at 66.76%
1    361aa, >gi|315661179|gb|ADU... at 70.36%
2    118aa, >gi|375968555|gb|AFB... at 70.34%
3    208aa, >gi|194307477|gb|ACF... at 61.54%
```

The format of the file is similar to a FASTA file where the "&gt;" sign at the left-most column identifies a cluster with the following lines showing the IDs of the sequences in the cluster.  To extract just the clustered IDs, we cannot just do `grep '>'` as we'll get both the cluster IDs and the protein IDs.

```
$ grep '>' cdhit60.3+.clstr | head -5
>Cluster_5086
0    358aa, >gi|317183610|gb|ADV... at 66.76%
1    361aa, >gi|315661179|gb|ADU... at 70.36%
2    118aa, >gi|375968555|gb|AFB... at 70.34%
3    208aa, >gi|194307477|gb|ACF... at 61.54%
```

We'll need to use a regular expression \(the `-e` for "extended" on most greps, but sometimes not required\):

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
0    358aa, >gi|317183610|gb|ADV... at 66.76%
1    361aa, >gi|315661179|gb|ADU... at 70.36%
2    118aa, >gi|375968555|gb|AFB... at 70.34%
3    208aa, >gi|194307477|gb|ACF... at 61.54%
4    358aa, >gi|291292536|gb|ADD... at 68.99%
```

From this output, we'd like to extract the "&gt;gi\|317183610\|gb\|ADV..." bit, which is the third column when split on whitespace.  The tool `awk` is perfect for this, and whitespace is the default split point \(as opposed to `cut` which uses tabs\):

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

So we see that from "gi\|317183610\|gb\|ADV..." we need to extract just the second field when splitting on the vertical bar.  Again, `awk` is perfect, but we need to tell it to split on something other than the default by using the "-F" flag:

```
$ grep -ve '^>' cdhit60.3+.clstr | awk '{print $3}' | awk -F'|' '{print $2}' | head -5
317183610
315661179
375968555
194307477
291292536
```

These are the protein IDs for those that were successfully clustered, so we just need to capture these to a file which we can do with a redirect `>`.  Looking ahead, I know that these will need to be sorted for another tool to work, so I'll add that to the pipeline.  Also, each protein might be clustered more than once, so I should `uniq` the list, and you'll recall that input must be sorted first. You'll notice that this is now the "clustered-ids" target.  Because the dollar sign is used by `make` to indicate variables, we need to escape them with another dollar sign to make them literals:

```
clustered-ids:
           grep -ve '^>' cdhit60.3+.clstr | awk '{print  $$3}' | awk -F"|" '{print $$2}' | sort | uniq > clustered-ids.o
```

Now we need to extract the protein IDs which we can do with `grep`, but we also need to remove the "&gt;" sign.  Additionally, some of the IDs have characters other than digits that we will need to remove.  To demonstrate this, I'll use `grep -P` to indicate a Perl regular expression and combine with "-v" to invert it:

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
2. a digit \(0-9\)
3. one or more
4. end of the line

Looking at the above output, we can see that it would be pretty easy to get rid of everything starting with the vertical bar.  Since that is not a part of the sequence, we should be safe using `sed` to clean up the proteins file:

```
clean-protein-ids:
    sed "s/|.*//" proteins.fa > proteins-clean.fa
```

Also, I'll sort the output for use by a later step and redirect to a file, and this now becomes the "protein-ids" target:

```
protein-ids: clean-proteins
           grep -e '^>' clean-proteins.fa | sed "s/^>//" | sort > protein-ids.o
```

To find the lines in "protein-ids" that are not in "clustered-ids," I can use the `comm` \(common\) command.  Notice that this target has two dependencies:

```
unclustered-ids: clustered-ids protein-ids
           comm -23 protein-ids.o clustered-ids.o > unclustered-ids.o
```

To extract the actual sequences from the "proteins.fa" file using the IDs in "unclustered-ids," `seqmagick convert` program is a great choice:

```
unclustered-proteins: unclustered-ids
           seqmagick convert --include-from-file unclustered-ids.o clean-proteins.fa unclustered-proteins.fa
```

Lastly, we'd like to see confirmation that we got a reasonable output, so we can examine the resulting FASTA file with `seqmagick info`:

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
     1    #!/bin/bash
     2
     3    set -u
     4
     5    function lc() {
     6      wc -l $1 | cut -d ' ' -f 1
     7    }
     8
     9    echo $(lc clustered-ids.o)   > count-clustered.o
    10    echo $(lc unclustered-ids.o) > count-unclustered.o
    11
    12    bc <<< "$(cat count-clustered.o)+$(cat count-unclustered.o)" > count.o
    13
    14    grep -e '^>' proteins.fa | cut -d ' ' -f 1 | wc -l > proteins-count.o
    15
    16    MYCOUNT=$(cat count.o)
    17    PROTCOUNT=$(cat proteins-count.o)
    18
    19    if [[ "$MYCOUNT" -eq "$PROTCOUNT" ]]; then
    20      echo "Counts match, all's good."
    21    else
    22      echo "Not OK (MYCOUNT='$MYCOUNT', PROTCOUNT='$PROTCOUNT')";
    23    fi
```

And run it as a target:

```
$ make check
<...output elided...>
Counts match, all's good.
```

When I create this pipeline, I worked out each step and then added it to the "Makefile."  In doing so, I completely documented my work while also creating a way to execute the entire workflow in one command, `make`.  Huzzah for reproducibility!

