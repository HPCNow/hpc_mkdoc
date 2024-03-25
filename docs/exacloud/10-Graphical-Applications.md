ACC - Advanced Computing Center : Exacloud: Graphical Applications
==================================================================


It is possible to run applications with a graphical user interface (GUI) on an Exacloud in two different ways.

1.  Lightweight graphical applications (like Firexfox) can be run on a purpose-bulit system **exalogin.ohsu.edu**
2.  Computationally intentse graphical applications can be run on compute nodes through the Slurm scheduler.

In either case, your workstation needs to be equipped for X11 forwarding.

### Mac OS X, Linux, etc.

X11 forwarding works out-of-the-box with the built-in ssh client on Mac OS X and Linux workstations.

Make an ssh connection with X11 forwarding enabled and then launch a graphical application with srun:

``` sh
$ ssh -Y exahead1.ohsu.edu
$ srun -t 3:00:00 /path/to/gui/application
```

Or if you just want to run Firefox:
``` sh
$ ssh -Y exalogin.ohsu.edu
user@exalogin-el7 $ /usr/bin/firefox
```

### Windows

There are a number of solutions available for Windows, including:

-   [Xming](https://sourceforge.net/projects/xming/) X11 server with [putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/) ssh client
-   [MobaXTerm](https://mobaxterm.mobatek.net/), an integrated X11 server and ssh client

Once logged in to a headnode  or exalogin with X11 forwarding enabled, the process is identical to the above:

### To run on a node

``` sh
$ srun -t 3:00:00 /path/to/gui/application
# Or to run on exalogin:
$ /usr/bin/firefox
```