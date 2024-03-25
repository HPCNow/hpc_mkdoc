ACC - Advanced Computing Center : Exacloud Troubleshooting
==========================================================

This document provides basic troubleshooting methods for various common issues on Exacloud. If any of the below explanations are insufficient, please contact ACC for support through [the web portal](https://service.ohsu.edu/servicedesk/customer/portal/10) or by emailing <acc@ohsu.edu>.

### Jobs

Below are common issues with submitting and running jobs in Exacloud. This section is organized based on whether the job submission was accepted, whether the job was accepted but is still waiting in the queue, or whether the job ran but encountered an error.

#### Why wasn't my job accepted?

Sometimes the scheduler will reject your job at submission, without ever inserting it into the queue. Below are some common reasons for submissions being rejected.

##### Group Not Permitted

!!! failure "srun"
    **error: Unable to allocate resources: User's group not permitted to use this partition**

The above error messakge indicates that your ACC user account is not entitled to use the partition you submitted to. If you are a Basic Access user, be sure to add the arguments `-p basic -A basic` when submitting jobs. Paid Exacloud users may see this when attempting to submit to a condo partition for which they lack authorization.

##### Invalid account or partition

srun: error: Unable to allocate resources: Invalid account or account/partition combination specified

The above indicates that you are using an invalid account, (`-A` argument). To see which accounts you have access to, run:

``` sh
$ sshare -U
```

See the [Job Scheduler Accounting documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-Accounting) for more details.

##### Unable to allocate resources

srun: error: Unable to allocate resources: Requested node configuration is not available

The above may appear after a message about memory specification or CPU count "cannot be satisfied". In all cases it means that your submission has requested resources which are greater than can be provided by available nodes. The likely candidates are CPU (`-c`), memory (`--mem`), disk (`--gres=disk:<num>`) or GPUs (`--gres=gpu:<num>`). In the case of GPUs, you need to make sure you are submitting to the GPU partition with `-p gpu`.

##### Maximum Submissions

sbatch: error: QOSMaxSubmitJobPerUserLimit\
sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)

