ACC - Advanced Computing Center : Exastack: VM Configuration with Cloud-Init
============================================================================

Most Linux distributions will allow you to pre-configure a new virtual machine (VM) using [cloud-init scripting](https://cloudinit.readthedocs.io/en/latest/reference/index.html). You may find pre-configuration very useful, especially when you often create virtual machines needing various user accounts, package installations, or customized configuration files. Each distribution (e.g., CentOS, Rocky, or Ubuntu) needs a slightly different cloud-init script, so please consider the scripts below to be guides for your work rather verbatim examples.

### Whitespace

The cloud-init interpreter reads [YAML](http://yaml.org/), a text-based data format. It's worth noting that YAML treats leading whitespace as significant. The indentations in the examples below aren't just visual hygiene; they're necessary for parsing. If you're unsure of your formatting, you can use an online parser like [YAML Lint](http://www.yamllint.com/) to test all or part of your markup.

### Usage
You can use the cloud-init configurations below in one of two ways:

1.  in the Configuration tab when you launch a VM [via the web interface](03-Launch-a-Virtual-Machine.md)
2.  via the `--user-data` option when you launch a VM [via the command line](01-Command-Line-Operation.md)

### CentOS


My initial starting point was the [official CentOS 7 image](http://cloud.centos.org/centos/7/images/). My goals for the customized VM were

-   create myself a functioning user account with sudo privileges
-   update the local package set
-   install and activate PostgreSQL and Apache

In the OpenStack Dashboard GUI, I launched a new instance using the CentOS image. As part of the process, I pointed the launcher at my configuration file:

``` yaml
#cloud-config
users:
- name: heinlein
  gecos: Paul Heinlein
  sudo: ['ALL=(ALL) NOPASSWD:ALL']
  groups: wheel
  ssh-authorized-keys:
  - ssh-rsa AAAAB3NzaC1...1234 work-key
  - ssh-rsa AAAAB3NzaC1...5678 home-key

# update all installed packages
package_upgrade: true

# install Apache and PostgreSQL
packages:
- httpd
- php
- postgresql
- postgresql-server

# make sure Apache and PostgreSQL both start at boot-time
runcmd:
- [ postgresql-setup, initdb ]
- [ systemctl, enable, httpd.service ]
- [ systemctl, start, httpd.service ]
- [ systemctl, enable, postgresql.service ]
- [ systemctl, start, postgresql.service ]

```

Once the VM was fully launched, and I'd assigned it a floating IP address, I was able to login using my username and SSH keys. The CentOS approach to cloud-init will also work with Rocky Linux and some versions of Fedora Linux.

### Ubuntu


Things were slightly different when using the official [Ubuntu cloud images](http://cloud-images.ubuntu.com/). OpenStack will correctly install your SSH key into the root account, but it will get configured so that you cannot login as root. Setting up a user account with sudo privileges is absolutely necessary to use the Ubuntu image.

My goals for the Ubuntu image were simple:

-   update local packages at installation time
-   install Apache and aptitude (which I prefer to apt-get)
-   make sure my user account had sudo privileges.

``` yaml
#cloud-config
users:
- name: heinlein
  gecos: Paul Heinlein
  sudo: ['ALL=(ALL) NOPASSWD:ALL']
  shell: /bin/bash
  groups: sudo
  ssh-authorized-keys:
  - ssh-rsa AAAAB3NzaC1...1234 work-key
  - ssh-rsa AAAAB3NzaC1...5678 home-key

package_upgrade: true

packages:
- apache2
- aptitude

```

### For Further Reading

-   [Cloud config examples](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)
-   [What is Cloud-Init?](https://phoenixnap.com/kb/what-is-cloud-init)