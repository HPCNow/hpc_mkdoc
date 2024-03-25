ACC - Advanced Computing Center : Parallel Computation
======================================================

There are a number of different ways to achieve parallel computation using the slurm scheduler in Exacloud. The best choice will depend on the software and the nature of the computation for any given workload.


### Multi-Threading

A single application process can start multiple sub-processes known as "threads." These threads can run in parallel in order to speed up computation. This is a common technique for parallel computation. The application must be written by its authors to support multi-threading. Simply requesting a larger number of CPUs will not enable multi-threading.

It is important to match the number of threads started by the application with the number of CPUs requested from the scheduler (using the **-c** or **--cpus-per-task** options). Most applications will have an option for specifying the number of threads which should be started. See your application's documentation for details. For example, if we want to run a job with 8 threads, and the application has an option **--threads** to specify that number, the sbatch script would look like this:

**Multi-threaded jobs**
```
#!/bin/bash
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=2G
srun /path/to/your/application --threads=8z
```

The scheduler constrains your job's threads to the number of CPU cores you request, so starting more threads than the number of requested CPUs will not result in any faster computation. In fact, it will cause the application to slow down due to the overhead of over-subscribing the CPUs. Sometimes an application may support threads but not offer a configuration option for controlling them. In most cases, such an application will set the number of threads to the number of CPU cores detected on the node where it is running. If that is the case, it is best to submit the job using the slurm **--exclusive** flag, which will ensure that the job is the only one running on a given node.

### Message Passing
Message passing is a technique that allows multiple application processes to be started, and then for data to be shared between them using a defined protocol. Since this message passing protocol can run over a network, it allows for jobs to scale beyond the resource constraints of a single node. A message passing library must be used by your application's developers to enable this option.

The most prominent message passing system is [MPI](https://www.mpi-forum.org/docs/), though there are others like [OpenMP](https://www.openmp.org/). With an MPI application, you specify the number of concurrent tasks with the **-n** or **--ntasks** option. For example, to start a job with 80 concurrent worker processes:

**MPI sbatch**
```
#!/bin/bash
#SBATCH --ntasks=80
#SBATCH --mem-per-cpu=10G

srun /path/to/your/mpi/application
```

The scheduler will handle allocating resources and starting the job, even if the number of tasks requires more than one node. See the [Exacloud MPI guide](http://fshead1:8080/ACC/22053384.html) for additional guidance on using MPI applications in Exacloud.

### Scheduler Techniques


When the application itself does not support parallel computation, the slurm scheduler can be used to parallelize computation. This is sometimes referred to as "embarrassingly parallel", but there's nothing to be ashamed of!

#### Multiple Submissions with Arguments


The most basic form of this is simply submitting multiple sbatch jobs to the queue at once, which is fairly self-explanatory. These jobs will be started by the scheduler according to the queue, and depending on cluster conditions, this can lead to many of your tasks running in parallel.

You can re-use the same sbatch script for multiple runs by passing arguments to your sbatch script, which can then be passed as arguments to your application. For example:

**sbatch arguments**
```
#!/bin/bash
#SBATCH --cpus-per-task=1
#SBATCH --mem=16G
#SBATCH --time=12:00:00

srun /path/to/your/pipeline.py --input=$1
```
In the above, **$1** refers to the first argument passed to the sbatch script. Then the above can be submitted like so:

**Submit sbatch with arguments**
```
$ sbatch myscript.sh /path/to/your/samples/1.tar
```
Multiple submissions can be made manually, changing the argument(s) for each. With a large number of submissions, you may want to use a script or shell loop to help with submission.

For example, looping over files in a directory:

**Loop of files**
```
$ for id in /path/to/your/samples/*; do echo sbatch myscript.sh $id; done
```
Or if your script takes a sequence of integer inputs:

**Sequence of integers**
```
$ for id in $(seq 1 505); do echo sbatch yourscript.sh $id; done
```
!!! note
    In the above examples, I have included an **echo** command so that the commands which would be run would be printed first. After running that and checking it for sanity, remove the **echo** to have it run for real.

If the parallel work requires more complex orchestration than the above, see below for some techniques.

#### Multiple Submissions with Dependencies

As mentioned above, you can always just submit multiple sbatch jobs independently. But suppose that your workflow has some tasks which need to be run serially after other parallel tasks have completed. You could simple watch the queue and wait to submit the next job until after the prior required jobs complete, but this is not an efficient use of your time or the cluster. Fortunately the scheduler has a dependency system which allows many jobs to be submitted at once, but to only run when certain conditions are met.

The slurm sbatch command provides the option **-d** or **--dependency** to specify the conditions for when the submitted job can start. The most basic form is to specify that a submission can start after a given JobID has completed successfully (using the "afterok"dependency type):\
`$ sbatch -d afterok:123456`

See the [sbatch documentation](https://slurm.schedmd.com/sbatch.html) for more examples of dependency types which can be specified using the **-d** option.

The above example illustrates the concept, but would not be particularly useful in practice because it would require a lot of tedious manual submission of jobs. It is possible to create systems for submitting dependency-based pipelines by storing the JobIDs of previous submissions. The Biowulf project at NIH provides some suggestions for [building pipelines using slurm dependencies](https://hpc.nih.gov/docs/job_dependencies.html). There is also [the sdag tool](https://github.com/abdulrahmanazab/sdag) which allows defining job dependencies in a directed acyclic graph data structure. You can make your system as simple or as complicated as it needs to be.

#### Shell Background Jobs

The logon shell you use when connecting to Exacloud has the capability to start processes and then put them into the background. The background processes, called jobs, continue to run while the shell can be used for other tasks. In the context of cluster scheduling, this can allow you to start several tasks all at the same time within the same submission on the same node. This option may be right for you if you need a basic level of parallel orchestration, but don't require the complexity of the dependency system.

The code below illustrates how to use shell background jobs to achieve parallel computation in an sbatch submission:
```
#!/bin/bash
#SBATCH --ntasks=9
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G

echo "$(date): start"

for i in $(seq 10 10 90); do
  srun -n 1 --exact bash -c "sleep $i; echo Slept $i seconds" &> logs/$i-log.txt &
done

wait

echo "$(date): end"
exit
```

The above illustrates using a for loop to submit jobs.

!!! note
    -   Set the **--ntasks** option to equal the maximum number of jobs you'll be running in parallel.
    -   Each computational task must be started with **srun -n 1 --exact**. If not, the tasks will not run in parallel as desired.
    -   Instruct the shell to background a task by adding **& **at the end of the line. If you don't, each task will be run serially.
    -   Use the shell keyword **wait** to cause the script to pause until all background jobs have completed. Otherwise the script will complete immediately and the background jobs will be terminated.
    -   (Optional) You may want to redirect each tasks output into a different log file, e.g. `&> logs/$i-log.txt` above.