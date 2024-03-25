ACC - Advanced Computing Center : R kernel in jupyter notebook
========================================================================


First, load a spack module with R:

``` sh
spack load /o3avrxs\
This loads R@4.2.2 with the irkernel package installed
```
Next, load the most current version of python:

``` sh
module load python

python -m venv myEnv\
```
You can name your environment anything you would like, for this example we are calling it myEnv.

``` sh
source myEnv/bin/activate

(myEnv) $ python -m pip install jupyter\
Start an R session:

(myEnv) $ R

> IRkernel::installspec(name = 'ir422', displayname = 'R 4.2.2')
```

That will create an R kernel and make it available when you start an exajupyter session.

Assuming you want to install some R packages to use on exajupyter, you will need to let R know where you want to install packages:
``` sh
(myEnv) $ jupyter kernelspec list
Available kernels:
  ir422 /home/users/perrymil/.local/share/jupyter/kernels/ir422\
```

Lists available kernels and the path to the directory with your kernel.json file. You can edit that file like so:
``` sh
vi /home/users/perrymil/.local/share/jupyter/kernels/ir422/kernel.json\
```
Note, your username should replace "perrymil" in that path, and whatever you named your R kernel will need to match as well.

The kernel.json file allows you to append environment variables such as:
``` json
{ 
  "argv": ["/home/exacloud/software/spack/opt/spack/linux-centos7-ivybridge/gcc-8.3.1/r-4.2.2-o3avrxsie2rfjqjzpbg33ifuyvh6i3hq/rlib/R/bin/R", "--slave", "-e", "IRkernel::main()", "--args", "{connection_file}"],
  "display_name": "R 4.2.2",
  "language": "R",
  "env": {"R_LIBS":"/home/exacloud/gscratch/ACC/perrymil/Rlibs"},
  "env": {"R_LIBS_USER":"/home/exacloud/gscratch/ACC/perrymil/Rlibs"}
}
```
Yours won't have the "env": lines until you add them in.

After adding those "env": lines in, the next time you start this kernel in jupyter the R kernel will source those environment variables and know to install packages in the path you set R_LIBS to. Now you can install.packages('packageName').

When starting a jupyter notebook, select the "New" button and select your new R kernel from the list.

You probably need r-devtools as well for installing R packages.

``` sh
$ spack load /ln4v6sm

$ R

> library(devtools)
```

#### example below to install a package from github
``` sh
install_github("miccec/yaGST", lib="/home/exacloud/gscratch/ACC/perrymil/Rlibs")\
```
As long as you set lib= to the same path that you have it set in your R kernel.json env statement, the installed packages will be available in your R environment on exajupyter