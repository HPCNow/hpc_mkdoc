ACC - Advanced Computing Center : Exacloud: GPU
===============================================

Exacloud contains the following GPU node configurations:

-   6 nodes (2 in GPU partition, 4 in guest) containing 8 [Nvidia A40](https://www.nvidia.com/en-us/data-center/a40/) GPUs (44GB DRAM)
-   8 nodes containing 4 [Nvidia Tesla V100](https://www.nvidia.com/en-us/data-center/tesla-v100/) GPUs (32GB DRAM)
-   8 nodes containing 4 [Nvidia Tesla P100](http://www.nvidia.com/object/tesla-p100.html) GPUs (16GB DRAM)
-   8 nodes containing 4 [Nvidia RTX 2080 Ti](https://www.nvidia.com/en-us/geforce/graphics-cards/rtx-2080-ti/#specs) GPUs (11GB DRAM)

### Access to GPU partition

The GPU nodes are not a part of the standard slurm partitions in exacloud (see the [job scheduler documentation](05-Job-Scheduler.md)for additional details). Use the **gpu** partition to access GPU nodes.

Access to the GPU partition is included with all paid Exacloud accounts.

### Guest Partition GPUs


The guest partition contains 4 systems which each include 8 Nvidia A40 GPUs. By submitting to the **guest** partition your jobs may be scheduled when these resources are not in use by their owners. However, be aware that your job can be killed without warning if the owner submits a job.

### Running GPU jobs with slurm

In addition to specifying time, CPU, and RAM requirements for a job, GPU jobs must also request cards as "generic resources" (or "gres" in slurm).

!!! note
    You must request GPUs via the `--gres` flag, or your job will not have access to a GPU card.

Run pipeline.sh in the gpu partition, requesting a single GPU:

``` sh
$ srun -p gpu --gres gpu:1 pipeline.sh

```

You can specify the model of GPU (rtx2080, p100, v100, or a40 in our environment):

``` sh
$ srun -p gpu --gres gpu:rtx2080:1 pipeline.sh
$ srun -p gpu --gres gpu:p100:1 pipeline.sh
$ srun -p gpu --gres gpu:v100:1 pipeline.sh$ srun -p gpu --gres gpu:a40:1 pipeline.sh
```

More than one GPU can be requested for a single job if necessary:

``` sh
$ srun -p gpu --gres gpu:a40:6 pipeline.sh

```

The same syntax applies to **salloc** and **sbatch**. Also remember to include time requests if you need more than the default 1 hour. The CPU and memory requirements of GPU jobs will vary based on application.

### CUDA tools


ACC recommends using the [Modules ](13-Modules.md)system for loading CUDA and related libraries. For example, to load CUDA 11.4 and cuDNN 8.2.4:

``` sh
$ module load cuda/11.4.2
$ module load cudnn/8.2.4.15-11.4
```

We may have additional CUDA installations available via [Spack](18-Spack.md).

The Exacloud head nodes and GPU nodes also have the following versions of the CUDA toolkits available:

-   **7.0**: `/usr/local/cuda-7.0/`
-   **7.5**: `/usr/local/cuda-7.5/`
-   **8.0**: `/usr/local/cuda-8.0/`
-   **9.0**: `/usr/local/cuda-9.0/`
-   **10.0**: `/usr/local/cuda-10.0/`
-   **10.2**: `/usr/local/cuda-10.2/`

The default /usr/local/cuda is a symlink to the 10.2 installation.

### How busy are the GPU's right now?

The **sinfo** command will show the count of fully allocated, idle, and mixed (partially allocated) GPU nodes.  The **squeue** command can show what jobs are running and queued (pending) on the GPU nodes:

``` sh
$ sinfo -p gpu
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
gpu          up 14-00:00:0      1  alloc exanode-8-8
gpu          up 14-00:00:0      8    mix exanode-7-[16-17],exanode-8-[5,9-13]
gpu          up 14-00:00:0     16   idle exanode-7-[18-23],exanode-8-[6-7,14-20]

$ squeue -p gpu
JOBID       PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
86024156          gpu   cmd123   user19 PD       0:00      4 (Resources)
85995240          gpu     bash   user20  R 2-21:04:41      1 exanode-7-17
85995924          gpu     bash   user20  R 2-22:25:36      1 exanode-7-16
86023927          gpu   cmd921   user20  R   22:25:53      1 exanode-8-13
86023735          gpu jupyter-   user42  R    4:19:28      1 exanode-8-5
86024221          gpu     bash   user66  R      52:41      1 exanode-8-13
86024590          gpu     bash   user14  R    1:09:46      1 exanode-8-13
86024224          gpu   cmd123   user19  R      43:00      4 exanode-8-[9-12]
```