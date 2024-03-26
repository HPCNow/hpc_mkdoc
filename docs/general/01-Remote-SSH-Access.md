ACC - Advanced Computing Center : Remote SSH Access
========================================================

You will often want to access ACC resources like Exacloud or Exastack while working remotely, but there is no direct Internet access to them. Many people will use the OHSU VPN or Citrix facilities to gain remote access, and those are fine tools. Others, perhaps unwilling or unable to use Citrix or the VPN, may consider remote access via SSH. Let's consider a very simplistic view of the networking topology in this situation:

``` mermaid
graph LR
    A[Internet] ---- |137.53.248.122| B[Bastions/Jump];
    A ---- |10.96.10.55| C[acc.ohsu.edu];
    B ---- |10.0.0.0/8| D[Private/.ohsu.edu];
    C ---- |10.0.0.0/8| D;
```
!!! info
    The bastion host can also be called a "jump host." The ACC jump host has two interfaces: public (137.53.248.122) and private (10.96.10.55). (Note: these IP addresses may change over time.)

### The Simple Way


The simple way to access a *.ohsu.edu host (e.g., exacloud.ohsu.edu) is with a shell one-liner:

``` sh
# plain ssh
ssh -J username@acc.ohsu.edu username@exacloud.ohsu.edu
# scp
scp -o 'ProxyJump username@acc.ohsu.edu' myfile.txt username@exacloud.ohsu.edu:/my/dir

```

!!! tip "When Simple Isn't Enough"
    It's often the case, however, that you login to several ohsu.edu hosts and that you further have some internal-only web applications you want to access there. (What follows will work more easily if you have SSH keys working with ssh-agent, but it's not necessary.)

### MS Windows

Now is a good time to mention that the SSH configurations and examples shown below were written and tested on Mac and Linux workstations using the system-provided ssh utility. Windows users, on the other hand, will need to choose among several possible ssh installations:

