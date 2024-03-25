ACC - Advanced Computing Center : Exacloud: Singularity
=======================================================


SingularityCE** is a container platform which was designed for use in High Performance Computing (HPC) environments like Exacloud. It provides a portable and reproducible means to package and run software. It is similar to [Docker](http://fshead1:8080/ACC/21896493.html) and can run unmodified Docker containers directly. However, Singularity has a number of advantages over Docker due to being designed for HPC. Singularity is not used for running persistent services in the way that Docker was originally designed for. Exacloud is a batch processing cluster and persistent services cannot be run in the cluster.

Documentation for using Singularity in Exacloud in particular is found below. Please consult the [SingularityCE User Guide](https://sylabs.io/guides/3.8/user-guide/) for official documentation on the software in general.

### Installation Location


Singularity is installed on compute nodes at `/opt/singularity/current` . 

!!! note
    Singularity is not installed on the Exacloud head nodes nor exalogin. It must be used via [the scheduler](http://fshead1:8080/ACC/22053376.html).

A convenient way to set up your environment for Singularity is to use the module command:
```
$ module load /etc/modulefiles/singularity/current
```

Alternatively you can manually manage your own environment to add the appropriate directories to your `PATH` and `MANPATH` variables, e.g. in `~/.bashrc`:
```
export PATH="/opt/singularity/current/bin:$PATH" \
export MANPATH="/opt/singularity/current/share/man:$MANPATH"
```

Once either of the above is completed, you will be able to run singularity commands directly.

### Cache

Certain Singularity operations will result in files being downloaded to a local cache directory. By default, this directory is located in the user's home directory at `~/.singularity/cache` . This can quickly use up your account's home directory quota. Therefore it is recommended to place the cache in a different location by setting the `SINGULARITY_CACHEDIR` variable in your shell environment. You'll want to specify a directory with sufficient space and which is unique to your user account (cache directories cannot be shared with multiple users, even if they all have filesystem permissions). ACC recommends the use of gscratch space for this purpose. To set `SINGULARITY_CACHEDIR`, one can add the following to `~/.bashrc` :

```
export SINGULARITY_CACHEDIR=/home/exacloud/gscratch/MyLab/username/cache
```

Singularity can summarize and clean the cache using the cache sub-command. See the Singularity documentation on [cache list](https://sylabs.io/guides/3.8/user-guide/cli/singularity_cache_list.html) and [cache clean](https://sylabs.io/guides/3.8/user-guide/cli/singularity_cache_clean.html) operations.

### Images

In Singularity, images are files which encapsulate containers. You must have an image file to run Singularity containers. You can obtain image files from either the [SyLabs Cloud Library](https://cloud.sylabs.io/library) or [Docker Hub](https://hub.docker.com/). Keep in mind that image files are downloaded to your current directory by default, so be mindful of quotas.

To obtain an image from SyLabs, simply use the pull sub-command with the `library://` URI
```
$ singularity pull library://ubuntu
```
To create an image file from a Docker container, specify the image file name before the `docker://` URI:
```
$ singularity pull ubuntu_latest.sif docker://ubuntu
```

In either case you will have a .`sif` image file (e.g. `ubuntu-latest.sif` or similar) which can then be used for Singularity container operations.

### Running containers with slurm


Singularity is not installed on the Exacloud headnodes and must be run on compute nodes via [the job scheduler](http://fshead1:8080/ACC/22053376.html).

For downloading images and doing development work. ACC recommends an interactive session, either via `salloc` or starting a shell directly with `srun`:

```
$ srun -p --pty /usr/bin/bash -i

user@exanode-0-0 ~ $ singularity pull ...
```
For running established container workflows, and especially for long-running jobs, ACC recommends the use of an `sbatch `script, e.g. create example.sh:
```
#!/bin/bash
#SBATCH --partition exacloud
#SBATCH -c 1
#SBATCH --mem 4G
#SBATCH --time 4:00:00
#SBATCH --job-name container

module load /etc/modulefiles/singularity/current

srun singularity run -B /home/exacloud/gscratch/[MyLab pipeline.sif](http://mylab/data%20pipeline.sif)
```

And then submit via sbatch:

```
$ sbatch example.sh
```
### Container Operations

There are three basic Singularity sub-commands for using containers:

-   **[shell](https://sylabs.io/guides/3.8/user-guide/cli/singularity_shell.html)** - opens a shell within the container
-   **[exec](https://sylabs.io/guides/3.8/user-guide/cli/singularity_exec.html)** - runs the specified command within the container
-   **[run](https://sylabs.io/guides/3.8/user-guide/cli/singularity_run.html)** - runs the specified runscript within the container, if it exists

A very concise example of **exec** uses the `ubuntu_latest.sif` image file (see above):
```
$ singularity exec ubuntu_latest.sif whoami

username
```

Opening a shell can provide a good interactive workflow:

```
$ singularity shell -B /home/exacloud/gscratch/[MyLab:/home/exacloud/gscratch/MyLab:ro](http://acc/data/ACC:ro) ubuntu_latest.sif
Singularity> cd /home/exacloud/gscratch/MyLab
Singularity> ls
...
Singularity> exit`
```
The availability and use of run will vary based on the container you are using. Singularity provides an amusing example:
```
$ singularity run lolcow.sif
```

### Filesystems


By default a Singularity container only mounts your user home directory, and there is no access to other filesystems on the compute node. If your container needs access to data in other locations (including gscratch, RDS, or /mnt/scratch), that needs to be specified when you start a container.

To that end, Singularity's run, exec, and shell sub-commands have the `-B` or `--bind` option, which allows you to specify a path to add to your container:
```
$ singularity exec -B /home/exacloud/gscratch/MyLab ubuntu_latest.sif yourcommand
```
An RDS mount can be made available within your container using the same arguments:
```
$ singularity exec -B /home/groups/MyLab ubuntu_latest.sif yourcommand
```
Singularity can bind to a different path inside the container if desired (e.g. mounting `/home/exacloud/gscratch/MyLab` as `/data/MyLab` inside the container):
```
$ singularity exec -B /home/exacloud/gscratch/[MyLab:/data/MyLab](http://acc/data/ACC) ubuntu_latest.sif yourcommand /data/MyLab
```
Additionally, you can specify that the mount point is read-only within the container by adding :ro to the bind argument. This is useful for ensuring a container only reads and cannot change data.
```
$ singularity exec -B /home/exacloud/gscratch/[MyLab:/home/exacloud/gscratch/MyLab:ro](http://acc/data/ACC) ubuntu_latest.sif yourcommand /home/exacloudgscratch/MyLab
```
Multiple bind arguments can be specified by use of a comma:

```
$ singularity exec -B /home/exacloud/gscratch/[MyLab,](http://acc/data/ACC:ro,/home/exacloud/software)/mnt/scratch/username ubuntu_latest.sif yourcommand
```
Remember that these options come after the sub-command (shell, exec, or run) but before the image file. See [Exacloud storage documentation](http://fshead1:8080/ACC/22053392.html) for more information about these storage locations and how to request `/mnt/scratch` space in jobs.

### Building Images

At present it is not possible for Exacloud users to build new Singularity images. This is due to security considerations. ACC plans on adding a secure container build host to allow this capability in the future. Singularity can run existing containers in both the Singularity image and Docker formats.