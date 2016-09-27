# HPC

HPC is an acronym for "high-performance computing," and it generally means using a cluster of computers.  Our students have access to several clusters (Ocelote, HPC, ICE) at the University of Arizona, and most anyone is welcome to use the clusters at TACC.  To use a cluster, it's necessary to submit a batch job along with a description of the resources you need (e.g., memory, number of CPUs, number of nodes) to a scheduler that will start your job when the resources become available.  We will discuss schedulers "PBS" used at UA and "SLURM" used at TACC.  In the Github repo, you will find an "hpc" directory that contains examples for submitting to each queue.

To interact PBS and SLURM, you must log in to the "head" node(s).  Often you will be placed on a random nodes such as "login1."  YOU ARE NOT ALLOWED TO DO HEAVY LIFTING ON THE HEAD NODE.  For our class, you can write files, interact with the Perl RELP, run small scripts, etc., but you should never run BLAST or launch long-running jobs on these machines.  They are intended to be used to submit jobs to the queue.

# Handy aliases

To make it easier to go back and forth between PBS and SLURM, I create aliases so that I can execute the same command on both systems:

```
alias qstat="/usr/local/bin/qstat_local"
ME="kyclark"
alias qs="qstat -u $ME"
alias qt="qstat -Jtu $ME"
function qkill() {
  if [[ "${#1}" -eq 0 ]]; then
    echo {Now I crush you!"
    OUT=$(qstat -u $ME | grep $ME | cut -f 1 -d ' ' | sed 's/\[\]\..*/[]/' | xargs qdel)

    if [[ $? -eq 0 ]]; then
        echo "Jobs killed"
    else
        echo -e "\nError submitting job\n$OUT\n"
    fi
  else
    echo Argument = \"$1\"
    echo "This isn't the command you're looking for. I don't take arguments"
  fi
}

function qr() {
  WHO=${1:-$ME}
  echo qstat for \"$WHO\"
  OUT=`qstat -Jtu $WHO | tail -n +6 | awk '{print $10}' | sort | uniq -c`
  if [ -n "$OUT" ]; then
      echo "$OUT"
  else
      echo No jobs currently running.
  fi
}
```

## Stampede/SLURM

```
ME="kyclark"
alias qs='squeue -u $ME | column -t'
```
