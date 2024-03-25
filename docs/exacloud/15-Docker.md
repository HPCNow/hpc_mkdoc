ACC - Advanced Computing Center : Docker
==================================================

!!! note
    **ACC highly recommends the use of [Singularity ](07-Singularity.md)for running containerized workflows. Singularity can work with docker containers. An older version of Docker is provided for convenience, but the necessary wrapper (see below) will prevent some workflows from working as advertized.**

Docker is available on exacloud compute nodes to facilitate the use or creation of custom environments which are not easily recreated within the cluster's node configuration.

!!! warning "Import"
    **Docker is not available on the exacloud head nodes.** 

### Access to Docker on exacloud

Access to Docker is not included by default with new exacloud accounts. Contact <acc@ohsu.edu> to request access to this resource. Please include a brief description of your planned use of Docker.

### Docker wrapper

Users cannot directly invoke the `docker` command in exacloud. Instead use `sudo` with the `/opt/acc/sbin/exadocker` script.

exadocker accepts the same arguments as the docker command. So for example:

```
docker run ubuntu whoami

```

becomes:

```
sudo /opt/acc/sbin/exadocker run ubuntu whoami

```

!!! warning "Import"
    **The `exadocker` script is not installed on the head nodes, only on the compute nodes. You'll need to submit a job via slurm, or launch an interactive session to a compute node, to use it.**

Existing docker pipeline scripts can be updated to work with the new environment by simply replacing `docker` with `sudo /opt/acc/sbin/exadocker`.

### Docker best practices

It is best to wrap docker pipelines in a shell script, either in an `sbatch` script, or submitting a pipeline shell script with `srun`.

Do not leave docker containers running after your work completes. If you are starting a container using the docker `start` sub-command, please ensure that your pipeline script stops and deletes that container at the end.

If you are invoking a one-off command using the docker `run` sub-command, include the parameter `--rm=true` to ensure that the container created from that run command is removed after execution is complete.

!!! note
    **ACC will periodically check for abandoned containers and stop and delete them without notice.**

### Mounting Docker volumes

Docker has the capability to mount filesystems inside containers, known as the "volumes" feature (see the docker-run man page for more details). When attempting to mount a shared filesystem like RDS or gscratch, you may see an error like this:

!!! failure ""
    **/usr/bin/docker-current: Error response from daemon: mkdir /home/groups/foolab/test: permission denied.**

This is due to the docker process on nodes not being able to see inside these shared filesystems. In order to make this work, you need to set -v flag to the filesystem mountpoint.

For gscratch this is the same for all users:

```
-v /home/exacloud/gscratch:/home/exacloud/gscratch
```
For RDS, this depends on your group's path:
```
-v /home/groups/MyLab:/home/groups/MyLab
```

Then within your container the files in the mounted volume will be available at the same paths as what you are used to in

The `exadocker` wrapper script sets the UID and GID of commands run in containers to match the UID and current GID of the user who submitted the job. In some cases, your default GID will not be permitted access to a shared filesystem location. In this case you'll need to [change your UNIX group](http://fshead1:8080/ACC/Changing-Your-Unix-Group-ID_22053174.html) prior to submitting the job to match the path you are mounting. Note that this does not allow you to set groups of which you are not a member.