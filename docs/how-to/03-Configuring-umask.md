ACC - Advanced Computing Center : Configuring umask
===================================================


The shell built-in command [umask](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-umask) is used to control which permissions are set on newly created files and directories.

### ACC Default

The default umask on ACC systems is `0027`. This setting results in the following permissions on new directories and files:

-   `User`: The creator has full permissions to new directories and read+write permissions to new files.
-   `Group member`: Can enter new directories, and read (but not write) new files.
-   `Other users`: Cannot enter new directories and cannot read or write new files.

Below is a demonstration of the permissions of new directories and files when using the `0027 umask`:

``` sh
$ mkdir newdir
$ touch newfile
$ ls -l .
total 1
drwxr-x--- 2 user FooLab 2 Oct 11 10:59 newdir
-rw-r----- 1 user FooLab 0 Oct 11 10:59 newfile
```

The default umask does not prevent you from manually changing the permissions of directories and files with `chmod`.

### Changing umask


If for some reason the default umask is not satisfactory, it can be changed, either temporarily or permanently.

To change umask in your current shell session, use the built-in `umask` command:
``` sh
$ umask 0007
```
To change umask for all future shell sessions, add the `umask` command to the `~/.bashrc` file in your ACC home directory:

``` sh title="~/.bashrc"
umask 0007
```

### Collaboration

ACC recommends always using `7` for the final digit in your umask, as this is the best privacy-preserving option. If you need to make some new directories and files accessible to users outside your group, consider using a temporary umask, or manually changing the permissions of files. However, opening up permissions to other users will result in any ACC user, not just your collaborators, being granted access.

If you need to grant access only to certain outside users and/or selectively to certain files and directories, [contact ACC](mailto:acc@ohsu.edu) for assistance.

### Understanding umask


The term "umask" refers to the fact that the setting masks, or prevents, certain permissions from being set on new files. It is in a sense an inverse of the file permission mode used by `chmod`.

To find the resulting permission for a given umask, you subtract the umask numbers from the default directory (`0777` or `drwxrwxrwx`) or file (**0666** or `-rw-rw-rw-`) modes.

For example:

=== "With umask 0077:"
    -   New directories have 0700 or `drwx------`
    -   New files have 0600 or `-rw-------`
=== "With umask 0002:"
    -   New directories have 0775 or `drwxrwxr-x`
    -   New files have 0664 or `-rw-rw-r--`