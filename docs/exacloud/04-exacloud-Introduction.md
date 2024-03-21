### Requesting an account
To request access to the cluster, send an email to [acc@ohsu.edu](mailto:acc@osu.edu). In the email, include:

* Your department or lab group
* How you will be using the cluster
* OHSU internal billing Alias, FAID and name of the financial admin for the funds source
Policy


***The use of Exacloud is governed by policies approved by the Exacloud Steering Committee.***

Log in to head node to log in to the cluster, point your SSH client to either one of the two cluster head nodes, e.g.:

### First head node
```
ssh exahead1.ohsu.edu
```
### Second head node
```
ssh exahead2.ohsu.edu
```

**This name can also be used to access a head node**
```
ssh exacloud.ohsu.edu
```

Give your OHSU username and password as credentials at log-in.

You will then have an open shell session, which will be the gateway to using te job scheduler and accessing the filesystem.

Lab Group Directory locations
When the user logs into the Exacloud cluster, they are placed in the ACC NFS home directory at `/home/users/<user>`. It has only a **10GB** hard quota and it is **not recommended that you use this storage for cluster jobs**.

The best practice is to use your group's directory on the Lustre parallel file system, `gscratch`. The `gscratch` file system is both faster and capable of handling more load. Due to limited capacity resources, please do not store data unrelated to cluster usage in your gscratch directory.

Your group's directory can be found at `/home/exacloud/gscratch/<group>/`. A common practice is to create a directory for yourself in `/home/exacloud/gscratch/<group>/<user>`, or to use a project-specific sub-directory. Check with your group to coordinate this space.

!!! warning "Import"
    Please see the Storage page for more details on gscratch and other filesystems available in exacloud.

Finding Software
The Exacloud head and compute nodes have a large variety of software applications in the standard system directories like `/usr/bin`.

If you cannot find an application in the standard directories, a few other directory trees contain an assortment of applications and libraries requested by the OHSU user community:


=== "Modules"
    * Spack (advanced users)
    * `/home/exacloud/software`
    * `/opt/installed`
    * `/usr/local`


**Another set of those applications are available via the Red Hat/CentOS Software Collections Library (SCL).**

Submitting a Job
To see how to submit your first job, proceed to the [Job Scheduler page]().

Support
If you encounter an obstacle, please consult the Exacloud Troubleshooting documentation. If you are still at a dead end, please contact ACC for support through the web portal or by emailing [acc@ohsu.edu](mailto:acc@osu.edu).

!!! tip
    There is also an Exacloud Teams Channel intended for user mutual support and communication. ACC staff participate in the channel, but support issues still need to be escalated as above.