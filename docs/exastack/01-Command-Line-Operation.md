ACC - Advanced Computing Center : Command Line Operations
===================================================================

Created by Paul Heinlein, last modified on Apr 04, 2023

Exastack operations can be automated using command-line instructions (CLI). The instructions below assume you're using MacOS or Linux computer. We don't yet have instructions for Windows. You'll need a few prerequisites to do command-line work:

-   the python openstack utility and several of its support libraries
-   a connection to OHSU internal networks
    -   on-campus wired or secure wireless networking
    -   OHSU VPN
    -   [an SSH proxy](http://fshead1:8080/ACC/140896710.html)
-   a set of environment variables specific to your project(s)

!!! note
    Several of the script fragments below reference shell variables that have not been defined. They are there to let you know that you will need to add your specific information in that part of the command.

### Install the openstack utility on Mac or Linux


``` sh
#!/bin/bash
#
# getting the openstack command-line utility running in your
# own workstation environment.
#
# ======================================================================

# set up a python 3.x virtual environment in ~/exastack/cli.
# this path name is only an example. you can use or change it
# as desired.

cd
mkdir exastack
python3 -m venv ~/exastack/cli
source ~/exastack/cli/bin/activate
pip install -U pip
pip install -U wheel pysocks

# install various openstack-related utilities using official
# project constraints

pip install -U\
  -c https://releases.openstack.org/constraints/upper/master\
  python-openstackclient\
  python-magnumclient\
  python-barbicanclient\
  python-heatclient\
  python-zunclient\
  python-cinderclient\
  python-glanceclient\
  python-keystoneclient\
  python-neutronclient\
  python-novaclient\
  osc-placement

```

### Project Environment Information

At this point, the `openstack` utility should be in your search PATH. Now you reach a fork in the road: how to provide information about how to contact and authenticate against Exastack. You can

1.  Use dynamic shell environment variables exclusively
2.  Use a shell RC file containing your environment variables
3.  Use a YAML file containing your configuration and authentication information
4.  Mixing and matching dynamic variables with either an RC or YAML file.

Whatever path you choose, you need to provide several bits of information.

=== "Whatever path you choose, you need to provide several bits of information:"
    * [x] the authentication URL
    * [x] your project name
    * [x] your username
    * [x] your user domain
    * [x] your password
    * [x] the OpenStack "region"
    * [x] the API version (YAML only)

I don't like storing my password in a file, so I always load my password dynamically, but all the other information can be stored in a file without much worry about security. If you want to use a YAML file, the [OpenStack Command Line Configuration](https://docs.openstack.org/python-openstackclient/zed/configuration/index.html) documentation is the best place to start. Easier, however, is using the Exastack-provided shell RC file:

-   Log into exastack.ohsu.edu
-   In the upper left-hand corner of the screen, just to the right of the OpenStack logo, use the drop-down menu to navigate to your preferred project.
-   In the upper right-hand corner of the screen, click on your username and then choose the "OpenStack RC File" option.

Your browser will store in your default downloads directory a file named `project-openrc.sh`, which is a bash shell script. So if your preferred Exastack project is called "mylab," then your RC file will be named `mylab-openrc.sh`. You can source that file to pick up all the necessary environment variables:

``` sh
source mylab-openrc.sh

```

It will prompt you for your OHSU password. At that point, you should be able to use the `openstack` utility you installed above. If you are not connected to an OHSU secure network, you will need to set up [a SOCKS5 proxy using SSH](http://fshead1:8080/ACC/140896710.html) before the openstack utility can connect to Exastack.

``` sh
# if you are off campus and using an SSH tunnel with a SOCKS proxy,
# you will also need this variable. make sure the port number matches
# your setup.
export HTTPS_PROXY="socks5h://127.0.0.1:1080"

# if necessary, make sure your ssh tunnel/proxy is running, then
# try this command
openstack server list

# after you've done that, you can leave your virtual environment
deactivate
unset OS_PASSWORD HTTPS_PROXY

```

### Shell automation for openstack environment


Here's how I configured my shells to start and stop access to openstack utility. The first example is for the bash shell, the second for zsh.

``` sh title="bash"
function exastack {\
  local OP="$1"

  case "$OP" in\
    start)\
      source ~/exastack/acc-test-openrc.sh\
      export HTTPS_PROXY="socks5h://127.0.0.1:1080"\
      source ~/exastack/cli/bin/activate\
      ;;\
    stop)\
      unset OS_AUTH_URL OS_IDENTITY_API_VERSION OS_INTERFACE \\
            OS_PASSWORD OS_PROJECT_DOMAIN_ID OS_PROJECT_ID \\
            OS_PROJECT_NAME OS_REGION_NAME OS_USERNAME \\
            OS_USER_DOMAIN_NAME HTTPS_PROXY\
      deactivate\
      ;;\
    *)\
      echo "usage: $FUNCNAME host [start|stop]"\
      ;;\
  esac\
}\
complete -W 'start stop' exastack
```


``` sh title="zhs"
function exastack {
  local OP="$1"

  case "$OP" in
    start)
      source /path/to/yourlab-openrc.sh
      export HTTPS_PROXY="socks5h://127.0.0.1:1080"
      source ~/exastack/cli/bin/activate
      ;;
    stop)

```

### Example: Launch VM


``` sh
# get a list of flavors, choose one
openstack flavor list

# get a list of OS images, choose one
openstack image list

# get a list of virtual networks, choose one, but don't
# choose "ohsu214"
openstack network list

# get a list of ssh keys, choose one (preferably your own)
openstack key list

# get a list of security groups; multiple can be specified,
# each with its own --security-group option
openstack security group list

### XXX replace or set all variables! this can't run as-is

# method 1: boot from OS image and use copy-on-write
# that only saves changes to VM's volume
openstack server create\
  --flavor $FLAVOR\
  --image $IMAGE\
  --network $NETWORK\
  --key-name $SSHKEY\
  --security-group $SECGROUP1 [--security-group $SECGROUP2 ...]\
  $VMNAME

# method 2: create new bootable volume of specified size (in
# GB) from OS image; VM has a self-contained boot volume
openstack server create\
  --flavor $FLAVOR\
  --image $IMAGE\
  --network $NETWORK\
  --key-name $SSHKEY\
  --security-group $SECGROUP1 [--security-group $SECGROUP2 ...]\
  --boot-from-volume 80\
  $VMNAME

```

If you choose to [use cloud-init to customize your instance](http://fshead1:8080/ACC/140894852.html), add the --user-data option to your command:

``` sh
openstack server create --flavor $FLAVOR --image $IMAGE \\
  --network $NETWORK --key-name $SSHKEY \\
  --user-data /path/to/myconfig.yaml
```
Here is an actual working example of a command-line launch. Please **note** that you cannot simply cut and paste this example because the network, SSH key, the "webservices" security group are not yours. The actual values will need to be replaced.

``` sh
openstack server create\
  --flavor m1.small\
  --image cirros\
  --network accnet\
  --key-name heinlein-ed25519\
  --security-group default\
  --security-group webservices\
  --boot-from-volume 30\
  test01

```
!!! example "Assign floating IPv4 address to VM" 
    NETI has made available the 10.96.14.0/24 subnet for OpenStack VMs that need to be addressable directly from other OHSU hosts. Addresses in that subnet can be allocated to customers; they are called "Floating IP" addresses. Here's how to create and/or assign one to a VM.

``` sh
# list floating ip; choose one that is unassigned,
# that is, with no fixed ip address or port
openstack floating ip list

# if necessary, create a new floating ip; it will live on the
# network that Exastack knows as "ohsu214"
openstack floating ip create ohsu214

openstack server add floating ip $VMNAME $FLOATINGIPV4# using the working example above, here how I attached# a floating IP to it; again, you cannot simply cut and# paste this example.openstack server add floating ip test01 10.96.14.172
```