The above indicates that you have reached the maximum allowed submissions for the account and partition combination. In most cases this is because you are a [Basic Access](http://fshead1:8080/ACC/Basic-Access_105786318.html) user who has tried to submit too many jobs. Otherwise you may have reached the cluster-wide submission limit of 5,000 jobs per user.Invalid

#### Why isn't my job running?


You can see the state of your submitted jobs by running the following (substituting your username):
``` sh
$ squeue -u username
```
Jobs which are submitted but not yet running will show PD ("pending") in the ST ("state") column. The reason for a job pending will in the final column. See below for information on each pending state.

##### Priority or Resources

If you see **(Priority)** or **(Resources)** as the reason your job is pending, that is expected. Exacloud is a batch-scheduled shared resource. When there is contention for resources, there is a priority system which sorts the queue. The primary factor in that priority score is how much your lab has been using the cluster lately (QOSMaxCpuPerUserLimit users are prioritized over heavy users), but mediated by the size of the lab (larger labs have more priority shares than smaller labs). The next most important factor is how long the job has been waiting in the queue. So the best time to submit jobs is always "now". They will wait in queue until they are scheduled, and then they will run. You can read more about the priority system in the [Job Scheduler Priority documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-Priority).

If you need an interactive session and do not want to wait in queue, the default or basic partitions may be best suited to your needs. You can read more about these options in the [Job Scheduler Partition documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-Partitions).

##### Dependency

If you see **(Dependency)** as the reason your job is pending, that is due to you having specified dependencies using the `--dependency` or -`d` arguments to slurm, but those dependencies have not yet been met. This is typically due to a job higher in the dependency graph still running. See the slurm documentation for more details on the dependency system.

##### Time Limit

If you see **(PartitionTimeLimit)** as the reason your job is pending, that is due to you having requested more time than is allowed for the partition. You need to either reduce the requested time, or use one of the time-extended QOS options. You can read more about these options in the [Job Scheduler Time Limits documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-TimeLimits).

##### QOS Limits

If you see **(QOSGrpJobsLimit)**, it is due to your selected QOS having a global limit of running jobs. This applies to the **long_jobs **(100 maximum running jobs across the whole cluster), **very_long_jobs **(30), and **gpu_long_jobs **(6) QOS options. Your job will wait until there other jobs using the same QOS complete. See the [Job Scheduler Time Limits documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-TimeLimits) for more details.

If you see **(QOSMaxJobsPerUserLimit)**, or **(QOSMaxCpuPerUserLimit)**, that is due to you submitted to a partition which has resource limits. This can happen in two ways:

-   Submitting to the default **interactive** partition while using more than 12 CPUs cluster-wide. Use the [**exacloud** partition](http://fshead1:8080/ACC/22053376.html) instead.
-   A [Basic Access](http://fshead1:8080/ACC/Basic-Access_105786318.html) user who has either met the maximum of running jobs (4), or has requested more total CPU than is allowed (36). If you need more resources, you'll need to upgrade your account.

##### Partition Node Limit

If you see **(PartitionNodeLimit)** as the reason your job is pending, that is due to you having requested more nodes (`-N` argument) than the partition allows. This will most often occur with [Basic Access](http://fshead1:8080/ACC/Basic-Access_105786318.html) users, who are limited to a single node per job.

##### Partition Config

If you see (PartitionConfig) as the reason your job is pending, this is due to you requesting more resources than can be provided in the given partition. This typically occurs due to requesting more memory or CPU per node than is available.

##### Ivalid Partition/Account Combination

If you see **(Job's QOS not permitted to use this partition (basic allows basic not normal))** as the reason your job is pending, or on submission you see "Requested partition configuration not available now", it is due to specifying an invalid combination of account and partition. This most often happens when trying to use the basic partition. Specify `-A basic` along with `-p basic` when using basic. See the [Basic Access](http://fshead1:8080/ACC/Basic-Access_105786318.html) documentation for more information.

#### Why did my job fail?
--------------------

If a job was successfully submitted and is no longer in the queue, the scheduler ran the job. However in same cases jobs that run can fail. To see the status of a completed job, run:
``` sh
$ sacct -j <jobid>
```
The State column will indicate how the job ended. The ExitCode column can also be relevant. See below for guidance on the various states of failure. See the slurm documentation for more details on the use of the `sacct` command.

##### Timeout

If the job state was **TIMEOUT**, this indicates that your job reached its requested time limit without completing. The default time-limit for jobs is **1 hour**, unless otherwise specified with the `--time` argument. Increase the requested time to allow the job to complete. See the [Job Scheduler Time Limits documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Job+Scheduler#Exacloud:JobScheduler-TimeLimits) for more information.

##### Out of Memory

If the job state was **OUT_OF_MEMORY**, this indicates that your job used more memory than it requested and was killed by the scheduler. You may see the accompanying messages in your jobs logs:

srun: error: exanode-2-45: task 0: Out Of Memory\
slurmstepd: error: Detected 1 oom-kill event(s) in StepId=20600248.0. Some of your processes may have been killed by the cgroup out-of-memory handler.

The default memory request is **4 GB**.  You can specify more using the `--mem` argument, e.g. to request 64 GB, add `--mem=64G` to your request. See the forthcoming Job Profiling documentation for more details on how to observe RAM usage and right-size job requests.

##### Cancelled

If the job state was **CANCELLED**, it indicates that the job was stopped before it could complete execution on its own. A job can be cancelled either when it is in queue, or after it has already started running.

Cancellation can be initiated by the user in two basic ways:

-   Using the `scancel` command.
-   By using shell signals in an `srun` command, most commonly by typing Control-c twice in quick succession.

Jobs can also be cancelled by administrative action, in which case it should be accompanied by an email notification from ACC. If you are uncertain why your job was cancelled, contact ACC.

##### Failed

If the job state was **FAILED**, it indicates that job was started by the scheduler, but could not complete successfully. In most cases this indicates that the application experienced an error and stopped with a non-zero exit code. If the software you are using publishes documentation about the meaning of its exit codes, you can observe the ExitCode column, take the number before the ":", and look up the meaning of that code. Otherwise you should consult the output logs to see if the application indicated why the execution failed.