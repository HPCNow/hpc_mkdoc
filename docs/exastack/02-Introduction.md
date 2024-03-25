ACC - Advanced Computing Center : Exastack: Introduction
========================================================

The Advanced Computing Center's Exastack environment is a local instance of [OpenStack](https://www.openstack.org/), a very popular open source cloud software. Exastack will allow you to set up and managed your own virtual machines (VMs) and private networks. Some of your VMs may be accessible from other OHSU networks; some may be private, accessible only to your other Exastack VMs. You can use your VMs to develop and run software, deploy Docker containers, run web services, and/or manage a database service. Along with your VMs, you can create virtual hard drives that can live on one VM or be moved from VM to VM as your work dictates.

If you're interested in getting access to Exastack, please open a Jira ticket with ACC with either [the ACC service desk](https://service.ohsu.edu/servicedesk/customer/portal/10) or by sending an e-mail to <acc@ohsu.edu>.

### Quota Reservations

Access to Exastack is purchased in quota-reserve units. Each unit reserves for your project

-   Two hyper-threaded (one physical) CPU cores; a hyper-threaded CPU is also called a vCPU (virtual CPU)
-   16 GB RAM
-   200 GB disk space

Based on how many units you've purchased, we automatically set some other limits: floating IPv4 addresses, networks and subnets, Unlike the bullet-listed limits, however, these related limits can be negotiated if you have special requirements.

It's worth noting that the CPU and RAM quotas are *runtime* limits, while the disk quota applies regardless of runtime. Why is that important? Consider the following example. Your lab purchases a single quota unit, and you configure virtual machines with 2 vCPUs, 16 GB RAM, and 50 GB of disk space. In that case you could have four VMs configured at any one time --- since combined they reserve 200 GB of disk space --- but only one running at a time, since each VM uses your full quota of CPU and RAM.

Please contact ACC for current pricing, which may change between fiscal years.

### Caveats

There are some important restrictions that apply to Exastack that need to be noted from the beginning. Exastack runs largely as a self-service operation, it does not provide ACC network and file services, and it is available only from machines on a secure OHSU network.

### Self Service

Up front, we will note that Exastack is predominately a self-service operation. ACC will set up your basic access and will configure your quotas. The setup process will also create a basic networking infrastructure that you may (or may not) choose to use for your VMs. Beyond providing basic access, ACC turns over *all responsibilities* to you. That is, you are responsible to launch your VMs, to keep them patched, to troubleshoot their software failures, to create user accounts, etc. ACC has no "back door" into your VMs and cannot fix their internal problems. The same is true of data protection. ACC provides no native backup of Exastack resources. You will need to determine how best to safeguard your data.

### No ACC Services

Due to several technological limitations, Exastack VMs **cannot** use ACC LDAP services, mount ACC home directories, mount NFS shares from the Research Data Storage (RDS) platform, or directly access Exacloud services like slurm or Luster. There are no exceptions to these limitations.

### On-Campus Only

At present, only computers connected to an OHSU secure network or endpoint --- via an on-campus wired connection, OHSU-Secure wifi, OHSU VPN, or [an SSH session](http://fshead1:8080/ACC/140896710.html) --- can connect to VMs running in Exastack. They are not eligible to be visible to the public Internet. We understand that this restriction might be frustrating, but ACC and the security teams have not yet devised a way to guarantee that Exastack VMs consistently adhere to OHSU best security practices.

### Logging In


The easiest way to operate within Exastack is to login via the web console, found at [exastack.ohsu.edu](https://exastack.ohsu.edu/). The login screen will request

-   Username --- your ACC username (typically the portion of your OHSU e-mail address before the '@' character)
-   Password --- your ACC password (typically the same as your OHSU e-mail password)
-   Domain --- should be 'ACC' (not 'Default')

Alternatively, you can manage Exastack operations using [command-line tools](01-Command-Line-Operation.md).

### What You Receive


When you first are given access to Exastack, we will pre-create for you several resources to house and assist your work.

#### Projects

"Project" is the OpenStack term for how customers are kept separate from one another. You will receive access to two projects: one subject to your purchased quota (typically named for your project or lab) and one subject to twice your purchased quota (with your project or lab name appended with the word "Flex"). Note that you can only view one project at a single time. The Exastack web interface provides a drop-down menu (upper left corner) so you can switch between your projects.

The quota and flex projects differ from each other in significant ways.

#### Quota Project

ACC makes an effort to guarantee the compute resources (CPU, RAM, disk) purchased for your quota project. VMs and disk volumes that run in your main project should never be starved for resources. You can keep your quota project VMs and disk volumes active for as long as you see fit.

### Flex Project

The flex project, on the other hand, is intended only for temporary VMs and volumes. ACC reserves the right to administratively remove or destroy flex VMs and volumes after a period of a few days. (This policy is still in flux and will be defined more clearly in the future.) We envision your flex project to be used only when you have a pressing and temporary need for Exastack resources that surpass your purchased quota. All flex projects are confined to a subset of the Exastack hardware and are more likely to encounter resource contention than quota projects.

### Networking

Each of your projects will come with a pre-created virtual network you can use to host your virtual machines. Depending on your project's quota, you will be able to create other networks and subnets as required.

When you [launch your first VM](http://fshead1:8080/ACC/140894875.html), you will likely find it unreachable using SSH or similar login tools. Please see [Exastack: Networking](http://fshead1:8080/ACC/140895333.html) for an explanation of the cluster's networking environment and how to make a VM visible to the wider OHSU network