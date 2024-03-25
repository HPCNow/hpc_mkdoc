ACC - Advanced Computing Center : Security Groups
===========================================================


Security groups is an essential tool to understand in order for you to ensure the security and integrity of your instances, as well as making your application work correctly and as intended.

### What is a Security Group?

The very short answer to this question is that a security group is a firewall. A security group contains a number of rules that define which traffic is allowed into (ingress) and out from (egress) your virtual machines (instances). The rules within a security group may allow only a single IP address/port combination, or they may open up completely and thereby negating the effect and protection of a firewall. It is utterly important that you understand what your security groups and the rules do and how they work, in order to properly secure your instances and applications.

### The default Security Group

A newly created Exastack project will have a security group called "default". This security group is unique for this project, and all projects will have a security group called "default." This security group cannot be deleted. To find it in the dashboard GUI, navigate to **Project** > **Network** > **Security Groups** and click on the Manage Rules button toward the right side of the page.

The notation can be slightly confusing, especially for the standard Ingress rules. Any rule that lists a "remote security group" only applies to machines covered by the listed group. Your default security group has two rules that allow inbound traffic -- but only from machines also covered by your "default" group.

If left with just those ingress rules, Exastack security groups would close off traffic to your instances to all hosts except those on your project network. ACC typically modifies the default rules to allow inbound SSH, HTTP, and HTTPS traffic. You can remove or add to those modifications as you see fit.

So an ACC-modified default security group will have seven rules:

| Direction | Ether Type | IP Protocol | Port Range | Remote IP Prefix | Remote Security Group | Description |
|-----------|------------|-------------|------------|------------------|-----------------------|-------------|
| Egress    | IPv4       | Any         | Any        | 0.0.0.0/0        | -                     |  -          |
| Egress    | IPv6       | Any         | Any        | ::/0             | -                     |  -          |
| Ingress   | IPv4       | Any         | Any        | - | default      | -                     |             |   
| Ingress   | IPv4       | TCP         | 22         | 0.0.0.0/0        | -                     | SSH         |
| Ingress   | IPv4       | TCP         | 80         | 0.0.0.0/0        | -                     | HTTP        |
| Ingress   | IPv4       | TCP         | 443        | 0.0.0.0/0        | -                     | HTTPS       |
| Ingress   | IPv6       | Any         | Any        | -                | default               |  -          |


In summary, the "default" security group does the following:

-   Allow all outgoing (egress) traffic. The instance may initiate contact and communicate with any host at OHSU or on the internet, on any port using any protocol.
-   Allow all incoming (ingress) traffic from other instances within the same project, if they also have the "default" security group applied.
-   Allow all incoming (ingress) TCP traffic from OHSU on ports 22 (ssh), 80 (http), and 443 (https).
-   Deny all traffic not specifically allowed above.

Many instances will run services that require additional firewall rules. Our recommendation is to create new security groups with the required ruleset, and apply those **in addition to** the default security group.