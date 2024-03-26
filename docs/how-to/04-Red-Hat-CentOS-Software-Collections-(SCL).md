ACC - Advanced Computing Center : Red Hat/CentOS Software Collections (SCL)
===========================================================================

Red Hat promises software compatibility for the life of any given RHEL release. It will not upgrade major applications mid-release. For example, if RHEL 6.0 contains PostgreSQL 8.4, RHEL 6.7 cannot move to PostgreSQL 9.4. Too many applications will break.

Yet some customers require the upgraded software. By way of an answer, Red Hat and the CentOS project have published what are called Software Collections (SCL). Packages provided in the SCL repositories typically provide newer versions of software that play a key role in the Linux world: Python, Apache, PostgreSQL, MySQL, gcc, etc.

### Accessing SCL Repositories

Red Hat Enterprise Linux (RHEL) and CentOS have different ways of subscribing to yum repositories.

#### RHEL

RHEL uses the `subscription-manager` utility:

``` sh
# for RHEL 6
subscription-manager repos --enable rhel-6-server-optional-rpms
subscription-manager repos --enable rhel-server-rhscl-6-rpms

# for RHEL 7
subscription-manager repos --enable rhel-7-server-optional-rpms
subscription-manager repos --enable rhel-server-rhscl-7-rpms

```

You'll have to contact your local licensing guru if those repositories aren't visible from your host(s).

#### CentOS

Use `yum` to make SCL repositories available on a CentOS machine:

``` sh
# CentOS 6 and 7
yum install centos-release-scl

```

### Listing Packages

The SCL repositories have a **lot** of packages, but you may be interested to see everything that's offered. I suggest piping the output of the following commands to `less` or your favorite pager to ease the reading.

``` sh
# RHEL 6
yum --disablerepo='*' --enablerepo='rhel-server-rhscl-6-rpms' list available

# RHEL 7
yum --disablerepo='*' --enablerepo='rhel-server-rhscl-7-rpms' list available

# CentOS 6 and 7
yum --disablerepo='*'\
    --enablerepo='centos-sclo-rh'\
    --enablerepo='centos-sclo-sclo'\
    list available

```

### Taking the Road Less Traveled

To avoid any system-wide conflicts, SCL packages are installed in `/opt/rh`. This allows, for instance, you to install Python 3.5 on a CentOS 7 machine without removing or interfering with Python 2.7, which is used by roughly one gazillion administration applications crucial to the system's functioning.

Since SCL packages are off the beaten path, people wanting to use them need to upgrade a variety of environment variables: `PATH`, `LD_LIBRARY_PATH`,`MANPATH`, etc. There are two main ways to do this.

#### Use a subshell

There's a command-line tool, `/usr/bin/scl`, that eases access to these packages. It will get installed via the scl-utils package the first time you install an SCL rpm. See the *scl(1)* man page for full usage.

In the session below, I've installed the rh-python35 package on a CentOS 7 machine. The default python is version 2.7.5. When I enable the rh-python35 package using `scl`, I gain access to Python 3.5.1.

``` sh
[heinlein ~]$ scl --list
rh-python35
[heinlein ~]$ echo $PATH
/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
[heinlein ~]$ which python && python -V
/usr/bin/python
Python 2.7.5
[heinlein ~]$ scl enable rh-python35 bash
[heinlein ~]$ echo $PATH
/opt/rh/rh-python35/root/usr/bin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
[heinlein ~]$ which python && python -V
/opt/rh/rh-python35/root/usr/bin/python
Python 3.5.1
[heinlein ~]$ exit
exit
[heinlein ~]$ which python && python -V
/usr/bin/python
Python 2.7.5
```

The upside to this approach is that your permanent shell environment isn't cluttered with `/opt/rh` paths. The downside is that the subshell doesn't necessarily have access to your normal shell aliases, functions, etc.

#### Source the package-provided enable script

SCL packages provide shell scripts that define the environment variables necessary for using the included applications. You can find them in the filesystem as `/opt/rh/<<package name>>/enable`. It's easy to source the `enable` script directly into your working environment.

```sh 
[heinlein ~]$ which python && python -V
/usr/bin/python
Python 2.7.5
[heinlein ~]$ source /opt/rh/rh-python35/enable
[heinlein ~]$ which python && python -V
/opt/rh/rh-python35/root/usr/bin/python
Python 3.5.1

```

The upside to this approach is that all your normal shell conveniences are available. The downside is that you'll have to logout (or do some extensive variable editing) to restore your pristine environment.

### Conquering your daemons

There's an added twist when the SCL package installs a daemon. How does it get started? I don't have a definitive answer for all the packages, but from what I've seen, SCL init scripts work just fine.

For instance, I recently installed the rh-postgres94 package on a RHEL 6 host. Activating the daemon was easy:

``` sh
chkconfig rh-postgresql94-postgresql on
service rh-postgresql94-postgresql initdb
service rh-postgresql94-postgresql start
```