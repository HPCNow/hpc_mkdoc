ACC - Advanced Computing Center : cryo-EM
==================================================

The Exacloud cluster and other ACC services provide a platform for the processing and storage of cryo-EM data at OHSU. This page details the software, computational hardware, and storage options, as well as providing some recommendations, for the cryo-EM community.

### Software

#### Relion

[Relion](https://relion.readthedocs.io/) is a software suite for cryo-EM structure determination. It supports both CPU and [GPU](09-GPU.md) computing. Its CPU computing can be scaled out to multiple nodes using the [MPI](14-MPI.md) technology.

To load Relion version 4.0-beta, run the following command:

**Activating Relion**
``` sh
spack load /eq3k6hk
```
The above can be added to sbatch scripts to ensure the environment is configured correctly for batch runs. This version of Relion is compiled with both MPI (OpenMPI 4.1.1 with UCX fabric support) and GPU (CUDA-11.4) support.

Over time, ACC may add additional Relion builds to spack. These can be located with:

And then can be loaded by using the seven character short-code before each version, e.g. for relion 3.0.8:

#### CryoSPARC


[CryoSPARC](https://cryosparc.com/) is a web-based software suite for cryo-EM. This software requires persistent web and databases services which cannot be run as a batch-scheduled job and therefore require dedicated hosting resources. Additionally, the data security model of CryoSPARC requires that each lab have its own instance.

ACC provides hosting of Exacloud-adjacent CryoSPARC instances on a fee-for-service basis. The system runs in a virtual machine and has access to your data in gscratch and/or RDS, and can submit jobs on behalf of your group to run on Exacloud compute nodes. Please contact <acc@ohsu.edu> to get started with setting up a CryoSPARC instance in Exacloud.

#### Other Software

Other cryo-EM software can be installed and run in the Exacloud environment. If a tool could be useful to multiple labs, ACC may install it in a central location. Otherwise labs can install software in their own storage locations. For assistance with installing or running software in Exacloud, contact <acc@ohsu.edu>.

### Computation

cryo-EM software makes use of a few different modes of computation. The two most important are GPU computation and MPI computation, which are described in brief below. Basic single-node CPU jobs do not require any further comment.

#### GPU


Some cryo-EM software is written with support for GPU acceleration. This can provide significant speed-ups for certain tasks. Exacloud has a [GPU](09-GPU.md) partition which is available for this purpose. ACC-provided installations of Relion and CryoSPARC have GPU support by default. Other software may also support GPUs, but may require special configuration at compile time to enable it. Also note that most software requires some run-time configuration to tell the application to make use of GPUs.

#### MPI

Relion has support for running as an [MPI](14-MPI.md) job. MPI jobs can scale beyond a single node, utilizing the combined CPU and RAM of multiple systems in a single large job. ACC-provided installations of Relion have MPI support by default.

When running Relion MPI jobs, it is necessary to match the `--j` argument (threads per task) matches the slurm argument `--cpus-per-task`. For example, as an sbatch script:

``` sh title="Relion MPI"
#!/bin/bash

#SBATCH --mem 120G
#SBATCH --ntasks 12
#SBATCH --cpus-per-task 4
#SBATCH --time=1-6

unset OMPI_MCA_btl
export OMPI_MCA_pml="ucx"
spack load /eq3k6hk

srun relion_refine_mpi --j 4 ...
```

The above will start a relion_refine_mpi job with 120GB of RAM per node, 12 total tasks, and 4 threads per task, for a total of 48 concurrent processes.

### Storage


There are three primary classes of storage available within Exacloud: RDS, gscratch, and local scratch. [Object storage](http://fshead1:8080/ACC/ACC-Object-Storage_22053368.html) is also available, though not commonly used.

Here is a generally recommend storage usage pattern:

1.  After acquisition, raw data is copied directly to gscratch
2.  Compute jobs read the raw data from gscratch, and use the same location for intermediate and final products
3.  Final products are copied to a permanent home on RDS
4.  Once the raw data is no longer needed for active computation, it can be archived to RDS, or deleted

See below for a brief description of each of the options.

#### RDS


[**Research Data Storage**](https://www.ohsu.edu/advanced-computing-center/acc-and-research-data-storage-rds) is an NFS mounted filesystem which is available throughout the Exacloud environment. RDS paths typically start with the path `/home/groups/YourLab/`. RDS is a high-capacity storage solution and is designed as "warm" storage and is best suited as the permanent home for data. Raw data and final products are ideally stored in RDS. It is not optimal for computational jobs to use data stored on RDS directly. This is because RDS does not have the performance characteristics to support high volume and parallel access. RDS suffers significant performance issues when operating over many small files.

#### gscratch


**gscratch** (mounted at `/home/exacloud/gscratch`) is a Lustre filesystem and is designed as a temporary scratch workspace for data that is undergoing active computation. It is designed as a "hot" storage space best suited as a temporary data store for raw data, intermediate products, and final products of computational pipelines. gscratch is ideal for parallel workloads, whether for singular multi-node jobs, or running many jobs on the same data in parallel. Multi-node jobs (e.g. MPI) must use gscratch for storage, since node local scratch is not available to all nodes in a job. gscratch is not ideal for data comprised of many small files, but works best with a smaller number of larger files.

#### Local scratch

**Local scratch** (mounted at `/mnt/scratch`) is a local filesystem of direct attached disk on each node. It provides the lowest latency option for data access. Single-node jobs which work with many small files can see a significant performance improvement by using local scratch rather than a network storage solution (RDS or gscratch). For example, if a job needed to work on the contents of a tar file, you can first copy the file to local scratch, extract it, do the necessary computation, copy out the products, and then delete the small files. Local scratch must be populated and cleaned for each compute job, so in certain cases the overhead of copying data in an out may outweigh its latency benefits. See the Exacloud Storage [section on `/mnt/scratch` ](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Storage#Exacloud:Storage-/mnt/scratch)for more information on how to use it.