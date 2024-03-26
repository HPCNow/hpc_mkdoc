ACC - Advanced Computing Center : Duo
==========================================

As of December 31, 2018, remote access to [acc.ohsu.edu](http://acc.ohsu.edu/) requires use of a Duo Mobile app or a Duo security token (aka, a "fob"). If you are already enrolled in Duo for other OHSU logins, there is no additional action for you to take. Once you are enrolled, the SSH server address depends on whether or not you are using the mobile app:

-   **acc.ohsu.edu** - This system uses automatic push and is only compatibly with the mobile app
-   **acc-alt.ohsu.edu** - This system allows either push or code entry from a security token

!!! note
    You are considered "remote" unless you are connected to the OHSU-Secure wifi network or a wired connection within OHSU.

To install the Duo Mobile app, please follow the [instructions available on O2](https://o2.ohsu.edu/information-technology-group/help-desk/it-help-pages/duo-enroll.cfm). If you need a security token, please request one from [your IT Contact](https://bridge.ohsu.edu/community/itc/SitePages/Find-an-ITC.aspx) as soon as possible; they can take up to two weeks to be delivered.

Additionally, logins using SSH key-based authentication have been disabled from the internet. In order to authenticate, you need to type your password, and then either respond to a Duo push or enter a duo passcode. If you have a need to multiplex many ssh-based connections at once, you might find our [Remote Access](https://wiki.ohsu.edu/display/HHT/Remote+Access) instructions useful for reducing the number of times you need to enter your password.