ACC - Advanced Computing Center : Exacloud Julia
================================================

[Julia](https://julialang.org/) is a high-level programming language with an emphasis on performance which is commonly used for numerical analysis and scientific computing. Julia code can be run as a script or in a web-based notebook interface, and has built-in support for visualization and parallelism.

For more information, see the [Julia Documentation](https://docs.julialang.org/en/v1/).

### Running Julia in Exacloud

Exacloud includes Julia in the [module system](13-Modules.md):

``` sh
$ module load julia/1.9.4
$ julia --version
julia version 1.9.4
```

It is also possible for users to install their own versions of Julia in any storage accessible in the cluster (ACC recommends gscratch for this purpose).

#### Interactive

After being loaded, Julia can be run in an interactive mode (the REPL) within an interactive session:

``` sh
$ srun --time=8:00:00 --pty /usr/bin/bash -i
user@exanode-11-24 ~ $ module load julia/1.9.4
user@exanode-11-24 ~ $ julia
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.9.4 (2023-11-14)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> sqrt(100)
10.0
```

#### Julia Scripts

Julia code can also be run in batch mode by saving the code in a script:

``` sh
$ srun -p exacloud julia myscript.jl
```

Long-running scripts and multi-step pipelines should be run with `sbatch`.

Very short scripts can be fed directly on the command-line using the `-e` argument:
``` sh
$ srun julia -E 'sqrt(pi)'
1.7724538509055159
```

#### Notebooks

##### Jupyter Notebooks

Julia code can be run inside [Jupyter Notebooks](https://jupyter-notebook.readthedocs.io/en/latest/).

To add Julia as a kernel available to Jupyter, install the **IJulia** package:
``` sh
$ srun julia -e 'using Pkg; Pkg.add("IJulia")'
```

The above will install the needed Julia packages and also create a Julia kernelspec for Jupyter to find. Now when you [run a Jupyter notebook in Exacloud](12-Jupyter.md), you can choose Julia as a kernel. This can be achieved either by using the Exajupyter interface or by running your own Jupyter environment.

##### Pluto Notebooks

There is a Julia-native notebook option called [Pluto](https://plutojl.org/). Pluto notebooks must be started via an sbatch submission, and then connected to using an SSH tunnel.

First install Pluto:
``` sh
$ srun julia -e 'import Pkg; Pkg.add("Pluto")'
```

Once Pluto is installed, create an sbatch script using the below:


``` sh title="pluto.sh"
#!/usr/bin/bash
#SBATCH -p exacloud
#SBATCH --time=8:00:00
#SBATCH --mem=64G

module load julia/1.9.4

node=$(hostname -s)
port=$(/usr/local/bin/get-open-port.py)

echo "SSH connection string:"
echo "  $ ssh ${USER}@exahead1.ohsu.edu -L ${port}:${node}:${port}"
echo
echo "Once the ssh connection is established, copy the URL printed below which"
echo "starts with http://0.0.0.0:${port}"
echo
echo "Navigate to that URL in your local browser."

srun julia -e "using Pluto; Pluto.run(launch_browser=false, host=\"0.0.0.0\", port=$port)"
```

Then submit the sbatch scrcipt:
``` sh
$ sbatch pluto.sh
Submitted batch job 23624515
```

Give the notebook at a minute to start up, and then check the output logfile:

``` sh
$ cat slurm-23624515.out
SSH connection string:
$ ssh user@exahead1.ohsu.edu -L 34655:exanode-09-20:34655
```

Once the ssh connection is established, copy the URL printed below which
starts with [http://0.0.0.0:34655]()

```
Navigate to that URL in your local browser.
[ Info: Loading...
┌ Info:
└ Go to http://0.0.0.0:34655/?secret=GLTj3hCT in your browser to start writing ~ have fun!
┌ Info:
│ Press Ctrl+C in this terminal to stop Pluto
└
```

Based on the above, take the following steps:

1.  Run the indicated SSH command on your workstation to establish a connection from your workstation to the Pluto notebook
2.  Open the link (e.g. http://0.0.0.0:34655/?secret=GLTj3hCT in this example) in your local browser

Once you are done, close the browser tab, exit the extra ssh connection, and `scancel` the job ID.

### Configuration


Julia can be configured in various ways to better adapt it to the Exacloud environment and to your needs.

-   [Command Line Interface switches](https://docs.julialang.org/en/v1/manual/command-line-interface/)
-   [Environment Variables](https://docs.julialang.org/en/v1/manual/environment-variables)

Some options of particular interest are described below:


#### Depot Path

By default, Julia saves packages in your home directory under `~/.julia`. For performance reasons, or to save space in your home quota, or to share with other users, you may want to move the location where Julia saves modules.

To do so, set the [JULIA_DEPOT_PATH](https://docs.julialang.org/en/v1/manual/environment-variables/#JULIA_DEPOT_PATH) environment variable in your `~/.bashrc` file by adding the following:

``` sh title="~/.bashrc"
export JULIA_DEPOT_PATH=/home/exacloud/gscratch/FooLab/user/julia-depot
```

Be sure to create the specified directory. ACC recommends gscratch for this use.

#### Parallel Processing


Julia has support for several modes of parallel processing including multi-threading, multi-processing, and multi-node (not compatibly with Exacloud unless using an MPI-based module). If your Julia code or a module you want to make use of are written for parallel processing via the process or threads mode, be sure to request a number of CPUs (`-c` option) equal to the number of threads if only using the `-t` flag, or equal to **(1+process)*threads** if using both the `-p` and `-t` flags.

For example, if a module called for 10 threads:

``` sh
$ srun -p exacloud -c 10 julia -t 10 myscript.jl
```


Or if code called for 4 additional worker nodes, with 5 threads each:

``` sh
$ srun -c 25 julia -p 4 -t 5 somescript.jl
```