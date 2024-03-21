ACC - Advanced Computing Center : Basic Access
==============================================

The Exacloud Basic Access service allows any paid ACC account holder to access a limited cluster computing environment within Exacloud.  This Basic Access service contrasts with the long-standing Exacloud Standard Access service, which provides project- or lab-based access to the entire Exacloud compute cluster, at a higher per-user fee.

If you are new to Exacloud, please have a look at the Exacloud documentation here:  [Exacloud User Guide](Exacloud-User-Guide_21896659.html)

If you do not already have a paid ACC account, or if you need assistance beyond what you have found in these documents, please contact the ACC Service Desk by sending email to acc@ohsu.edu, or by entering a ticket here (on-campus or VPN):  [https://service.ohsu.edu/servicedesk/customer/portal/10](https://service.ohsu.edu/servicedesk/customer/portal/10)

### Obtaining Access

Existing paid Exacloud Standard Access users already have basic access granted. New basic-only users may need to run the following setup command to get registered in the cluster scheduler's database:

``` sh title="ssh connection"
ssh exahead1.ohsu.edu  
  . . .  
sudo /usr/local/bin/basic-access.sh

```
### Usage

To submit jobs to the basic partition, specify both the basic partition (`-p basic`) and account (`-A basic`) in your srun/sbatch/salloc invocation, e.g.

``` sh
srun -p basic -A basic ...
```

### Resources

!!! note
    If any of the above constraints prove to be obstacles, please contact ACC about upgrading to Exacloud Standard Access.

The basic partition includes the following compute resources:

*   22 compute nodes, each with 36 CPUs and 256 GB of RAM
*   2 GPU nodes, each with 4 P100 GPUs, 44 CPUs, and 512 GB of RAM

### Resource Limits

Basic partition resource allocations are limited to the following:

*   4 running jobs per user
*   8 total submissions per user
*   36 CPU (across all jobs)
*   512 GB RAM (across all jobs)
*   4 GPUs (across all jobs)
*   1 node per job
*   24 hours maximum runtime