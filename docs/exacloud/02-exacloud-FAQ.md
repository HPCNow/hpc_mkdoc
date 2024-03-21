ACC - Advanced Computing Center : Exacloud FAQ
===============================================

### How can I obtain an exacloud account?

See the [Exacloud Introduction](21896490.html) document.

### How can I find software on exacloud?

If you cannot find an application in your default path, please check the /opt/installed directory tree; it contains a wide assortment of applications and libraries requested by the OHSU user community. If what you are looking for is not present, but you think it may be useful to the wider exacloud user community, contact ACC to request the software to be added. Otherwise software can be installed in and run from project directories or user home directories.

### Why was my job submission rejected?

See the Troubleshooting "[Why wasn't my job accepted?](https://wiki.ohsu.edu/display/ACC/Exacloud+Troubleshooting#ExacloudTroubleshooting-not-accepted)" documentation.

### Why isn't my job running?

See the Troubleshooting "[Why isn't my job running?](https://wiki.ohsu.edu/display/ACC/Exacloud+Troubleshooting#ExacloudTroubleshooting-not-running)" documentation.

### Why did my job fail?

See the Troubleshooting "[Why did my job fail?](https://wiki.ohsu.edu/display/ACC/Exacloud+Troubleshooting#ExacloudTroubleshooting-fail)" documentation.

### How do I see resource usage of running or completed jobs?

Slurm has different tools for getting stats from jobs, depending on whether they are currently running or completed.

#### Running Jobs

For running jobs, use `sstat`:

```
sstat -j 2379069 --format "JobID,MaxVMSize"

JobID        MaxVMSize  AveCPU
------------ ---------- ----------
2379069.0    398432K    00:31.000
```    

This displays the job ID and the maximum memory size used thus far. Run `sstat -e` to see a full list of parameters. See the `sstat` manual page for more details.

#### Completed Jobs

For complete jobs, use `sacct`:

```
sacct -j 1000000 --format "JobID,Elapsed,ReqCPUs,CPUTime,ReqMem,MaxVMSize"

JobID          Elapsed   ReqCPUS    CPUTime     ReqMem  MaxVMSize
------------ ---------- -------- ---------- ---------- ----------
1000000        00:01:37        1   00:01:37       10Gn
1000000.bat+   00:01:37        1   00:01:37       10Gn    626872K
```
  

This reports on how long the job took, how many CPUs were requested, how much CPU time was used, the requested memory, and the memory high-water mark. Use this information to adjust resource requests for subsequent jobs. Requesting just a bit above required resources for a job will ensure optimal queue placement. See the `sacct` manual page for more details.

Another useful tool for obtaining stats from completed jobs is the job profiler script. This script will print human-readable job resource stats from slurm accounting database.

to see recommendations for a particular jobs, use the -j argument to pass the job number into the profiler

```
/usr/local/bin/jobprofile.py -j 12345678
``` 
```
/usr/local/bin/jobprofile.py --help
usage: jobprofile.py [-h] [--nosteps] [--csv]
                     [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}]

optional arguments:
  -h, --help            show this help message and exit
  --nosteps             Suppress individual job steps (Warning: may result in incomplete data)
  --csv                 Print extended data in csv format
  --log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}
```
!!! tip
    jobprofile also accepts sacct arguments such as `-u`, `-j`, `-A`, `-S`, and `-E`. See sacct documentation for more details.

If your jobs are being terminated due resource limitations, we would recommend resubmitting the job with higher resource request that will allow the job to run to completion. Then after the successful completion of the job, check the job profiler script to see its actual usage. By right-sizing your job submissions, you can minimize the amount of time your submission spends in the queue.

### Why can't I see the stats on commands in my sbatch?

If you find that using `sstat` or `sacct` that you cannot see stats for all of your batch job, it is likely because you have not prefixed your computational commands with **srun** in your batch file. Commands in the sbatch file which do not start with **srun** are not tracked by the job accounting system. When in doubt, use srun in your batch submission scripts.

### How can I run graphical applications on compute nodes?

See the [Graphical Applications via X11 forwarding](22053373.html) guide.