ACC - Advanced Computing Center : Exacloud: Job Scheduler
=========================================================

Exacloud uses the **slurm** job scheduler. There are different slurm partitions for jobs with different requirements and characteristics.

The installed version is `21.08.8`, and full documentation is published on the [slurm website](https://slurm.schedmd.com/documentation.html), as well as in man pages on the Exacloud headnodes, e.g.:

```
$ man srun
```

### Basic Use
The simplest way to execute a job on a compute node is to prefix the desired command with srun, e.g.:

```
srun whoami

doejohn
```

That's it! There is a lot more detail than that, which is detailed below. srun is the command you'll most frequently use when developing a new execution pipeline. When it comes time to run your pipeline in a more robust and repeatable fashion, you'll want to switch to batch processing.

#### Running Longer Jobs
By default, jobs are limited to 1 hour. You can request longer *run-time* using the `--time` option, e.g.:

```
srun --time 1-0
```
The above requests 1 day of runtime. See the time limits section below for more information on requesting longer run-times.

#### Requesting Resources

By default your job submissions will have 1 CPU and 4 GB of RAM made available to them. These limits are enforced by the scheduler, so if your application requires more of any compute resource, you'll need to request it.

##### CPU
To request additional CPUs, use the `-c` or `--cpus-per-task` options, e.g.:

```
srun -c 4 /path/to/example/application --threads=4
```
!!! note
    Adding more CPUs to your job does not automatically increase the speed of your application. See the Parallel Computation guide for a primer on how applications can make use of multiple CPUs.

##### Memory

If your application needs more memory (aka RAM), your job will be killed with a message like this:

```
srun: error: exanode-2-45: task 0: Out Of Memory
```
In that case, you'll want to increase the amount of memory requested for your job using the `--mem` option, e.g.:

```
srun --mem 8G /your/application
```

The default unit for the memory option is MB. You can specify higher quantities by using the suffix G (for GB) or even T (for TB). When you are first running a new application, start with a relatively high memory request until you know what the job really needs. Then you can right-size the request, which will help your jobs run faster through the scheduler.

##### Scratch space

Local scratch space is a disk storage system present in compute nodes. It is useful because it provides much lower latency access than other storage options. The down-side is that since the space is unique on each node, your data must be loaded and cleaned up on a per-job basis.

The local scratch is found at the `/mnt/scratch path`. It is requested as a Slurm "generic resource" via the `--gres` flag, e.g.:

```
srun --gres disk:1024 /path/to/command
```

The unit specified for disk is in GB. The above requests 1TB of scratch space.

To request scratch space on a node with **SSD scratch drives**, use the slurm constraint feature `-C`:

```
srun --gres disk:1024 -C ssdscratch /path/to/command
```

##### GPU
See the [Exacloud GPU guide]() for information on getting started with GPUs in the cluster.


!!! question "¿Failed jobs?" 
    When something goes wrong if you use Exacloud for anything more than trivial computation, you may encounter errors. Please see the Exacloud troubleshooting guide for a starting point. If you still need help, you can contact ACC by sending an email to [acc@ohsu.edu](mailto:acc@ohso.edu).


***This includes the basic introduction to the use of the Slurm job scheduler in Exacloud. See below for more advanced scheduler topics, or consult the rest of the Exacloud User Guide for more on other topics.***


### Interactive sessions

Interactive sessions provide a means to develop and debug pipelines as well as use high-resource post-processing tools (such as image viewers), or any other activity which cannot be easily scripted. Slurm has a few options for interactive sessions. In all cases, interactive jobs should use the interactive partition.

!!! note
    Unlike batch submissions, interactive sessions are not resilient. If your shell is interrupted, or if there is a service interruption on the head node, you may lose work. It is recommended at the very least to use a terminal multiplexer like screen or tmux as a wrapper around interactive sessions.

#### Remote shell

If the above salloc workflow cannot work for some reason, the belew srun invocation can be used to get an approximation of a logon shell on a compute node:

```
$ srun --pty /usr/bin/bash -i

```

#### srun
For any one-off commands, a simple srun followed by the executible provides the simplest way to get interactive commands. Slurm will run the requested command on a compute node, and will stream the output back to your console session.

```
$ srun -c 4 pipeline.sh
```

#### salloc
The salloc command provides you with a persistent allocation of requested resources on a single node. This can be particularly useful for developing a multi-stage pipeline which makes use of node local filesystems like /mnt/scratch or Docker containers.

```
salloc -c 2 --mem=12G
salloc: Granted job allocation 1083
```
The above command puts you into a sub-shell, but you are still running on the headnode. However, any commands prefixed with srun are actually run on the allocated compute node:

```
$ srun stepa.sh
$ srun ls /mnt/scratch/user/foo
$ nano stepa.sh
$ srun stepa.sh
```

Once you are done with the allocation, simply exit:

```
$ exit
exit
salloc: Relinquishing job allocation 1083
```

And you'll exit the subshell. Jobs started using salloc have the same resource and runtime rules as jobs submitted as batches.

### Batch Processing
Once you have developed a working pipeline, you'll want to submit it as a batch job using the sbatch command. Once submitted, the job will wait in queue until the scheduler is ready to start it using available resources. Unlike interactive jobs, you are free to close your session and log out after submitting a batch job. The output of the submission will be sent to a log file.

In order to submit an sbatch job, you'll need to create a submit script. This is a shell script which tells the scheduler how to run the job.

The sbatch command accepts most of the same command line arguments as srun. It also has also has a system for encoding those options in the submission file itself. This is done by specifying the options after #SBATCH in the header of the script, one-per-line.

Below is an example sbatch script:

``` title="example.sh"
#!/bin/bash
#SBATCH -p exacloud
#SBATCH -c 8
#SBATCH --mem=96G
#SBATCH --time=6:00:00
```
```
srun ./example_application
```
Cope the above contents into a file accessible in Exacloud and customize to meet your needs. For example you would change the partition `(-p)` from "exacloud" to "guest" if you wanted to run in the guest partition.

Then submit the batch job:
```
$ sbatch example.sh
```

### Partitions
Exacloud has a number of different partitions in which jobs can run. Partitions have different hardware characteristics and intended uses. It is important to select the appropriate partition for your job.

Specify a partition by passing the `-p` option to your srun, sbatch, or salloc command:

```
$ srun -p exacloud /path/to/script.sh
```
Environment variables can be used to set default partitions for different types of partitions, e.g.:

``` title="~/.bashrc"
# Default srun partition
export SBATCH_PARTITION="interactive"

# Default sbatch partition
export SBATCH_PARTITION="exacloud"

# Default salloc partition
export SALLOC_PARTITION="guest"
```

Below is a description of the different partitions available in the cluster.

#### interactive
The interactive partition is the default partition and is intended to be used for light-weight interactive work.
=== "Details Partition"
    * Nodes: 16 nodes
    * Default Time: 1 hour
    * Time Limit: 36 hours
    * Limits: 12 CPU per user (across all jobs in partition)

#### exacloud
The exacloud partition should be used for batch submissions, high-resource jobs, and multi-node jobs.
=== "Datail Partition"
    * Nodes: All (except GPU)
    * Default Time: 1 hour
    * Time Limit: 36 hours (can be extended via QOS)
    * Available QOS: normal, highio, long_jobs, very_long_jobs
    !!! note
        MPI jobs should use the the exacloud partition. See the MPI documentation for more details.

#### gpu
The gpu partition contains nodes which have GPUs installed. GPUs are a co-processor which can provide significant speed-up to certain applications which are written to take advantage of them. Please see the GPU guide for more information.
=== "Details Partition"
    * Nodes: GPU nodes
    * Default Time: 1 day
    * Time Limit: 14 days (can be extended via QOS)
    * Available QOS: gpu, gpu_long_jobs

#### guest
The guest partition is made up of nodes owned by individual labs. You are free to submit jobs to this partition to make use of these nodes when they are available. If a job is submitted by the lab which owns the node you are running on, your job will be killed immediately. Only use this partition if you have a system for check-pointing work, or if you are OK with jobs being killed with no notice.
=== "Details Partition"
    * Nodes: Various lab-owned systems, including some GPUs
    * Default Time: 1 hour
    * Time Limit: 36 hours (cannot be extended)
    * Available QOS: normal, highio

#### basic
See the [Basic Access](01-basic-access.md) documentation for more details on this partition.

### Scheduling Policy
Exacloud's instance of slurm is not a first-come, first-served system. There are several factors which affect how soon jobs will start, and the policy and implementation of these is described below. Note that this only comes into play when there is contention on the cluster. When the cluster is not highly utilized, jobs will typically start immediately upon submission.

***The policy for the scheduler is set by the Exacloud Steering Committee.***

#### Accounting
Exacloud implements the slurm accounting system. In slurm parlance, an *"account"* is a collection of users to which cluster usage is attributed. It does not refer to user accounts. See the slurm priority documentation for more information.

Each slurm account equates to an Exacloud project. A project is typically designated for the lab of a P.I., but there are also projects for departments or other programs. Users may belong to multiple projects.

The slurm accounting system serves a two-fold purpose in Exacloud:

* It allows for accurate tracking of resource usage on a per-project basis.
* It is used to calculate the "fair-share" priority factor for jobs in queue (see Priority below).

Jobs can be counted against a specified account by adding the `-A` flag to `srun`, `sbatch`, and `salloc` invocations, e.g.:

```
$ srun -A CurieLab --time=2:00:00 ...
```
!!! note
    It is important that all jobs are assigned to their correct accounts. If no -A option is specified, the job will be charged to your user's default account, which may or may not be correct. It is a best practice to always specify -A on job invocations.

To see your default account, use the sacctmgr command, for example if your username is frank:

```
$ sacctmgr show user frank
	  User   Def Acct     Admin
---------- ---------- ---------
	 frank   franklab      None
```
To see which accounts your user is associated with, run the sshare command:

```
$ sshare -U -u frank
			 Account       User  RawShares  NormShares    RawUsage  EffectvUsage  FairShare
-------------------- ---------- ---------- ----------- ----------- ------------- ----------
franklab                  frank     parent    0.077689          60      0.252222   0.637586
CurieLab                  frank     parent    0.015936         120      0.334220   0.054619
```
To change your default account, for example to change it to CurieLab:
```
$ sacctmgr modify user frank set defaultaccount=CurieLab
```

To see the full details of all users in an account, e.g. the CurieLab account:
```
$ sshare -a | grep CurieLab
  curieLab                           8    0.015936         120      0.334220   0.054619
  CurieLab            sally     parent    0.015936           0      0.334220   0.054619
  CurieLab              jim     parent    0.015936           0      0.334220   0.054619
  CurieLab            frank     parent    0.015936         120      0.334220   0.054619
  CurieLab          lincoln     parent    0.015936           0      0.334220   0.054619
  CurieLab            marie     parent    0.015936           0      0.334220   0.054619
  CurieLab           pierre     parent    0.015936           0      0.334220   0.054619
  CurieLab            dolly     parent    0.015936           0      0.334220   0.054619
  CurieLab             mary     parent    0.015936           0      0.334220   0.054619
```
To see the total usage for an account, use `sshare -A <accountname>`.

```
$ sshare -A CurieLab
			 Account       User  RawShares  NormShares    RawUsage  EffectvUsage  FairShare
-------------------- ---------- ---------- ----------- ----------- ------------- ----------
CurieLab                                 8    0.015936         120      0.334220   0.054619
```
See the sshare manpage for full usage details.


**Here is what the columns signify:**


| Column             | Description                          |
| :------------------| :----------------------------------- |
| `RawShares`        | the count of shares for each account. This is set to the number of users in the corresponding project. For individual users, this is set to "parent" which means all users in an account are the same.  |
| `NormShares`       | is the portion of theRawShares for the account of the total for the cluster. |
| `RawUsage`         |calculation of recent usage of the cluster by the account. The number is calculated by this formula: RawUsage = (CPUs * seconds) + (20 * GPUs * seconds). This number decays with a half-life of 7 days.|
| `EffectvUsage`     | RawUsage of this account as a portion of the total for the cluster. This number decays with a half life of 7 days. |
| `FairShare`        | inversely proportional to EffectvUsage (see slurm documentation for details). This number is multiplied by a coefficient to become the FairShare factor of your job priority. |
 
If there are any issues with how accounting is configured for your user, or if your user needs to be added to additional accounts, please contact ACC.
                
#### Priority
Slurm is configured to take a number of factors into account when prioritizing the queue. Most significant of these the recent usage of the user's project, also known as "fair-share."

As described in the slurm documentation:

>The fair-share component to a job's priority influences the order in which a user's queued jobs are scheduled to run based on the portion of the computing resources they have been allocated and the resources their jobs have already consumed.

When there is contention, your jobs may have to wait longer to start than those of projects which have not used the cluster as much recently. The next most significant factor in priority is how long the job has been waiting in queue. When there is contention, leaving your jobs in queue will increase your priority over time.

To get a summary of the priorities of jobs, use the sprio command:

```
$ sprio 
      JOBID   PRIORITY        AGE  FAIRSHARE    JOBSIZE  PARTITION
   10058174      83348      25130      57085        134       1000
   10058304     105628      24940      79561        128       1000
   10058390      50685      24812      15679        194      10000
   10058453      50539      24714      15679        146      10000
```

The higher the number in the PRIORITY column, the sooner the job will run. The PRIORITY column is a sum of the subsequent four columns. The other columns:


| Column        | Description |
| :-----        | :---------- |
| `AGE`         | increases the longer a job is waiting in queue.|
| `FAIRSHARE`   | reflects recent usage of the cluster. It is inversely proportional to recent usage - the top user will receive 0, and a new user will have a very high score. See Accounting section above.|
| `JOBSIZE`     | increases the more resources requested by a job.|
| `PARTITION`   | increases based on the job being submitted to certain partaitions.|

 
To see the relative priority of jobs in queue, run the following (if you aren't using the exacloud partition, change that to your partition name):

```
$ squeue -hp exacloud -t PD --format="%Q %A %a %u %C %m %l %q" | sort -n
```
This will provided a list of jobs sorted by priority (highest at the bottom).

Slurm provides an estimated start time for some jobs in the queue:

```
$ squeue -u <username> --start
```

Start times will often be sooner, because the start time is estimated to be after running jobs use their complete run-time. Most jobs finish before that.

If you want an interactive session and do not want to wait in the queue, see the interactive or basic partitions as options.

#### Backfill Scheduler

Slurm also make use of backfill scheduling, which allows smaller, shorter jobs to start before higher-priority jobs, if the smaller job would not cause the larger job any delay in starting.

For example, consider the following two jobs:

!!! example
    ```
    Job 144: 24 CPUs, 128 GB Ram, 36-hour runtime, priority score of 50000
    Job 145: 1 CPU, 4GB RAM, 1-hour runtime, priority score of 1000
    ```
In this scenario, there will not be any nodes with 24 cores and 128 GB RAM to run Job 144 for 3 hours. However there is a node with 1 CPU, and 4GB RAM available. Since Job 145's maximum runtime (1 hour) is less than the minimum wait time for Job 144 (3 hours), the back-fill scheduler will start Job 144 first, even though Job 145 has a higher priority.

Jobs which are optimized for low resource consumption and short runtimes will therefore move more efficiently through the cluster.

#### Submission Limits

In order to prevent resource exhaustion on the head nodes, slurm is configured to accept a maximum of 50,000 job submissions. This includes any jobs visible in squeue as well as recently-completed jobs.

Additionally, in order to reduce the likelihood of a single active project consuming all of the submission slots, a limit of 5,000 submissions has been imposed on each Exacloud project. After the limit is reached, further submissions to the queue for that project will be rejected like so:

```
$ sbatch myjob.sh
sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits) 
```

To get an idea of how many of these slots you have taken, run the following command, replacing "project" with the name of your project:

```
$ sacct -n -a -A project -S $(date --date '-300 seconds' +"%H:%M:%S") | wc -l
```

#### Time Limits

Each job has a time limit. At the conclusion of the time limit, the job will be killed and not put back into queue. You'll see a notification like the following:

```
srun: Force Terminated job 1075
srun: Job step aborted: Waiting up to 32 seconds for job step to finish.
slurmstepd: error: *** STEP 1075.0 ON exanode-2-8 CANCELLED AT 2017-06-19T10:05:17 DUE TO TIME LIMIT ***
srun: error: exanode-2-8: task 0: Terminated
```

The time limit should be specified in your `srun`, `salloc`, or `sbatch` invocations with the `--time` parameter. The format is "Days-Hours:Minutes:Seconds". For example:

```
$ srun --time=1-0       # One day
$ salloc --time=6       # 6 minutes
$ salloc --time=6:30:00 # 6 hours, 30 minutes
$ srun                  # Uses the default
```

If omitted, the job is assigned the default time for that partition. In most cases the default is 1 hour. The default maximum runtime is 36 hours, but that can be extended by the use of the slurm QOS feature. For example, to request 7 days of run-time in the exacloud partition, or 20 days in the GPU partition:

```
$ srun --qos long_jobs --time=7-0
$ srun --qos gpu_long_jobs --time=20-0
```

See below for additional details on the QOS feature.

If your jobs are hitting their time limits, you should ask for more time using the `--time` argument. The execution time of jobs can vary based on several factors, the most significant of which is shared storage contention (RDS or gscratch). When deploying a new pipeline or running a one-off job, it is recommend that you request a longer run-time. Then you can check the run times of completed jobs and adjust down your time request as appropriate for subsequent runs. Asking for too long will cause your job to wait longer in queue (shorter, low-resource jobs are eligible for "backfill" scheduling while larger jobs wait). If you find that your jobs still timeout no matter how long the run-time, there is another issue we'd need to identify and address. Please contact ACC if that is the case.

#### QOS
Slurm's QOS feature can be used to adjust limits on resources. This is typically done to increase one limit (e.g. run time) in exchange for decreasing another limit (e.g. number of concurrent jobs). The QOS feature provides the same functionality as the former `highio`, `long_jobs`, `very_long_jobs`, and `gpu_long_jobs` partitions.

QOS is invoked by adding the `--qos` flag followed by the QOS name to `srun`, `salloc` ,`sbatch` invocations:

```
$ srun --time=1-0 --qos highio
```

You can view the details of QOS policies by running:
```
$ sacctmgr show qos format=Name,GrpJobs,MaxWall,MaxTRES,MaxTRESPerUser
```

A brief description of the relevant fields:

 Column             | Description                                                                                 |
| :-----            | :----------                                                                                 |
| `GrpJobs`         | The maximum count of concurrently running jobs with this QOS, regardless of user or project.|
| `MaxWall`         | The maximum run time for each job.                                                          |
| `MaxTRESS`        | The maximum CPU and/or RAM per job.                                                         |


**Below is a summary of the available QOS policies. Not all QOS policies can be used with all partitions (see below).**

=== "highio"
    * QOS: highio
    * GrpJobs: 90
    * Description: ACC may direct you to use this QOS for jobs with significant impact on storage resources.

=== "long_jobs"
    * QOS: long_jobs
    * GrpJobs: 100
    * MaxWall: 10 days
    * Description: For running long jobs.

=== "very_long_jobs"
    * QOS: very_long_jobs
    * GrpJobs: 50
    * MaxWall: 30 days
    * MaxTRES: 24 CPUs
    * Description: For running exceptionally long jobs.
=== "gpu_long_jobs"
    * QOS: gpu_long_jobs
    * GrpJobs: 6
    * MaxWall: 35 days
    * Description: For running long gpu jobs.

