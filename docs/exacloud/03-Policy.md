ACC - Advanced Computing Center : Exacloud Policy
===============================================

### Policy violations

If ACC determines that you have violated any of the below Exacloud policies, your cluster access will be deactivated. Your account will not be reactivated until ACC receives a formal request for reactivation from the primary investigator of your project.

### ACC communications

See [ACC contact information](https://www.ohsu.edu/advanced-computing-center/acc-contact-information).

All service requests and inquiries should be emailed to acc@ohsu.edu. This will open a service ticket in our JIRA system, which will be assigned to an available ACC staff member.

Announcements regarding Exacloud are made via the [exacloud-announce](http://mailman.ohsu.edu/cgi-bin/mailman/listinfo/exacloud-announce) mailing list.

### Data security and OHSU policy compliance

Exacloud and RDS are secure computing locations approved for data classified as Restricted under OHSU policy, including protected health information, but ACC resources cannot be used for projects involving direct patient care. ACC complies with the OHSU records retention and contingency planning policies. Users are responsible for obtaining any required authorization, such as data use agreements, before transferring data between OHSU and external sources.

### Computing resource limits


See the [Job Scheduler documentation](05-Job-Scheduler.md) for details on how to run jobs on compute nodes. See below for specific policies surrounding resource use:

### Do Not Run Programs On The Head Nodes

Simple commands (to compress data, create directories, etc.) that run within a few minutes on the head node are okay, but any scripts, software, or other processes that perform data manipulations/creation can be disruptive to the system. To ensure proper functioning of the cluster for ALL users, computational work should always be run within an interactive session or batch job.

ACC staff have been empowered by the Steering Committee to kill any long-running or problematic processes on the head nodes and/or deactivate user accounts that violate this policy. Users may not be notified of account deactivation. Processes that only occasionally perform work (e.g. crontab, etc.) are still a violation of this policy.

### Do Not Use The Gpu Partition Unless You Are Using Gpus

Jobs submitted to the gpu partition must explicitly request GPU resources, otherwise none will be assigned. On the flip side, jobs which do not make use of GPUs must be submitted to the exacloud partition so that they do not deprive the use of GPUs to other users.

### Infrastructure reliability/uptime and service level


ACC staff provide "best effort" support during business hours to reported issues. We do not provide 24/7 on-demand support service.