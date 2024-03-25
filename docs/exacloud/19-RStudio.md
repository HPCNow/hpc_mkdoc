ACC - Advanced Computing Center : Exacloud RStudio
==================================================

[RStudio Server](https://posit.co/products/open-source/rstudio-server/) is a browser-based development environment for the R programming language. It provides a graphical user interface for the interactive writing and testing R code, but running in a server environment rather than on the developer's workstation.

In the Exacloud cluster, RStudio Server can provide two basic benefits for R coders:

1.  The ability to run R scripts on compute nodes with significant compute resources.
2.  Access to the Exacloud storage environment, including gscratch.

There are two basic modes for using RStudio Server in Exacloud, which are detailed below.

### exarstudio.ohsu.edu


ACC provides an instance of RStudio Server in the Exacloud environment: [https://exarstudio.ohsu.edu](https://exarstudio.ohsu.edu/). This is the quickest and easiest way to get up and running with RStudio Server in Exacloud.

To use it:

1.  Log in to [https://exarstudio.ohsu.edu](https://exarstudio.ohsu.edu/) with your OHSU username and password.
2.  You now have an R environment with access to Exacloud filesystems.
3.  When done, choose **File → Quit Session**.

!!! note
    This system is hosted on a virtual server with relatively limited resources. If you find that you need more than what is provided by exarstudio, please consult the Custom RStudio Server section below.

### Custom RStudio Server

The custom RStudio Server option allows you to start your own instance of RStudio Server on a compute node. This should be the option you choose in the following circumstances:

-   If your code requires the resources of a compute node, and cannot run on exarstudio.ohsu.edu.
-   If you need to run a newer version of RStudio Server than is provided by exarstudio.ohsu.edu.

ACC recommends the use of the [Rocker ](https://rocker-project.org/)distribution of containerized RStudio Server for use in Exacloud. In particular, we recommend the use of the [singularity ](08-Singularity.md)container version. The below instructions are adapted from [Rocker project singularity documentation](https://rocker-project.org/use/singularity.html).

#### Rocker Environment Set-Up

To set up an RStudio environment using Rocker:

1.  If you have not used singularity in Exacloud, see the [singularity guide](http://fshead1:8080/ACC/71962787.html). Be sure to configure an appropriate `SINGULARITY_CACHEDIR` in your `~/.bashrc`.
2.  Create a directory for rocker in Exacloud-accessible storage, optimally in gscratch, e.g.:\
    `$ mkdir -p /home/exacloud/gscratch/ExampleLab/exampleuser/rocker`
3.  Navigate to that new directory, e.g.:\
    `$ cd /home/exacloud/gscratch/ExampleLab/exampleuser/rocker`
4.  Active the singularity environment:\
    `$ module load singularity/current`
5.  Pull the rocker container into your current directory (this will take a few minutes):\
    `$ srun singularity pull <docker://rocker/rstudio:4.2>`
6.  Verify that the file `rstudio_4.2.sif` is present.
7.  Copy the example rocker sbtach script (see below) as `rocker.sh` in your rocker directory.
8.  Edit the `rocker.sh` script to customize its contents as follows:
    1.  Modify and/or add SBATCH headers as appropriate. See slurm [sbatch documentation](https://slurm.schedmd.com/sbatch.html) and the [Job Scheduler guide](05-Job-Scheduler.md).
    2.  Modify the **workdir **variable to point to your rocker directory, e.g.:\
        `workdir=/home/exacloud/gscratch/ExampleLab/exampleuser/rocker`
    3.  Modify the **user_paths **variable to include any filesystems you'll need to access from RStudio. Entries to the list are separated by commas, e.g.:\
        `user_paths="/home/exacloud/gscratch/ExampleLab,/home/groups/ExampleLab"`

You now have a Rocker environment set-up. The above actions should only need to be performed once, unless you want to update the version of RStudio.

### Example rocker sbatch script


``` sh title="rocker.sh"
#!/bin/bash
#SBATCH --time=08:00:00
#SBATCH --signal=USR2
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=8192

# Create temporary directory to be populated with directories to bind-mount in the container
# where writable file systems are necessary. Adjust path as appropriate for your computing environment.
workdir=/home/exacloud/gscratch/ExampleLab/exampleuser/rocker

# Comma-separated list of paths to have mounted in your container
# e.g. user_paths="/home/exacloud/gscratch,/home/groups/ExampleLab"
user_paths="/home/exacloud/gscratch"

mkdir -p -m 700 ${workdir}/run ${workdir}/tmp ${workdir}/var/lib/rstudio-server
cat > ${workdir}/database.conf <<END
provider=sqlite
directory=/var/lib/rstudio-server
END

# Set OMP_NUM_THREADS and R_LIBS_USER

cat > ${workdir}/rsession.sh <<END
#!/bin/sh
export OMP_NUM_THREADS=${SLURM_JOB_CPUS_PER_NODE}
export R_LIBS_USER=${HOME}/R/rocker-rstudio/4.2
exec /usr/lib/rstudio-server/bin/rsession "\${@}"
END

chmod +x ${workdir}/rsession.sh

export SINGULARITY_BIND="${user_paths},${workdir}/run:/run,${workdir}/tmp:/tmp,${workdir}/database.conf:/etc/rstudio/database.conf,${workdir}/rsession.sh:/etc/rstudio/rsession.sh,${workdir}/var/lib/rstudio-server:/var/lib/rstudio-server"

export SINGULARITYENV_RSTUDIO_SESSION_TIMEOUT=0
export SINGULARITYENV_USER=$(id -un)
export SINGULARITYENV_PASSWORD=$(openssl rand -base64 15)
readonly PORT=$(/usr/local/bin/get-open-port.py)

cat 1>&2 <<END
```

1. SSH tunnel from your workstation using the following command:
``` sh
ssh -N -L 8787:${HOSTNAME}:${PORT} ${SINGULARITYENV_USER}@exahead1
```
!!! warning ""
    and point your web browser to [http://localhost:8787]()

2. log in to RStudio Server using the following credentials:
``` sh
   user: ${SINGULARITYENV_USER}
   password: ${SINGULARITYENV_PASSWORD}
```
When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:
``` sh
      scancel ${SLURM_JOB_ID}
END
```

``` sh
singularity exec --cleanenv rstudio_4.2.sif\
    /usr/lib/rstudio-server/bin/rserver --www-port ${PORT}\
            --auth-none=0\
            --auth-pam-helper-path=pam-helper\
            --auth-stay-signed-in=0\
            --auth-timeout-minutes=5\
            --rsession-path=/etc/rstudio/rsession.sh\
            --server-user=${SINGULARITYENV_USER}
printf 'rserver exited' 1>&2
```
### Running Rocker


To start your RStudio session, submit the `rocker.sh` file using sbatch:\
`$ sbatch rocker.sh`

Make note of the Job ID printed when the sbatch command completes. Once your job begins running, it will produce output in its log file like so:
``` sh
$ cat slurm-22012921.out
```
1. SSH tunnel from your workstation using the following command:
``` sh
ssh -N -L 8787:exanode-2-44:36181 exampleuser@exahead1
```
!!! warning ""
    and point your web browser to [http://localhost:8787]()

2. log in to RStudio Server using the following credentials:

``` sh
user: exampleuser
password: examplepassword
```

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:
``` sh
scancel -f 22012921
```

Follow the instructions for setting up an SSH tunnel to your RStudio instance. If you are using an alternative SSH client like Putty on Windows, see the [Windows SSH tunneling guide](http://fshead1:8080/ACC/Windows-SSH-Tunneling_127634264.html).

Connect to your session by pointing your browser to the indicated localhost link, and log in given your username and the one-time password.

Once you are done, follow the instructions from the log file to complete your session. You can also close out of the SSH connection you created for the tunnel.

Attachments:
------------

![](http://fshead1:8080/ACC/images/icons/bullet_blue.gif) [rocker.sh](http://fshead1:8080/ACC/attachments/127634244/127634276.sh) (application/x-sh)\
![](http://fshead1:8080/ACC/images/icons/bullet_blue.gif) [rocker.sh](http://fshead1:8080/ACC/attachments/127634244/127634275.sh) (application/x-sh)