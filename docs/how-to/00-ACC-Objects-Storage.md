ACC - Advanced Computing Center : Object Storage
====================================================

Object Storage is unlike any other storage services ACC offers. The Internet offers several discussions of "object storage vs. file storage," but in our environment you'll generally notice four points of difference.**You can only copy files to and from object storage. You cannot edit them in place.**

Object storage will not appear as a directory or drive letter; its files will not show up using traditional tools like ls (Unix) or File Manager (Windows). Files in object storage are always "remote" until you copy them to a traditional local or network filesystem. In this sense, object storage functions much more like an FTP service than a traditional filesystem.

To edit a file in object storage, you must first copy it to a local filesystem, do your work on it, and only then copy back into place in the object storage service. **Specialized tools are necessary to access object storage.**

ACC's object storage is accessed via the S3 API, an industry-standard protocol maintained by Amazon. There are other object-storage protocols, like Swift, but ACC provides only S3-based access.

There are a few programs that allow you to access S3 buckets. [Cyberduck](https://cyberduck.io/) provides a graphical user interface for Mac and Windows users. [CloudBerry Explorer](https://www.msp360.com/explorer/windows/amazon-s3.aspx) comes in free and commercial versions for Windows users.

For anyone wanting to access ACC object storage from Linux machines or from within shell or Python programs, the AWS-supplied [awscli](https://docs.aws.amazon.com/cli/index.html) suite of programs is the usual starting point. **File are stored in "buckets." Object storage doesn't offer directories.**

You can add a '/' to a filename and many object-storage access tools will present a falsified directory tree view of your files. That tree view, however, is just a pleasant fiction that helps you make sense of your files. Any directory you see in your tools will not really exist. All files just live in the main bucket. ***Access is granted by per-user keys, not by username and password.**

There is no traditional user or group ownership of files. While complex ACLs can be employed to limit access to files in an object-storage bucket, there's no way to, say, change the ownership of a file. Generally speaking, a user (actually, a set of access keys) can have read/write permissions to all files in a bucket, read-only permissions for those files, or no permissions at all.

### And One Other Thing


The ACC object storage setup is such that users need to know ahead of time the bucket(s) they can access. No one can browse a bucket list in our setup. Unless you know or can guess your bucket's name, you probably won't be able to use it.

This restriction is a technical necessity brought about by the fact that, unlike Amazon Web Services (AWS), we sell storage space to projects, not individuals. We don't like the restriction, but for now we cannot work around it.

### Buckets, Files, and Quotas


ACC object storage is a service shared among many OHSU labs and projects, making it necessary for us to set quotas to ensure everyone gets the space reserved for them.

-   Each project reserves a certain amount of space.
-   Each project has a list of users that can access the space. (A user may be a member of multiple projects.)
-   A project's space is divided into one or more buckets.
-   For each bucket, a project user may have one of three permission levels: administrator, read-write, or read-only.

The quota typically applies to the overall project space and not to the buckets created within that space.

### Visibility


ACC object storage is visible from all computers on OHSU trusted networks. It is not accessible from guest wireless networks or off-campus computers.

### Data Redundancy


ACC object storage currently provides **no object storage node redundancy**. Thus, loss of an object storage node may impact the availability of your object data until ACC staff can replace the failed node. As of January 2021, there is a total of six (6) storage nodes in the ACC object cluster.

Each **node does have internal drive redundancy**. The storage on the nodes is configured with 9+3 EC , so there is fault tolerance for three drives in a node before data loss.

Also note that your object storage archives would live in our West Campus data center, so there wouldn't be any physical separation of your main and archival data stores in the event of a physical disaster.

### Working with ACC Object Storage

When ACC sets up your storage, we will do a few things:

1.  Create the bucket(s) you request.
2.  Create the necessary user accounts. If a user already has an object-storage account from another project, that account will be valid for the new project as well.
3.  Convey to the holder of each new user account an "access key" and a "secret access key." If those keys are lost or compromised, ACC can create new keys fairly easily.
4.  Apply to the new bucket(s) the specifed access-control rules.

### Using awscli

On your own server or workstation, install the awscli Python suite. Most Linux distributions include a package for it. I've been using Ubuntu 18.04 and MacOS 10.13 for testing. You can find [more detailed installation instructions](https://cloudacademy.com/blog/aws-cli-a-beginners-guide/) online.

Once you've got the awscli package installed and the "aws" utility in your PATH, you'll want to do two things to get S3 operations to work.

First, create a section in your `~/.aws/credentials` file to hold your user keys (which will be conveyed you individually and securely). In the example below, the profile will be named `ohsu`.

``` sh
[ohsu]
aws_access_key_id = 12345678901234567890
aws_secret_access_key = 1234567890123456789012345678901234567890

```

Second, and this is optional but I suggest it, create a shell alias. In my example, I call it `oos` (OHSU Object Storage). Make sure the profile argument matches the name in your AWS credentials file.

``` sh
alias oos="aws --profile=ohsu --endpoint=https://rgw.ohsu.edu"
```

### Using Cyberduck


To use Cyberduck, you'll need to know the name of your bucket, your access key and your secret access key.

- [x]  In Cyberduck, press the Open Connection button and use the following information:
    * [x]  `Connection Type`: Amazon S3
    * [x]  `Server`: rgw.ohsu.edu
    * [x]  `Port`: 443
    * [x] `Access Key ID`: *your access key*
    * [x] `Secret Access Key`: *your secret access key*
- [x]   Press Connect
- [x] Navigate to Go --> Go to Folder (Ctrl+G)
- [x] Enter the path: `/MyBucket` (replacing "MyBucket" with the name of your bucket, but be sure to keep the opening slash character)

### Basic S3 operations

As mentioned above, an unfortunate side effect of the way ACC sets group storage quotas is that you cannot easily get a list of buckets to which you have access. You need to know their names beforehand. (ACC will provide that information for you.)

``` sh
# this uses the shell alias "oos" described above and assumes that your
# project bucket is named "MyBucket"

# copy a single file to your bucket
oos s3 cp myarchive.tar.gz s3://MyBucket

# copy that file back to your system; then delete it from your bucket
oos s3 cp s3://MyBucket/myarchive.tar.gz .
oos s3 rm s3://MyBucket/myarchive.tar.gz

# copy a whole directory, in rsync fashion
oos s3 sync /home/me/mybackup s3://MyBucket/mybackup

# restore local mybackup directory from Ceph
oos s3 sync s3://MyBucket/mybackup /home/me/mybackup

# empty your bucket
oos s3 rm --recursive s3://MyBucket/

```

### Versioning

The S3 API provides for versioned storage. That is, if you upload a file (for example, `myfile.zip`) and then upload a file with the same name, both versions will still be available. The latest one to be uploaded will become the default file available, but the older version will still be available if you specify its version ID.

When versioning is enabled, each version of a file counts separately against your quota. The versioned files are separate objects; even if the difference between one version and the next is very small, the full file size of each version is counted against your quota.

Versioning is not enabled by default. The awscli tools can be used to see if it's enabled for your bucket. As with other examples, this one uses the `oos` shell alias described above.

``` sh
[bash]$ oos s3api get-bucket-versioning --bucket MyBucket
{
    "Status": "Enabled",
    "MFADelete": "Disabled"
}

```

If versioning is not enabled on your bucket(s), there will be no message at all. In that case, when you overwrite an existing file, the old file is gone for good. A user with administrative rights to the bucket can enable versioning:

``` sh
oos s3api put-bucket-versioning\
  --bucket MyBucket\
  --versioning-configuration Status=Enabled

```

Once versioning is enabled, you have access to some new commands.

``` sh
# list versions of everything in your bucket
oos s3api list-object-versions --bucket MyBucket

# list versions for a single file
oos s3api list-object-versions\
  --bucket MyBucket\
  --prefix myfile.zip

# retreive a specific version; you need to specify the name of the
# output file when using the get-object subcommand.
oos s3api get-object\
  --bucket MyBucket\
  --key myfile.zip\
  --version-id OHFknTDRPEfMH7ugWvGt85XOf7xWCmr\
  myfile.zip

```

### Deleting Versioned Objects


The Ceph developers are [still wrestling with object versioning](https://tracker.ceph.com/projects/ceph/wiki/RGW_Object_Versioning). The upshot is that deleting a versioned file isn't as clean or straightforward as it might be. Javier Mellid has written an article that describes all the ways [versioned and unversioned operations differ](http://javiermunhoz.com/blog/2018/11/14/on-ceph-rgw-s3-object-versioning.html). There are too many to list in this overview, but Red Hat's official Ceph documentation says that to "delete an object when versioning is on, you must specify the versionId ... of the object to delete."

### Frequently Asked Questions


!!! question "I'm seeing odd errors when I try to send files to object storage."
    One quirk of our quota system is that the technology will allow you to create one (and only one) bucket, but won't allow you to put data into that bucket. Verify that you're trying to move data to the bucket assigned to your project.

    Another possible problem is that your bucket is nearing or at its quota. You can test that using the awscli command-line tools described above. The last couple lines of output will tell you the total number of objects in your bucket and their aggregate size.

    ``` sh
    # use the 'oos' alias:
    # alias oos="aws --profile=ohsu --endpoint=https://rgw.ohsu.edu"
    oos s3 ls --summarize --human-readable --recursive s3://MyBucket
    ```

!!! question "I'm connected to the OHSU-Secure network but I cannot contact rgw.ohsu.edu."
    It's likely that your machine was administratively cut off from the OHSU internal network due to network access control (NAC). You might try rebooting your machine first. If that is unsuccessful, your device may not meet the OHSU [requirements for computer security](https://o2.ohsu.edu/information-technology-group/help-desk/it-help-pages/faq-network-access-control.cfm). There's also a chance your machine was restricted accidentally.

    You can call the ITG Help Desk at 4-2222 to begin the process of bringing your machine into compliance or clearing up the problem.

!!! question "I can't rename a directory in object storage."
    As mentioned above, object storage doesn't have directories like ordinary filesystems, but you can put a `/` in the filename of files you upload. If you do so, most S3 clients will display your bucket as if there were directories. Despite that visual convention, you cannot, for example, rename a directory. You would instead need to rename all the files within that "directory.