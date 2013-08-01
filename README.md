batchqueue - a set of scripts for working with batch queues
-----------------------------------------------------------

This repo contains several useful scripts for working with batch queues.
The files are layout as follows:

- `bin/`: generic scripts for working with batch queues
- `slrum/`: scripts for working with SLRUM batch queues
- `cac/`: scripts for working with Cornell's CAC batch queue

## jobqueue

The main script provided by this repo is `bin/jobqueue`. I solves the
following problems:

- The native job queue does not fairly distribute work amongst users,
  e.g. it queues everything first-come first-serve. This limitation
  causes users to only queue a subset of their jobs at a time in order
  to not hog the job queue. `jobqueue` solves this by monitoring the
  native queue and only submitting your jobs if you currently have
  less than a specified number of jobs running.

- The cluster provides multiple cores per node, but the native queue does
  not bundle jobs to fully maximize each node. This places the burden on the
  user to bundle their jobs. `jobqueue` solves this by allowing one to
  specify a group name for each job. Jobs of the same group name will
  automatically be bundled together and submitted as one native job.

### Getting started

To get started with `jobqueue` run the script with no arguments.

```
jobqueue
```

This will create a directory `$HOME/jobs/` which will store all job
information.  You will be asked to edit a configuration file stored in
`$HOME/jobs/config/jobqueue.json`. This file defines how `jobqueue` should
interact with the native queuing system.

`jobqueue` assumes your native queuing system provides a command that
works like LSF's `bjobs` command or SLRUM's `sbatch`
command. Specifically, it assumes the queuing command can take a job
script from stdin and that allow queue specific commands can be
handled either in stdin or on the command line. For example, to queue
a job that lists the files in the current directory, we can do the
following:

```
jobqueue -g mygroup -- sbatch -n 16 -t 100 <<EOF
ls
EOF
```

This creates a job with a group `mygroup`. `sbatch` is the native queue
command and it is told to use the used to allocate 16 processor per node
and 100 minutes for the job. The actual job is specified in the
[here-document](http://en.wikipedia.org/wiki/Here_document).

To actually submit this job along with many others to the native queue,
simply run the job queue with the following command

```
jobqueue --run
```

This will run the queue until the user kills it with control-C. `jobqueue`
will monitor the directory `jobs` every minute and bundle and submit jobs.
