ACC - Advanced Computing Center : Modules
===================================================


Exacloud provides the [Environment Modules](https://modules.readthedocs.io/en/latest/) system for providing access to installed software. The `module` command automates the process of configuring the user's shell environment to add and remove software packages. Environment Modules allow you to load different versions of packages at runtime, which can help with maintaining different execution environments for different packages.

### Basic Use


The basic use of the `module` command is made up primarily of the following sub-commands:

-   `avail`
-   `load`
-   `list`
-   `rm`

These are introduced below. See the full [module command documentation](https://modules.readthedocs.io/en/latest/module.html) to go beyond the basics.

#### Listing available Modules

To see what modules are available, run the `module avail` command:

```
$ module avail

------------------------------------ /home/exacloud/software/modules ------------------------------------
### etc.
python/3.10.4                       python/3.7.6                            python/3.8.2
R/3.5.3                             R/3.6.3                                 R/4.0.2
### etc.
```

Exacloud-provided modules are named in the format software/version. You can have available modules from multiple sources. If there is a package or version that is missing that you would find useful, contact <acc@ohsu.edu> to request that it be installed. Otherwise it is possible to create personal modules for your own installed software (see **Personal Modules** below).

#### Loading Modules

To load a module, type the command `module load` followed by the module name/version:
```
$ module load python/3.10.4
$ python3 --version
Python 3.10.4
```

Above I demonstrate that after loading a module, the active version of `python3` is 3.10.4, not the Exacloud default of 3.6.8.

Changes to your shell environment on the headnode are automatically imported into your jobs, so after activating modules, jobs started with `srun` will have the modules loaded as well.

#### Listing Loaded Modules

To see a list of currently loaded module, use the module list command:

```
$ module list
Currently Loaded Modulefiles:
  1) use.own         2) python/3.10.4

#### Removing loaded Modules
``` 
To remove a module from your environment, use the `module rm` command:

```
$ module rm python/3.10.4
sminatha@exahead1 ~ $ module list
Currently Loaded Modulefiles:
  1) use.own
```


### Advanced Use

If you have some modules you'll know you always want loaded, you can add them to your ~/.bashrc to have them loaded at each login to Exacloud:

```
module load python/3.10.4
module load cuda/10.2.89
module load cudnn/7.6-10.2
```

If you have different runtime requirements for different jobs, you can either manage your environment at runtime before invoking `srun`, or you can add the appropriate module load commands to your sbatch script:

**Example sbatch**
```
#!/bin/bash
#SBATCH --partition exacloud
#SBATCH -c 2
#SBATCH --mem 10G
#SBATCH --time 4:00:00

module load R/3.6.3

srun Rscript /path/to/your/file
```
And then each job can have its own requirements specified in the script.

### Personal Modules

It is possible to create and load your own custom modules. This is useful for activating environments which are installed in your group's gscratch or RDS, or even small environments in your home directory.

First, create a directory called **privatemodules** in your home directory:
```
$ mkdir ~/privatemodules
```

Then you'll want to make a file in that directory named for the software and version. In this example, we are created a module file `MegaUltra-12.1` for the fictional software MegaUltra version 12.1, installed in the `/home/exacloud/gscratch/MyLab/MegaUltra/12.1` directory:

```
#%Module 1.0
#
#  OpenMPI module for use with 'environment-modules' package:
#
prepend-path            PATH             /home/exacloud/gscratch/MyLab/MegaUltra/12.1/bin
prepend-path            LD_LIBRARY_PATH  /home/exacloud/gscratch/MyLab/MegaUltra/12.1/lib
setenv                  MEGA_ULTRA_CACHE /home/exacloud/gscratch/MyLab/MegaUltra/cache
```

The above would add the application binary path to the user's executable **PATH**, add the lib dir to **LD_LIBRARY_PATH**, and set an environment variable **MEGA_ULTRA_CACHE**. These are common components to module files. See examples in `/home/exacloud/software/modules`, and consult the [modulefile documentation](https://modules.readthedocs.io/en/latest/modulefile.html). You can also view the contents of a loaded module using the `module show` command.

One the file is created, you can add your privatemodules directory to your list of available modules, and load your module:
```
$ module load use.own
$ module avail
---------------------------------- /home/users/sminatha/privatemodules ----------------------------------
MegaUltra-12.1
$ module load MegaUltra-12.1
$ which MegaUltra
/home/exacloud/gscratch/MyLab/MegaUltra/12.1/bin/MegaUltra
```

To have your personal modules always be available at start-up, add the following to your` ~./bashrc` file:
```
module load use.own
```

### Spack


The [Exacloud Spack](http://fshead1:8080/ACC/Exacloud-Spack_113945210.html) system provides modules for the software it installs. Advanced users may be interested in using spack's modules in addition to the default. Please note that the spack-provided module system Lmod has some different behaviors than the default "module" function.

By activating the spack module system, so you gain access to all of the packages and versions installed by spack, instead of the curated list provided by ACC in `/home/exacloud/software/modules`. The list can be quite overwhelming, but may prove useful to some users.

To activate the spack modules, run the following:
```
$ source $(spack location -i lmod)/lmod/lmod/init/bash
$ module use /home/exacloud/software/spack/share/spack/modules/linux-centos7-ivybridge
```

The above will allow you to use the module system and commands.

module spider cuda

Will output a list of all packages with cuda in their name. You can then paste in the suggested module load statement to load the module for cuda.
```
module load cuda-11.0.2-gcc-8.4.0-sf73m43
```

All of the same can be achieved natively by using the spack load command, so this is a matter of personal preference.