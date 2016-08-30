# PBS

The University of Arizona's HPC cluster uses the "PBS Pro" (portable batch system) scheduler.

Here are some important links:

* hpc-consult@list.arizona.edu (email list for help)
* http://rc.arizona.edu/hpc-htc/high-performance-computing-high-throughput-computing
* http://rc.arizona.edu/hpc-htc/using-systems/pbs-example

# Hello

Here is a "hello" script and a Makefile to submit it:

```
$ cat -n hello.sh
     1 	#!/bin/bash
     2
     3 	#PBS -W group_list=bhurwitz
     4 	#PBS -q standard
     5 	#PBS -l jobtype=cluster_only
     6 	#PBS -l select=1:ncpus=1:mem=1gb
     7 	#PBS -l walltime=01:00:00
     8 	#PBS -l cput=01:00:00
     9
    10 	echo "Hello from sunny \"$(hostname)\"!"
```


```
$ cat -n Makefile
     1 	submit: clean
     2 		qsub hello.sh
     3
     4 	clean:
     5 		find . -name hello.sh.[eo]* -exec rm {} \;  
```



```
$ make
find . -name hello.sh.[eo]* -exec rm {} \;
qsub hello.sh
818089.service0
$ qs

service0:
                                                            Req'd  Req'd   Elap
Job ID          Username Queue    Jobname    SessID NDS TSK Memory Time  S Time
--------------- -------- -------- ---------- ------ --- --- ------ ----- - -----
818089.service0 kyclark  clu_stan hello.sh      --    1   1    1gb 01:00 Q   --
```