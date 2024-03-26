ACC - Advanced Computing Center : Remote Access
===================================================


For people who are located off campus, ACC provides remote access via an ssh gateway: **acc.ohsu.edu**.
!!! note
    Those located on the Research Data Network (RDN) must use **acc-external.ohsu.edu.**

### Access Requirements


There are two major requirements to access the ACC ssh gateway:

1.  An ACC account
2.  Multi-factor authentication

#### ACC Account


OHSU users do not have ACC accounts by default. ACC accounts are granted in conjunction with projects hosted within the [Advanced Computing Center](https://www.ohsu.edu/advanced-computing-center), including Exacloud.

Contact ACC by sending an email to <acc@ohsu.edu> to initiate a request.

#### Multi-factor Authentication


Assuming your account is approved for remote access based on the above, it is also necessary for your account to be protected by multi-factor authentication in the form of Duo. This is necessary to help ensure that access to your account and ACC systems is secure in the event that your password is stolen by an outside entity.

Please note that Duo requires either a smartphone application or a hardware token. You must have one of these present with you when logging in to authenticate successfully. See the [Duo enrollment guide](https://o2.ohsu.edu/information-technology-group/help-desk/it-help-pages/duo-enroll.cfm) for instructions on enrolling your account.

### Remote Access Instructions


1.  Connect your ssh client to **acc.ohsu.edu**
    - [x]  RDN users connect to **acc-external.ohsu.edu**
    - [x]  Users without the Duo Mobile app connect to **acc-alt.ohsu.edu**
    - [x]  Users without Duo Mobile on RDN connect to **acc-external-alt.ohsu.edu**
2.  Enter your OHSU username (do not append @ohsu.edu).  Some examples:
    - [x]  `ssh username@acc.ohsu.edu`
    - [x]  `ssh -l username acc.ohsu.edu`
3.  Enter your OHSU password (ssh keys are not accepted for authentication)
4.  You will need to respond to a Duo push from your smartphone, or enter a code (if using the alternative host)

ACC SSH gateways have the following key fingerprints:

=== "rsa"
    -   SHA256: `z6XxFHZa7WiRFQ/YpYhUTn4O/3O+IO1Yjzgs+J1H5r0`
    -   MD5: `84:75:7e:9a:5b:a6:0c:a4:48:14:ec:b6:b3:32:af:95`
=== "ecdsa"
    -   SHA256: `Z/d7YYI7zhlJU6hHzkr5brOejfD9j/eS5Ep+bzH2du8`
    -   MD5: `df:88:86:1d:d2:1e:6b:85:44:f4:4e:6a:53:55:14:9c`
===  "ed25519"
    -   SHA256: `7q58ewgw1y5i3vThj0Af0u5A3epxkiSAVmJISXaCGoY`
    -   MD5: `30:97:5f:f1:32:7f:67:41:ab:77:18:08:d9:1d:4c:d5`

### SSH ControlMaster


If your ssh workflow includes making many different connections, it can be tiresome to have to type your password and Duo passcode each time you connect. The ssh client **ControlMaster** option can be configured so that you only need to authenticate once at the beginning of each work session. Subsequent connections can make use of the socket file created at **ControlPath** (which must be placed in a secure file location, preferrably in `~/.ssh/`).

To get started, add an entry to your `~/.ssh/config` file:


``` sh title="~/.ssh/config"
Host accgw 
  User username
  Port 22
  Hostname acc.ohsu.edu
  ControlMaster auto
  ControlPath ~/.ssh/cm-%r@%h:%p
```

If you already have an acc entry in your ssh config, just add the **ControlMaster** and **ControlPath** options. See `man ssh_config` for additional ssh client configuration options.

When using the above configuration, you will automatically receive a Duo push notification in the Duo Mobile app after successfully entering your password. Once you respond to the push, you will be logged in.


``` sh
~ > ssh username@accgw
(username@acc-ssh5) Password:
Autopushing login request to phone...
Success. Logging you in...

  acc-ssh5.ohsu.edu
  * RedHat 8.8
-----------------------------------------------------------------
NOTICE: This is a private system.

All users of this system are subject to having their activities
audited. This system is to be used only for OHSU business, and
access requires explicit written authorization.

Unauthorized access -- or attempts to damage data, programs, or
equipment -- may violate applicable law and could result in
corrective action or criminal prosecution.
-----------------------------------------------------------------

username@acc-ssh5 ~ $
```

If you do not have the Duo Mobile app, you'll need to use **acc-alt.ohsu.edu** as the "Hostname" in your ssh config file. In that case, after successfully authenticating, you'll be prompted to enter a passcode, or send a push notification.

Once you are connected, in a different terminal you can perform actions which use ssh transport, e.g. scp, rsync, and even new ssh connections:

``` sh
$ scp demo.sh accgw:demo.sh
demo.sh                             100%  225    38.5KB/s   00:00
$ rsync -av test/ accgw:test/
sending incremental file list
./
a
b

sent 295 bytes  received 117 bytes  824.00 bytes/sec
total size is 0  speedup is 0.00
$ ssh accgw
username@acc-ssh5 ~ $ exit
```

These will occur without requiring authentication. Then once the initial connection is closed, the next connection will require authentication again.

#### Getting to Other Internal Hosts


You can use your ControlMaster setup to log directly in to other OHSU hosts from your off-site location.

Add a stanza like this to your `~/.ssh/config` file:

``` sh title="~/.ssh/config"
Host *.ohsu.edu
  CheckHostIP no
  ProxyCommand ssh accgw -W %h:%p

```

!!! tip
    If you take your workstation to and from campus, you'll probably want to comment that stanza out when you're on campus.

In the ProxyCommand line, make sure that where the example says "accgw," you use the token that follows "Host" in the stanza that sets up the ControlMaster.

Then you can ssh directly to, say, exacloud.ohsu.edu using a simple set of steps:

1.  In one terminal window, run `ssh accgw` as listed above. (Don't run `ssh acc.ohsu.edu`; just use the short hostname or whatever token follows "Host" in the stanza that sets up the ControlMaster.)
2.  In a second terminal window, run `ssh exacloud.ohsu.edu`
3.  In that second window, complete your Exacloud work.
4.  Log out of exacloud.
5.  Log out of acc.

!!! note
    You aren't limited to one exacloud session. You could have several terminals running login sessions to exacloud (or other OHSU hosts on which you have SSH login permissions) at the same time.