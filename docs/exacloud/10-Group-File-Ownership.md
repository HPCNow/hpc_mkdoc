ACC - Advanced Computing Center : Group File Ownership
================================================================

Filesystems managed by ACC make heavy use of group ownership to limit access to data. Additionally, the Exacloud Lustre filesystem uses group quotas to help maintain fair usage patterns across OHSU research groups.

Our users nearly always are members of multiple groups, and it's not always obvious how to ensure that the proper group ID is being used at any given time. This FAQ seeks to ask and answer common questions related to group identities in a Unix environment.

### Personal Group Memberships


#### What group memberships do I have?

Use the Unix `groups` utility to list your group memberships.

```
[exahead1 ~]$ groups
HPCUsers accessACCext ACCadmin okenuser exacloud_mpi exacloud_gpu
exalabadmin accessexacloud

```

#### What is my current group membership?

Use the Unix `id` utility to see your working group.

```
[exahead1 ~]$ id -gn
HPCUsers

```

#### How can I change my working group?

Use the `newgrp` or `sg` utilities as described in our documentation on [Changing Your Unix Group ID](http://fshead1:8080/ACC/Changing-Your-Unix-Group-ID_22053174.html).

Unix groups
-----------

#### How can I tell which group owns a file or directory?

Within a shell session, the `ls` utility provides the easiest way to see file ownership:

```
[exahead1 lustre1]$ cd /home/exacloud/lustre1/Galaxy
[exahead1 Galaxy]$ ls -l
total 34
-rw-r--r--.  1 root       root     87 Nov  5  2015 00ACC_MOVED_DIRECTORIES.txt
drwxrwx---. 28 galaxyuser galaxy 4096 Dec 13  2017 collectl_logs/
drwxr-xr-x.  6 galaxyuser galaxy 4096 Jun  8  2014 database/
drwxr-xr-x. 37 galaxyuser galaxy 4096 Oct  2  2017 dataset_library_import/
drwxr-xr-x.  3 galaxyuser galaxy 4096 Mar  3  2016 shed_tools/
drwxr-xr-x.  4 galaxyuser galaxy 4096 Mar  3  2016 tool_dependencies/

```

In this case, user "root" and group "root" own the file named `00ACC_MOVED_DIRECTORIES.txt`, while user "galaxyuser" and group "galaxy" own everything else.

#### How can I discover group ownership within a script?

Most programming languages have methods for doing so. Within a shell script on Exacloud, you can use the `stat` utility to report group ownership. Here's a little shell loop that will report group ownership of all items in the current directory.

```
for ITEM in *; do
  GRP=$(stat -c '%G' "$ITEM")
  echo "$ITEM belongs to group '$GRP'"
done

```

#### How do I change group ownership of a file?

The `chgrp` utility was created just for that purpose:

```
# change ownership of a single file
chgrp MyLab /path/to/myFile
# recursively change ownership of an entire directory tree
chgrp -R MyLab /path/to/myDirectory

```
!!! question
    #### What group will own a file I create?

This is a tricky question!

Generally speaking, any new files or directories you create or copy will be owned by your current working group. If that group is `HPCUsers` rather than your lab group, then `HPCUsers` will own the new items.

There are two major exceptions to this rule.

The first involves the use of the `mv` utility. If you move (`mv`) a file rather than copy or create it, the file will retain its old ownership. Let's say you do some work for the (obviously fictional) "ThisLab" group. So you create several files with group ownership assigned to `ThisLab`. After a staff change, that project gets tranferred to "ThatLab." You're a member of both groups, so you use `mv` to move the project files from ThisLab's Lustre directory the ThatLab's. In that case, the files will still by owned by the `ThisLab` group.

The second exception involves directories that have the SGID (set group ID) bit set. Take a look at at the listings for two different directories:

```
drwxrwsr-x.  6 root     biostat    4096 Nov 17  2017 biostat/
drwxr-xr-x.  4 root     root       4096 Mar 25  2015 cBioPortal/

```
!!! tip
    Notice the "s" in the seventh column of the `biostat` directory's permission-mode list. That "s" indicates that the group that owns the directory will become the owner of new files or directories created within it.

One final note: the `mv` exception holds precedence over the SGID exception. If you `mv` files into a directory with the SGID bit set, those files will still retain their original group ownership.

#### That SGID thing sounds interesting. How do I set it?

If you are the owner of a directory, you can set the SGID bit using the `chmod` utility.

To add it to an existing directory, use `g+s`:

```
chmod g+s myDirectory

```

If you want to add it to all the directories in an exiting directory tree, the recipe is a bit more obscure. Some familiarity with the `find` utility is helpful.

```
# add the SGID bit to myDirectory and all of its
# subdirectories
find myDirectory -type d -exec chmod g+s {} \;

```

When creating a directory from scratch, you can use an octal mode to set it:

```
# using the "install" utility
install -d -m 2770 /path/to/myDirectory
# using mkdir
mkdir -m 2770 /path/to/myDirectory

```

#### How about rsync? What options should I use?

`rsync` is a powerful tool for securely copying files and verifying the integrity of the destination files as compared to the source files.

It is often invoked with the `-a` option which is shorthand for many commonly used option. Unfortunately, some of the options hidden within the `-a` invocation don't work well when the destination directory has the group sticky bit set.

If I have a Lustre directory with the SGID bit set and group ownership assigned to my project group (which is *not* my primary group) and use `rsync -a` to recursively copy a directory of files from my workstation to Exacloud, the result will probably not be what you want:

-   the top-level directory you copied will be owned by your project group, which is good, but
-   the files contained in that directory will be owned by your primary group, not your project group.

Instead, try `rsync -rltD`. Using that set of options will allow the group sticky bit on the destination directory to govern ownership of the files as they're copied.

In short:

-   avoid `rsync -a`
-   use `rsync -rltD`
-   use `rsync -rltDc` if you want checksum verification of files

### Lustre group quotas


#### How can I find out my group's quota?

The `lfs` utility will report the Lustre quota for any group of which you are member. (It won't allow you to see quotas for groups to which you do not belong.)

Lustre quotas actually manage two separate limits: one for the overall space used and one for the number of files owned. In the example below the Goodman Lab has used only 3.8 TB out of its 10 TB quota; the group owns 25,697 files, well under its quota of 900,000.

```
[exahead1 ~]$ lfs quota -h -g GoodmanLab /home/exacloud/lustre1
Disk quotas for group GoodmanLab (gid 3155):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
/home/exacloud/lustre1
                  3.85T     10T     11T       -   25697  900000 1000000       -

```

#### What happens when we reach our quota?

Bad things, some of which are hard to predict. Disk writes may fail. Files you're copying may get truncated. Your programs may stall or timeout. Slurm jobs will keep running (and failing) until they've exceeded their allotted time.

#### All my files are owned by the HPCUsers group. Help!

Use the `chgrp` utility (see above) to transfer their group ownership to your project group.

#### But it keeps happening. What now?

You'll want to change your group ID for your whole shell session. Use the `newgrp` or `sg` utilities as described in our documentation on [Changing Your Unix Group ID](http://fshead1:8080/ACC/Changing-Your-Unix-Group-ID_22053174.html) to do so.

#### Will that trick work with slurm?

Yes. If you change your working group ID prior to running `srun`, that ID will remain in place as your job runs