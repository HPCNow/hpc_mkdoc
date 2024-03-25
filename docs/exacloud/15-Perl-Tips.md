ACC - Advanced Computing Center : Perl Tips
=====================================================

### How can I install PERL modules on the cluster?

We won't be making any changes to the default OS system binaries and libraries beyond what is found in the CentOS repos for security updates.

We have built a perl binary in /opt/installed that can be used on the cluster in place of the system perl binaries. e.g. `#!/opt/installed/path/to/perl`

However, as the cluster is a shared resource we want to keep this install of perl "clean", with only those libraries installed that come with the source. The problem is that many of the softwares that are run on the cluster are sensitive to the versions of the libraries, modules, etc. It's a best practice at HPC clusters (and other shared computational platforms) to have the libraries and modules that are being used by a certain group to be installed in a location accessible to that group, but not in the way of other groups (which may depend on a different version of the same libraries and modules). This is the model we've been following for python, and we should do this for perl too.

To install and use perl modules in a common location, you will want to make use of PERL5LIB environment variable; e.g.

``` sh
 export PERL5LIB=/home/exacloud/lustre1/foo/perl/lib/perl5:$PERL5LIB
```

This will add the path to @INC.

And depending on how you install the modules (via cpan, via cpanminus, via source), you have to adjust the installation locations.

Here is some info:

-   [Installing Modules Perl](http://wiki.hpc.ufl.edu/doc/Installing_Perl_Modules)
-   [Perl](https://surfsara.nl/systems/lisa/software/perl)
-   [Comprehensive Perl Archive Network](http://shadow.cat/blog/matt-s-trout/but-i-cant-use-cpan/)