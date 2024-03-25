ACC - Advanced Computing Center : Exacloud Spack
================================================

[Spack](https://spack.io/) is a package management system designed for high performance computing clusters like Exacloud. ACC uses Spack to install common scientific software packages in Exacloud. Users can use spack to load various packages for their workflows.

The Spack system is intended for advanced users. New Exacloud users may want to try the [Exacloud Modules](13-Modules.md) system first.

### Basic Use


The `spack` command is available to all users upon logging in to Exacloud. Common use patterns are handled by the following sub-commands:

-   `find`
-   `load`
-   `unload`

These are introduced below. You can see the [spack command documentation](https://spack.readthedocs.io/en/latest/command_index.html) for full details on its use. Note that many functions are not available within Exacloud.

#### Locating Packages

To find installed packages, use the `spack find -x` command:
``` sh
$ spack find -x
-- linux-centos7-ivybridge / gcc@8.3.1 --------------------------
afni@20.1.01                          openmpi@3.1.6               py-toml@0.10.2
amber@18                              openmpi@3.1.6               py-toml@0.10.2
```


The above will produce an overwhelming volume of output, but can be useful if you want to get a sense for everything that is available. Information is organized at two levels:

1.  First by target architecture, which is the combination of the OS, CPU, and compiler version. The default for Exacloud is **linux-centos7-ivybridge / gcc@8.3.1**. This should be used in most cases to ensure compatibility across all nodes in the cluster.
2.  Second by package name and version.

Spack will install multiple versions of the same package, sometimes even the same version more than once. This will be due to differences in how the packages are configured.

To find available versions of a particular package:
``` sh
$ spack find -x python
==> 13 installed packages
-- linux-centos7-ivybridge / gcc@8.3.1 --------------------------
python@2.7.16  python@3.4.10  python@3.6.8  python@3.8.2   python@3.9.13
python@2.7.16  python@3.5.7   python@3.7.6  python@3.8.11  python@3.10.4
```


#### Loading Packages

To load a package, use the `spack load `command:

``` sh
$ spack load r@4.2.1
$ Rscript --version
Rscript (R) version 4.2.1 (2022-06-23)
```

You can then proceed with running your software. Changes to your shell environment on the headnode are automatically imported into your jobs, so after activating modules, jobs started with `srun` will have the modules loaded as well.

If more than one package matches the given spec, you'll see output like this:

``` sh
$ spack load python@3.8.2
==> Error: python@3.8.2 matches multiple packages.
  Matching packages:
    r2gdxd6 python@3.8.2%gcc@10.2.0 arch=linux-centos7-ivybridge
    2sveel3 python@3.8.2%gcc@8.3.1 arch=linux-centos7-ivybridge
  Use a more specific spec (e.g., prepend '/' to the hash).
```
In that case you'll want to select one of the two given package hashes (the first column), and load it like so by adding a slash "/" in front of the package hash:

``` sh
$ spack load /2sveel3
$ python3 --version
Python 3.8.2
```

#### Listing Loaded Packages

To list loaded packages, use the `--loaded` flag with the `spack find` command:
``` sh
$ spack find --loaded
==> 13 loaded packages
-- linux-centos7-ivybridge / gcc@8.3.1 --------------------------
expat@2.4.8    libffi@3.4.2   libxml2@2.9.12  sqlite@3.37.2           zlib@1.2.12
gettext@0.21   libiconv@1.16  openssl@1.1.1n  tar@1.34
libbsd@0.11.5  libmd@1.0.4    python@3.10.4   util-linux-uuid@2.37.4
```
#### Unloading Packages

To remove a package from your environment, use the `spack unload` command:

``` sh
$ spack unload r@4.2.1
```
### Requesting Packages

If there is a package or a version that would be useful but is not available in Spack, send a request to <acc@ohsu.edu> to open a ticket. If possible we will install that package. Generally speaking, packages installed via spack will also be available in the [Exacloud Modules](13-Modules.md) system.

### Advanced Usage

If you have some packages you'll know you always want loaded, you can add them to your ~/.bashrc to have them loaded at each login to Exacloud:

``` sh
spack load python@3.10.4
spack load cuda@10.2.89
spack load cudnn@7.6-10.2
```

If you have different runtime requirements for different jobs, you can either manage your environment at runtime before invoking `srun`, or you can add the appropriate module load commands to your sbatch script

``` sh
#!/bin/bash
#SBATCH --partition exacloud
#SBATCH -c 2
#SBATCH --mem 10G
#SBATCH --time 4:00:00

spack load R@3.6.3

srun Rscript /path/to/your/file
```

Then each job can have its own requirements specified in the script.

### Modules


Spack provides Environment Module files for each package it installs. See the [Exacloud Modules documentation](https://wiki.ohsu.edu/display/ACC/Exacloud%3A+Modules#Exacloud:Modules-Spack) for a brief example of how to load Spack packages as modules.

### Custom Installs


The Exacloud Spack environment is hosted in an area which is read-only for users. This means users cannot make use of commands which make changes, including `install`, `uninstall`, `env create`, and similar. It is possible for users to install and run their own Spack instance on gscratch. Please note that Spack is fairly storage intensive in terms of capacity and file counts, so installing Spack may require additional quota allocation. See the [Spack Getting Started documentation](https://spack.readthedocs.io/en/latest/getting_started.html) for guidance.

Spack provides the option to "chain" instances together. This allows a local copy of Spack to make use of our existing Spack instance, re-using its existing packages and avoiding duplicating space and install operations. This is done by specifying the main Spack in the file `~/.spack/upstreams.yaml`:

``` yaml
upstreams:
  home-exacloud-software:
    install_tree: /home/exacloud/software/spack/opt/spack
```
This may be automatically discovered by the Spack setup process.

See the [Chaining Spack documentation](https://spack.readthedocs.io/en/latest/chain.html) for further guidance.

ACC will be able to offer only limited support to custom Spack instances, but don't hesitate to contact us with questions.