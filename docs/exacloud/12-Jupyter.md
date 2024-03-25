ACC - Advanced Computing Center : Jupyter
===================================================

There are two basic ways to run Jupyter Notebooks in Exacloud:

1.  Through Jupyter Hub
2.  Directly via slurm

Using Jupyter Hub is often simpler, but it is less customizable. Running your own Jupyter notebook directly via the scheduler requires more start-up work, but allows a lot more flexibility.

See the below sections for guidance on these two options.

### Jupyter Hub [exajupyter.ohsu.edu]

Exacloud includes a [Jupyter Hub](https://jupyter.org/) instance to facilitate the running of Jupyter notebooks in the cluster. Users connect to a single web interface, but notebooks are spawned as [Slurm jobs](05-Job-Scheduler.md) on compute nodes. This allows Jupyter Notebooks to be easily run on the cluster without users having to do complex proxy configurations to use them.

#### Getting Started


1.  Connect to [exajupyter.ohsu.edu](https://exajupyter.ohsu.edu/).
2.  Sign in with your OHSU credentials.
3.  Click the **Start My Server** button.
4.  Fill out the form with appropriate options for your [job submission](05-Job-Scheduler.md):
    1.  **Partition**: The cluster partition (i.e. the -p option)
    2.  **Number of Cores**: The number of CPU cores (i.e. the -c option)
    3.  **Memory GB**: The amount of RAM in GB (i.e. the --mem option)
    4.  **GPUs**: Number of GPUs to request, only in conjunction with gpu partition above (i.e. the --gres option)
    5.  **Job Duration**: Max run-time for the job, subject to partition limits (i.e. the --time option)
5.  Click **Spawn**. Your personal hub instance should now be running.

#### Filesystems

Your personal hub will start in your home directory. The Jupyter Hub web UI does not have an easy way to navigate to other file-systems, but this can be done via symlinks. ACC recommends creating a symlink to your lab group storage in lustre in your home directory. E.g.:

``` sh
$ ln -s /home/exacloud/lustre1/foolab ~/lustre1-foolab

```

Then you can click "lustre1-foolab" link in the Hub web UI to reach that location.

#### Custom Python kernels

Over time newer versions of python than what is included in the default exajupyter configuration (python3.8) will be released. Additionally there are not many python packages available in that environment. If you require a newer version of Python, or additional package support, it is possible to configure additional kernels on a per-user basis.

The first step is create an activate an environment (via conda or vanilla virtualenv) that contains ipykernel and whatever other packages you might like to use. Installing jupyter is a good way to pull in the necessary dependencies.

Then with the new environment activated, run the following (example is based on a virtualenv running python-3.11.4):

``` sh
$ python -m ipykernel install --user --name python3.11.4 --display-name "Python 3.11.4"

```

Then once you launch a notebook via jupyterhub, you can choose that kernel for making new notebooks, or choose **Kernel** -> **Change Kernel** -> **Python 3.11.4** to select a kernel for an existing notebook.

Here is a full worked example:

``` sh 
$ /home/exacloud/software/python/3.11.4/bin/python -m venv newenv
$ source newenv/bin/activate
(newenv) $ pip install jupyter
(newenv) $ python -m ipykernel install --user --name mypy3.11.4 --display-name "My Python 3.11.4"
Installed kernelspec mypy3.11.4 in /home/users/<user>/.local/share/jupyter/kernels/mypy3.11.4
(newenv) $ jupyter kernelspec list
Available kernels:
  mypy3.11.4      /home/users/<user>/.local/share/jupyter/kernels/mypy3.11.4

```

Then after launching a new Exajupyter session, "mypy3.11.4" is an available kernel.

#### FAQ

!!! question "What if my web session is disconnected?" 
    Your personal Jupyter Hub instance will remain running as a Slurm job and will still be registered with exajupyter. Reconnecting exajupyter should connect you to your existing session, if it has not yet timed out.

### Direct Jupyter Notebooks via slurm

It is possible to directly run Jupyter notebooks without using exajupyter as proxy. This allows:

-   Fully customized python environments
-   Up-to-date jupyter installs
-   Greater flexibility with scheduler parameters

Direct Jupyter usage is a bit more challenging in that it requires some additional set-up for connectivity - namely there must be an ssh session with port forwarding established to allow connections from your local browser to the running Jupyter session.

ACC provides a convenience script for [creating Python virutal environments](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Python+Tips#Exacloud:PythonTips-VirtualEnvironmentScript) with Jupyter installed. If for whatever reason that process won't work for you, see below for more details on how to craft one manually.

#### Template for sbatch


The following can be used as an sbatch template
``` sh
#!/bin/bash
#SBATCH --partition exacloud
#SBATCH --account MyLab
#SBATCH --cpus-per-task 2
#SBATCH --mem 10G
#SBATCH --time 8:00:00
#SBATCH --job-name jupyter-notebook

# Path to environment containing jupyter
JUPYTER_ENV=/home/groups/MyLab/venv

source $JUPYTER_ENV/bin/activate

node=$(hostname -s)
port=$(/usr/local/bin/get-open-port.py)

echo "SSH connection string:"
echo "  $ ssh ${USER}@exahead1.ohsu.edu -L ${port}:${node}:${port}"
echo
echo "Once the ssh connection is established, copy the URL printed below which"
echo "starts with http://127.0.0.1:${port}"
echo
echo "Navigate to that URL in your local browser."

srun jupyter-notebook --no-browser --port=${port} --ip=${node}
```

Save the above as a file accessible in Exacloud, e.g. "jupyter.sh". Customize the #SBATCH parameters to meet the needs of your notebook. The CPU count should be n+1, where n is the maximum number of concurrent kernels you plan to run.

Start the job by running `sbatch jupyter.sh`. Once the job starts, the output log will provide some instructions, including an example ssh command, for making a tunnel from your local system to the running notebook.

!!! warning "import"
    Please note that the example ssh command printed in the log assumes you are on campus. You will need to take additional action (e.g. adding -J user@acc.ohsu.edu) in order to connect from off-site. Please see the [ACC Remote Access](http://fshead1:8080/ACC/ACC-Remote-Access_21896647.html)documentation for more details.

#### Custom jupyter environment creation


A custom Python virtual environment for running Jupyter can be constructed like so

``` sh
$ /home/exacloud/software/python/3.10.4/bin/python3 -m venv /home/groups/MyLab/jupyter
$ source /home/groups/MyLab/jupyter/bin/activate
(jupyter) $ pip install jupyter
(jupyter) $ pip install numpy etc...

```

In the above example, /path/to/environment is the location of the new environment, which can be added as `JUPYTER_ENV=/home/groups/MyLab/venv` to the sbatch template above. There are multiple versions of Python which can be used as the base version for an environment. See the full list in `/home/exacloud/software/python`.