ACC - Advanced Computing Center : Exacloud: Storage
===================================================

There are a number of different filesystems available within exacloud, and it is possible to move data to and from exacloud from outside storage platforms. Each has different characteristics and serves a different purpose.

### Cluster Storage

These are provided to facilitate the processing of your data in the cluster environment. None of these storage locations should be considered the permanent home of your data. ACC can provide high-capacity storage on a fee for service basis if needed.

#### /home/exacloud/gscratch


The gscratch filesystem is our fastest network filesystem; it is available on all Exacloud compute nodes. There is an annual fee, above and beyond normal user fees, for using gscratch.

A gscratch directory will normally be created for your Exacloud project (lab group) when the lab is first granted access to the cluster. If you don't have one, send an email to [acc@ohsu.edu](mailto:acc@ohsu.edu) detailing

* The name of your lab group/project,
* How much gscratch space you'd like to purchase.

##### Quotas

The gscratch space available to your lab or project is governed by a two-part quota. One part of the quota concerns your overall space allocation; it can only be expanded by altering your ACC invoicing costs. The other part concerns the number of files your space can contain. Some projects run into this second quota well before they have hit their space quota; please contact [acc@ohsu.edu](mailto:acc@ohsu.edu) if that happens.

Aggregate utilization will be determined by project ownership of files. A hypothetical "Park Lab" would have its gscratch files located in the `/home/exacloud/gscratch/ParkLab` directory, with that directory owned and writable by a group called "ParkLab." That directory will also be assigned a numeric Lustre project ID matching the ParkLab group, and all files created within that directory will be counted with that project ID. In this manner, an individual can do work within multiple labs and have the files associated with that work count against the correct Lustre project (lab) quota.

A command-line tool for identifying your project ID's usage against quota is available for your use.

You first need to discover the numeric project ID number of your top-level gscratch directory, and then use that project ID (3001 in the example below) to discover your quota and usage:

``` sh
$ cd /home/exacloud/gscratch/MyLabDir
$ lfs project -d .
3001 P .
$ lfs quota -h -p 3001 .
Disk quotas for prj 3001 (pid 3001):
     
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
              .  254.2G      0k     10T       - 1040581       0 1500000       -
$
```
(The `-h` switch is optional but results in output you may find easier to process quickly.)

!!! tip
    Please consult the lfs(1) man page for further information and other available options.

!!! tip "Data Backup Options"
    The gscratch file system is intended as scratch space for compute jobs. It is not backed up by ACC. Users can use any backup method which uses SSH for transport (e.g. rsync, scp, etc.) to copy important data through the head-node and on to other storage, such as ACC Bulk or RDS storage, or other storage which complies with OHSU security requirements dictated by the data classification in question.

### /mnt/scratch

`/mnt/scratch` is not a shared filesystem; it is particular to each node. It is intended to provide low-latency storage for data in the course of a single pipeline run. Your job should load data into this location at the beginning of each run, and clean up after the job completes (regardless of job success).

!!! note
    This location should be considered volatile, and data abandoned there will be deleted periodically without warning.


In order to utilize `/mnt/scratch`, you **must** request it as a generic resource (using the `--gres` flag) in your job request, e.g.:

``` sh
srun -p exacloud --gres disk:1024 /path/to/command

```

The unit of disk is GB. The above example requests 1TB of scratch space. The scheduler ensures that your job will not run on a node where space is not available. There is scratch space available in all slurm partitions. However some nodes do not have scratch space. Requesting the disk gres will ensure that your job is placed on a node with local `/mnt/scratch`.

#### Set-up and Tear-down

ACC provides two scripts, `/usr/local/bin/mkdir-scratch.sh` and `/usr/local/bin/rmdir-scratch.sh` to assist with creating and removing local scratch jobs within slurm jobs. The mkdir script creates a directory in `/mnt/scratch/N`, where N is the slurm job ID. The following provides a template for how these scripts can be used within an sbatch submission:

``` sh title="/mnt/scratch set-up" 
srun /usr/local/bin/mkdir-scratch.sh
SCRATCH_PATH="/mnt/scratch/${SLURM_JOB_ID}"

srun your_program_here --temp-dir $SCRATCH_PATH

srun /usr/local/bin/rmdir-scratch.sh
```

!!! warning "import"
    1.  Make sure that you prefix your application invocation with `srun`. This will ensure that it is tracked as a job step, and the failure of your application won't prevent the `rmdir-scratch.sh` script from running.
    2.  Be sure to include the `rmdir-scratch.sh` script as the final line of the sbatch script, and to prefix it with `srun` so that it runs even if   previous steps have failed.
    3.  If desired you can change the working directory of the sbatch into the scratch directory with e.g. `cd $SCRATCH_PATH` or `pushd $SCRATCH_PATH`. Be sure to back out of that path (e.g. using `cd -` or `popd`) before running the `rmdir-scratch.sh` script. This is only recommended on single-node jobs.

#### SSD scratch

On many compute nodes, `/mnt/scratch` storage is based on traditional hard drives. It will perform especially well for single-stream sequential reads, but poorly for random I/O. There is a subset of nodes which have a scratch space backed by SSDs.

To request scratch space on a node with SSDs, add the argument `-C ssdscratch` to your srun command, e.g.:

``` sh
srun -p exacloud -C ssdscratch --gres disk:8192 /path/to/command

```

`/mnt/scratch` is the correct path on both types of nodes.

### Other Storage


The below are not particular to exacloud. The users and groups shares are directly mounted on compute nodes, but the rest require data to be moved through another system (such as your workstation).Any ssh-compatible data-moving application (such as rsync, sftp, scp, or the putty versions of these) can be used to copy data into exacloud filesystems from your workstation.  Here are some more details on how some of these compare with each other: [ACC: Storage Services Comparison](https://wiki.ohsu.edu/display/ACC/ACC%3A+Storage+Services+Comparison)

#### /home/users


`/home/users` provides ACC home directory access in exacloud. It is available to provide shell profiles and store scripts, but should not be used for any data processing in the cluster.

#### /home/groups


`/home/groups` contains mount points for NFS shares hosted by ACC, including [Research Data Storage](https://www.ohsu.edu/advanced-computing-center/acc-and-research-data-storage-rds). If you have such a share and would benefit from direct mount access to that location in exacloud, contact ACC and we can arrange to have it mounted in the cluster. It is still advisable in most cases to copy data from these locations to gscrcatch before processing, and to move the results back after processing is completed. In particular, it is a best practice to avoid having multiple jobs access the same files and directories concurrently on RDS.

!!! info
    RDS may be necessary for applications which depend on POSIX locking. Such applications (like sqlite) are not supported on gscratch.

#### Globus

ACC operates a Globus data transfer node. We can provide POSIX storage gateways for RDS and Lustre, allowing you to transfer data in and out of the Exacloud environment to other Globus endpoints. Please contact ACC for more details.

#### H: and X: Drives

Neither the `H:\` nor `X:\` drive are directly available within exacloud. Data can be moved between these locations and exacloud using a workstation. However, it is important to note that neither `H:\` nor `X:\` are intended as a storage location for research data.