-   the [built-in SSH client](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui) (using at least Windows Server 2019 or Windows 10 build 1809) and PowerShell
-   the SSH client included with [the Windows Subsystem for LInux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/about)
-   third-party applications like [PuTTY](https://www.putty.org/), [MobaXterm](https://mobaxterm.mobatek.net/), and others

!!! info
    Each option has its own pros and cons, so you may need to experiment before settling on a solution.

### SSH Client Configuration


First, we need to setup your `~/.ssh/config file`. The Host stanza for your bastion host will also enable three optional SSH features: compression, agent forwarding, and SOCKS5 proxying (via the DynamicForward setting). The SOCKS5 proxy will allow you to reach your internal-only web apps such as those exposed by the Exastack API.
!!! note
    All the options configured below have command-line equivalents. For example, "ssh -C" is the same thing as setting "Compression yes." ACC provides these examples with the assumption that you will consult with the ssh manual pages if you have questions about a setting's propriety for your work.

``` sh title="~/.ssh/config"
Host acc.ohsu.edu
  Compression yes
  User username
  ControlPath ~/.ssh/cm-%r@%h:%p
  ControlMaster auto
  ControlPersist 9H
  ForwardAgent yes
  DynamicForward 127.0.0.1:1080
```
The various Control directives allow you to run multiple SSH sessions over a single network connection. The `ssh_config(5)` man page has more information.

The next customization for your ssh config file ensures any SSH session destined for *.my.com uses your control session. This stanza must specifically exclude our bastion host:

``` sh title="~/.ssh/config"
Host *.ohsu.edu !acc.ohsu.edu
  CheckHostIP no
  ProxyCommand ssh acc.ohsu.edu -W %h:%p
```

!!! warning "Import"
    You'll need to use fully qualified domain names (e.g., exahead2.ohsu.edu) rather than short versions (exahead2) if you want to use the multiplex connection.

### SSH Session Setup


Now setup the session:

``` sh
ssh -f -N acc.ohsu.edu 2>/dev/null

```

The `-f` `-N` options will invoke ssh without a remote command and put it into the background. You will need to enter your password and complete Duo authentication. Then you'll be returned to your local shell; that is, it won't look like you've logged into ACC. This is because the session is running in the background. ssh will allow you to check its status:

``` sh
ssh -O check acc.ohsu.edu

```

It will return something like "Master running" if successful.

Now you can do `ssh username@exacloud.ohsu.edu` in a terminal window and you will be able to log directly into the remote machine. (I say "directly," but if you run "who" while logged in, your session will show up as coming from the bastion host.) As a side benefit, you will also be able to use `scp` directly to ohsu.edu hosts. Remember, you need to use the FQDN.

When you want to close your SSH session, use the exit command:
``` sh
ssh -O exit acc.ohsu.edu
```

### SOCKS5 Web Proxy

You can also use SSH to set up a proxy that allows you to access on-campus web services. This can be useful for connecting to a site like this wiki. It's further helpful if, for example, you want to run the openstack command-line client on your workstation, or if you want to run a Python program that uses the openstack API. In the latter case, you will need to tell your program(s) about the SOCKS5 proxy you configured in your DynamicForward directive (above). Although you probably won't be using curl to access OHSU web services, showing how it finds the proxy can be instructive.

### curl

The easiest web client to configure for this is the `curl` command-line utility. You just need to set the proxy option:

``` sh
# all on one line
curl --proxy socks5h://127.0.0.1:1080 https://wiki.ohsu.edu/

# or set an environment variable for multiple invocations
export HTTPS_PROXY="socks5h://127.0.0.1:1080"
curl https://wiki.ohsu.edu/
curl https://exastack.ohsu.edu/

```

The "socks5h" protocol designation ensures that DNS name resolution is done at the SSH bastion host, not on your local workstation. The 
**ENVIRONMENT** section of the `curl(1)` man page lists the range of environment variables you can use to indicate the presence of a proxy.

### openstack

If you set the `HTTPS_PROXY` variable or the broader `ALL_PROXY` variable in your shell session, the openstack client will understand that you have a proxy in place and it will try to use it. Consider the following shell session. The user has successfully started a background SSH session, complete with DynamicForward, to acc.ohsu.edu. The user tries to contact Exastack but fails (with a very long error message). After setting the `HTTPS_PROXY` variable, the command succeeds:

``` sh title="Command Failed"
(cli) [heinlein@rocky ~]$ openstack server list
Failed to discover available identity versions when contacting [https://exastack.ohsu.edu:5000](https://exastack.ohsu.edu:5000/). 
Attempting to parse version from URL.
Could not find versioned identity endpoints when attempting to authenticate. 
Please check that your auth_url is correct. Unable to establish connection to [https://exastack.ohsu.edu:5000](https://exastack.ohsu.edu:5000/): HTTPSConnectionPool(host='[exastack.ohsu.edu](http://exastack.ohsu.edu/)', port=5000): Max retries exceeded with url: (Caused by NewConnectionError('<urllib3.connection.HTTPSConnection object at 0x7f3b189a38b0>: Failed to establish a new connection: [Errno -2] Name or service not known'))
```

``` sh title="Command succeed"
(cli) [heinlein@rocky ~]$ export HTTPS_PROXY=socks5h://127.0.0.1:1080
(cli) [heinlein@rocky ~]$ openstack server list
+--------------------------------------+--------+--------+----------------------------------+--------------------------+----------+
| ID                                   | Name   | Status | Networks                         | Image                    | Flavor   |
+--------------------------------------+--------+--------+----------------------------------+--------------------------+----------+
| e031c5c8-742f-4c88-994c-9a0e8d6a75a0 | test00 | ACTIVE | accnet=10.10.10.95, 10.96.14.249 | N/A (booted from volume) | m1.small |
+--------------------------------------+--------+--------+----------------------------------+--------------------------+----------+
```

### Firefox

If you want a full-featured web browser for accessing OHSU web sites like `exastack.ohsu.edu`, you can use Firefox for that:

1.  Launch Firefox
2.  Enter "about:config" into the URL box.
3.  Ignore the "this might void your warranty" warning.
4.  Change some settings:

``` sh
network.proxy.socks: localhost
network.proxy.socks_port: 1080
network.proxy.socks_remote_dns: true
network.proxy.type: 1

```

Again, you'll want to make sure the the port number matches the one specified in the DynamicForward setting in your ssh configuration file.

I've found it necessary to start my proxied browser within a couple minutes of setting up the control/multiplex session. Something times out otherwise, but I haven't really sought to understand that. At that point, Firefox ought to be able to browser your internal web sites.

It's also worth noting that, with those configuration changes in place, Firefox will fail to work properly if your ssh session isn't running. Personally, I only use Firefox for proxied communications, but if you want more flexibility you can setup a [Firefox profile](https://support.mozilla.org/en-US/kb/profile-manager-create-remove-switch-firefox-profiles) just for proxied sessions.

### FoxyProxy

Another option for proxying browser traffic is the extension available from [FoxyProxy](https://getfoxyproxy.org/). The company offers various commercial proxy and VPN services, but its proxy extension for various web browsers --- Chrome, Edge, Firefox, Safari, and others --- is open source and [free to download](https://getfoxyproxy.org/downloads/#proxypanel).

### SSH Session Close

As mentioned above, when you're ready to close your multiplex session, just ask ssh nicely:

``` sh
ssh -O exit acc.ohsu.edu
```