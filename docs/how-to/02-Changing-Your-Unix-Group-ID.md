ACC - Advanced Computing Center : Changing Your Unix Group ID
=============================================================

At login, Unix machines assign you

- User identification number (UID)
- Primary group identification number (GID)
- Optional list of supplementary group ID numbers

You can see all those numbers, and the symbolic names associated with them, with the `id` utility:

``` sh 
[heinlein@exahead1 ~]$ id
uid=4167(heinlein) gid=4167(heinlein) groups=4167(heinlein),
2020(accessACCext),3001(ACCadmin),3010(HPCUsers),[...]

```

It is sometimes the case that you want to switch things up temporarily, replacing your primary group ID with the ID of one of your supplementary groups.

### Why Change Groups?

There are a couple reasons why you might want to alter temporarily your group ID:

1.  You've encountered the 15-group NFS limit
2.  You're creating files within a certain lab's lustre quota

#### The 15-group Limit


Nearly all ACC NFS mounts use what's known as auth_sys credentials, the old-style Unix remote-procedure-call (RPC) credentials that have been around for decades. (An alternative to auth_sys could be, *e.g.*, auth_krb, Kerberos credentials.)

According to [RFC 5531](https://tools.ietf.org/html/rfc5531), an RPC credential can hold up to 15 --- 4 bits, or (2^4)-1 --- groups. That number is hard-wired into the RFC and, for a long time, was coded right into Unix kernels.

If you're a member of 16 or more groups, you may find yourself locked out of NFS directories owned by groups to which you belong. When that's the case, the only trustworthy workaround is to use one of the strategies outlined below to reset your group ID.

If none of the below work, put in a ticket with ACC to have 1 or more groups removed temporary or permanently to get under the 15 group limit.

#### Lustre Quota


You may sometime find yourself working for a group or lab different than your primary group or lab. Since ACC has [implemented Lustre quotas](http://mailman.ohsu.edu/pipermail/exacloud-announce/2017-November/000056.html), you may want to ensure that files you create for that secondary lab aren't counted against your primary lab's quota. In that case, too, changing your group ID for a certain period of time is a good idea.

### Shell Use Cases


There are essentially two different use cases for swapping your primary GID:

1.  For a one-time operation, *e.g.*, to run a program under a different GID so it can write output files into an access-limited directory.
2.  To change your primary GID for the duration of a shell session.

### One-Time Operations


For one-time group changes, the most convenient utility is `sg`, which executes a command under a different group ID.

In the shell session below, a user creates an empty file called `primary-id.txt` and then, using `sg` and another group to which the user belongs, a second empty file called `secondary-id.txt`.

``` sh
[bash]$ touch primary-id.txt
[bash]$ sg MyLab 'touch secondary-id.txt'
[bash]$ ls -l *-id.txt
-rw-r--r--. 1 ohsuuser HPCUsers  0 Oct  2 15:04 primary-id.txt
-rw-r--r--. 1 ohsuuser MyLab     0 Oct  2 15:04 secondary-id.txt

```
!!! tip
    Note that the group owner of the second file matches the group specified during the `sg` command invocation.

#### Shell Sessions


At other times, you might want to alter your group ID for the duration of a shell session. In that case, you have two utilities from which to choose: the aforementioned `sg` and another called `newgrp`.

Both these utilities can be used to create a subshell (from which you can later exit) in which your primary GID is altered. `sg` and `newgrp` interact with your shell settings is subtly different ways, so you're encouraged to do some empirical testing to see which approach is best for you.

Here's `newgrp` in action:

``` sh
[heinlein@exahead1 tmp]$ id -gn
heinlein
[heinlein@exahead1 tmp]$ newgrp HPCUsers
[heinlein@exahead1 tmp]$ id -gn
HPCUsers
[heinlein@exahead1 tmp]$ exit
exit
[heinlein@exahead1 tmp]$ id -gn
heinlein

```

And here's `sg` 

!!! tip
    note the dash in its invocation:

``` sh
[heinlein@exahead1 tmp]$ id -gn
heinlein
[heinlein@exahead1 tmp]$ sg - HPCUsers
[heinlein@exahead1 ~]$ id -gn
HPCUsers
[heinlein@exahead1 ~]$ exit
logout
[heinlein@exahead1 tmp]$ id -gn
heinlein

```

### A Scripting Example


The easiest way to run a script under a different group ID is to call it via `sg`:

``` sh
sg myLabGroup -c "bash myscript.sh"

```

You may at times, however, find it necessary to mix group operations within a shell script. Both `sg` and `newgrp` can be passed "here-documents" with a list of commands. The `sg` utility also accepts a string.

```sh
#!/usr/bin/bash

### Examples of how to embed the sg or newgrp utilities in scripts.

# Grab your first four supplementary groups
G2=$(groups | cut -d' ' -f2)
G3=$(groups | cut -d' ' -f3)
G4=$(groups | cut -d' ' -f4)
G5=$(groups | cut -d' ' -f5)

# These commands will be executed under your current default group.
MYDIR="test1"
printf "Current group is "; id -gn
mkdir -v $MYDIR
chmod g+s $MYDIR

# In this example, the commands are passed to sg as a single-
# quoted string. Note that it's necessary to set the MYDIR
# variable *within* the string set.

sg $G2 -c '
MYDIR="test2"
printf "Current group is "; id -gn
mkdir -v $MYDIR
chmod g+s $MYDIR
'

# Next, commands are passed to sg as a double-quoted string.
# Here that it's necessary to set the MYDIR variable *outside*
# the string set because variables are interpolated prior to
# being passed to sg.

MYDIR="test3"
sg $G3 -c "
printf 'Current group is '; id -gn
mkdir -v $MYDIR
chmod g+s $MYDIR
"

# In next example, commands are passed as a here-document. The delimiting
# word (TEST3) is unquoted, so in this case variables need to be set
# *outside* the command set because they are interpolated prior to being
# passed to sg.

MYDIR="test4"
sg $G4 <<TEST4
printf "Current group is "; id -gn
mkdir -v $MYDIR
chmod g+s $MYDIR
TEST4

# In the final example, the delimiter for the here-document (TEST5) is
# quoted. Variables are NOT interpolated with a quoted delimiter, so
# the MYDIR variable must be set within the command set.

newgrp $G5 <<"TEST5"
MYDIR="test5"
printf "Current group is "; id -gn
mkdir -v $MYDIR
chmod g+s $MYDIR
TEST5

# print results of our test
ls -ld test*

### eof

```

### Python Programming

If you are using a python script, use `os.setgid()` to set the running process to the group ID you need.