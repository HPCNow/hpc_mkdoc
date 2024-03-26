ACC - Advanced Computing Center : Using ProxyJump with SSH and SCP
==================================================================

It's somewhat common to have what's known as a "jump host" serve as an SSH gateway to a remote network. You use ssh to log into the jump host (or "jump server") and from there use ssh to log into an internal host that's not directly accessible from the Internet. ACC uses `acc.ohsu.edu` for this purpose.

With the release of ssh version 7.3, the OpenSSH folks made it easier to do the jump and internal login in one step.
Check what version of ssh you have installed on your system:

``` sh
ssh -V

```

If you have OpenSSH version 7.3 or higher, you can use the new `-J` (aka `ProxyJump`) command. Here's the basic invocation:

``` sh
ssh -J acc.ohsu.edu exahead1.ohsu.edu

```

You'll end up logged into `exahead1`, and ssh automatically takes care of the intermediate step of logging into `acc.ohsu.edu` first.

You can even use it as an option for secure file copies:

``` sh
scp -o 'ProxyJump acc.ohsu.edu'\
  myfile.txt\
  exahead1.ohsu.edu:/home/exacloud/lustre1/myLab/myDir

```
The file `myfile.txt` will end up in your lab's directory tree in the Lustre filesystem.

If you receive a connection closed by <ip address,port#> and no error, there is a chance you are over the 15 group limit. Please refer to this [article](https://wiki.ohsu.edu/display/ACC/Changing+Your+Unix+Group+ID) on how to resolve.