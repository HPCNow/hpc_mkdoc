ACC - Advanced Computing Center : Launch a Virtual Machine
====================================================================

Instances are virtual machines that run inside of Exastack.

Before you can launch an instance, gather the following parameters:

-   The **instance** source can be an image, snapshot, or block storage volume that contains an image or snapshot. ACC provides several standard Linux images, but you can also import your own.
-   A **name** for your instance. The name is visible only within your project, so you don't need to worry about it conflicting with other host names at OHSU.
-   The **flavor** for your instance, which defines the compute, memory, and storage capacity of nova computing instances. A flavor is an available hardware configuration for a server. It defines the size of a virtual server that can be launched. Some OS images will have minimum requirements that may limit how small your VM can be, and your quota may limit how large your VM can be.
-   Any **user data** files. A user data file is a special key in the metadata service that holds a file that cloud-aware applications in the guest instance can access. For example, one application that uses user data is the cloud-init system, which is an open-source package available on various Linux distributions and that handles early initialization of a cloud instance. ACC provides a short [Cloud-Init HOWTO](http://fshead1:8080/ACC/140894852.html) elsewhere in this section.
-   An **key pair** for your instance, which are SSH credentials that are injected into images when they are launched. For the key pair to be successfully injected, the image must contain the cloud-init package. Create at least one key pair for each project. If you already have generated a key pair with an external tool, you can import it into OpenStack. You can use the key pair for multiple instances that belong to that project.
-   A **security group** that defines which incoming network traffic is forwarded to instances. Security groups hold a set of firewall policies, known as security group rules. Your project was created with a default security group, but you are free to define your own groups as necessary.

If needed, you can assign a floating (public) IP address to a running instance to make it accessible from outside the cloud. You can also attach a block storage device, or volume, for persistent storage.

You can find some examples of [how to launch a VM using command-line tools](http://fshead1:8080/ACC/140894702.html) elsewhere in this section, but below we'll walk you through launching an instance while logged into the web interface.

### Launch an Instance via the Web Interface


Log into [exastack.ohsu.edu](https://exastack.ohsu.edu/).

Make sure you are logged into the appropriate project. If not, use the drop-down menu in the top-left next to the OpenStack logo to choose the correct project.

In the left-hand menu, navigate to Compute > Instances.

Press the Launch Instance button and you should see a "Launch Instance" floating window.

- [x]  In the Details tab:
    - Enter the Instance Name (required).
    - Optionally, enter a Description.
    - Leave the Availability Zone set to nova.
    - Set the Count, which is the number of identical instances that will be launched together. If you set the count to a number higher than 1, your machines will be given a number following the name you provided, e.g., myvm-1, myvm-2, etc.
    - Press the Next button
- [x]  In the Source tab:
    -   For Select Boot Source, you will want to leave the value at Image unless you have already created a bootable volume you wish to use.
    -   For Create New Volume, selecting Yes (the default) is the safest for long-term use. If you are just experimenting, you might find that selecting No will speed up operations a little bit.
    -   For Volume Size, you don't need to enter a number if you're going to use the disk size associate with the flavor (chosen in the following tab, below). You can, however, specify the volume size if you want a size other than that offered by the flavors.
    -   For Delete Volume on Instance Delete, you are deciding if you want Exastack to automatically clean up your volumes when you delete this instance. Volumes count against your quota, which might be a consideration for you.
    -   Under Available, choose one of the images by pressing the right-hand up arrow. The cirros image is good for testing, but not really geared for long-term use.
    -   Press the Next button
- [x]  In the Flavor tab:
    -   Under Available, choose a flavor. Flavors ineligible due to image requirements or quota restrictions will be marked with warning icons.
    -   Press the Next button
- [x]  In the Networks tab:
    -   You may find a network already allocated. If so, it's probably the best choice unless you specifically know otherwise.
    -   Otherwise, choose one of your project's network from the Available list.
    -   Press the Next button
- [x]  In the Network Ports tab:
    -   In most cases, there's nothing you need to do here.
    -   Press the Next button
- [x]  In the Security Groups tab:
    -   You can probably leave the default group in place unless you know otherwise.
    -   Press the Next button
- [x]  In the Key Pair tab:
    -   Select an Available key.
    !!! warning "Important"
        You need to be the owner of the key you choose. If you are unfamiliar with SSH, please consult a colleague or open a Jira ticket with ACC for instructions.
    -   Press the Next button
- [x]  In the Configuration tab:
    -   Select or paste in a [cloud-config YAML script](http://fshead1:8080/ACC/140894852.html) if you so desire.
    -   Unless you specifically know otherwise, leave Disk Partition set to Automatic.

At this point, you can probably press the Launch Instance button. The final tabs, Scheduler Hints and Metadata, are often not used at all.

If you did not provide user data via cloud-init, then the username you'll use to log into your instance will vary by Linux distribution:

-   Cirros username is 'cirros'
-   CentOS username is 'centos'
-   Rocky username is 'rocky'
-   Ubuntu username is 'ubuntu'

You may want to run an application on your VM that is blocked by the default security group. Please see our [Security Groups](http://fshead1:8080/ACC/140895578.html) page for information about modifying the inbound firewall rules for your VM.