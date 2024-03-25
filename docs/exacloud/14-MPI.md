ACC - Advanced Computing Center : Exacloud: MPI
===============================================


MPI ("Message Passing Interface") is a technology which enables high performance computing applications to utilize multiple servers as a part of a single job. This can provide enhanced throughput for very compute-intensive jobs. Applications must be written and compiled with support for MPI in order to utilize this technology.

!!! note
    Exacloud includes support for running MPI-enabled applications across multiple nodes.


### OpenMPI

Environment settings specific to the cluster are provided by `/etc/profile.d/openmpi.sh`. If you have standard shell dotfiles, you should receive these by default upon login.

ACC recommends the use of OpenMPI 4 or newer, built with UCX support. You can activate that environment module:

``` sh
$ module load openmpi/4.1.1-ucx
```

Once you do the above, you should be able to run applications which support this version, and can compile new MPI applications.

See the [Exacloud Modules](13-Modules.md) guide for more information on working with modules.

### Custom MPI

If you have need to utilize your own compiled MPI, set your ``LD_LIBRARY_PATH``, ``PATH``, and ``MPI_HOME`` environment variables to reflect the installed location, and be sure to also set the following.

``` sh
unset OMPI_MCA_btl\
export OMPI_MCA_pml="ucx"\
export OMPI_MCA_btl_tcp_if_exclude="lo,docker0,virbr0"
export OMPI_MCA_oob_tcp_if_include="172.20.12.0/22,172.20.11.0/24"
```

For your custom-compiled openmpi to work with slurm, it must be configured with the `--with-pmi` flag.

### Running MPI jobs with slurm

slurm will automatically handle OpenMPI setup on MPI-enabled executables. In other words, there is no need to use `mpirun` to invoke jobs.

For example, to start a job with 160 tasks:

``` sh
$ srun -n 160 --mem-per-cpu=10G /home/exacloud/gscratch/example/mpiprogram

```

This will spawn 160 tasks. The scheduler will handle distributing those tasks across available resources on multiple nodes. The same arguments can be used for ``salloc`` and ``sbatch`` invocations.

### Advanced Use


In most cases you should just specify the number of tasks and the memory required per CPU and let the scheduler handle the rest. If finer-grained control is required, the relevant slurm options for MPI jobs are:

-   `-n, --ntasks`: This sets the number of MPI processes to start.
-   `--mem-per-cpu`: In MPI jobs you'll typically want to use this rather than ``--mem``, which is per node.
-   `--ntasks-per-node`: Sets the maximum number of tasks to start on each node. Should be used with ``-N``.
-   `-N, --nodes:` Sets the minimum number of nodes to allocate for the job.
-   `-c, --cpus-per-task`: This is how many CPU cores to reserve for each task (``-n``) specified. This should typically be omitted, unless your application is multi-threaded in addition to supporting MPI.

In a multi-threaded MPI application, you could start 160 threads by starting 160 single-threaded tasks, or for example by starting 10 tasks with 16 threads each. There will be varying performance characteristics based on these choices.