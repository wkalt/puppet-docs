---
layout: default
title: "PE 3.3 » Installing » System Requirements"
subtitle: "System Requirements and Pre-Installation"
canonical: "/pe/latest/install_system_requirements.html"
---

Before installing Puppet Enterprise:

* Ensure that your nodes are running a supported operating system.    
* Ensure that your puppet master and console servers are sufficiently powerful (see the [hardware section](#hardware-requirements) section below).
* Ensure that your network, firewalls, and name resolution are configured correctly and all target servers are communicating.
* Plan to install the puppet master server before the console server, and the console server before any agent nodes. If you are separating components, install them in this order:  
    1. Puppet Master
    2. PuppetDB and PostgreSQL
    3. Console
    4. Agents

Operating System
-----

Puppet Enterprise 3.3 supports the following systems:

Operating system             | Version(s)                              | Arch          | Component(s)
-----------------------------|-----------------------------------------|---------------|----------------------------
Red Hat Enterprise Linux     | 4, 5, 6, & 7                                   | x86 & x86\_64 | all (RHEL 4 supports agent only)
CentOS                       | 4, 5, & 6                                   | x86 & x86\_64 | all (CentOS 4 supports agent only)
Ubuntu LTS                   | 10.04, 12.04, & 14.04                           | i386 & amd64  | all
Debian                       | Squeeze (6) & Wheezy (7)                | i386 & amd64  | all
Oracle Linux                 | 4, 5 & 6                                   | x86 & x86\_64 | all (Oracle Linux 4 supports agent only)
Scientific Linux             | 4, 5 & 6                                   | x86 & x86\_64 | all (Scientific Linux 4 supports agent only)
SUSE Linux Enterprise Server | 11 (SP1 and later)                      | x86 & x86\_64 | all
Solaris                      | 10 (Update 9 or later) & 11          | SPARC & i386   | agent
Microsoft Windows            | 2003, 2003R2, 2008, 2008R2, 7, 8, 2012, & 2012R2| x86 & x86\_64 | agent
AIX                          | 5.3, 6.1, & 7.1                         | Power         | agent
Mac OS X                     | Mavericks (10.9)                        | x86_64         | agent

>**Note**: Some operating systems require an active subscription with the vendor's package management system to install dependencies, such as Red Hat Network. 

>**Note**: In addition, upgrading your OS while PE is installed can cause problems with PE. To perform an OS upgrade, you’ll need to uninstall PE, perform the OS upgrade, and then reinstall PE as follows:
>
>1. [Back up](./maintain_backup_restore.html#back-up-your-database-and-puppet-enterprise-files) your databases and other PE files.
>
>2. Perform a complete [uninstall](./install_uninstalling.html) (including the -p -d uninstaller option).
>
>3. Upgrade your OS.
>
>4. [Install PE](/pe/latest/install_basic.html).
>
>5. [Restore](./maintain_backup_restore.html#restore-your-database-and-puppet-enterprise-files) your backup.

Hardware Requirements
-----

Puppet Enterprise's hardware requirements depend on the components a machine performs.

For the puppet master, PE console, PuppetDB and database support, and any agent nodes, we recommend that your hardware meets the following requirements.

* At least four processor cores per node
* At least 4 GB RAM per node
* Very accurate timekeeping 
* For `/var/`, at least 1 GB of free space for each PE component on a given node
* For PE-installed PostgreSQL, `/opt/` requires at least 100 GB of free space for data gathering
* For no PE-installed PostgreSQL, `/opt/` needs at least 1 GB of disk space available

Supported Browsers
-----

The following browsers are supported for use with the console:

* Chrome: Current version, as of release
* Firefox: Current version, as of release
* Internet Explorer: 9, 10, and 11
* Safari: 7


System Configuration
-----

Before installing Puppet Enterprise at your site, you should make sure that your nodes and network are properly configured.

### Timekeeping

We recommend using NTP or an equivalent service to ensure that time is in sync between your puppet master and any puppet agent nodes. If time drifts out of sync in your PE infrastructure, you may encounter issues such as nodes disappearing from live manangement in the console. A service like NTP ([available as a Puppet Labs supported module](https://forge.puppetlabs.com/puppetlabs/ntp)) will ensure accurate timekeeping.

### Name Resolution

* Decide on a preferred name or set of names agent nodes can use to contact the puppet master server.
* Ensure that the puppet master server can be reached via domain name lookup by all of the future puppet agent nodes at the site.

You can also simplify configuration of agent nodes by using a CNAME record to make the puppet master reachable at the hostname `puppet`. (This is the default puppet master hostname that is automatically suggested when installing an agent node.)

### Firewall Configuration

Configure your firewalls to accommodate Puppet Enterprise's network traffic. In brief: you should open up ports **8140, 8081, 61613, and 443.** The more detailed version is:

* If you are installing PE using the web-based installer, ensure port **3000** is open. You can close this port when the installation is complete.
* All agent nodes must be able to send requests to the puppet master on ports **8140** (for Puppet) and **61613** (for orchestration).
* The puppet master must be able to accept inbound traffic from agents on ports **8140** (for Puppet) and **61613** (for orchestration).
* Any hosts you will use to access the console must be able to reach the console server on port **443,** or whichever port you specify during installation. (Users who cannot run the console on port **443** will often run it on port **3000**.)
* If you will be invoking orchestration commands from machines other than the puppet master, they will need to be able to reach the master on port **61613.** (**Note:** enabling other machines to invoke orchestration actions is possible but not supported in this version of Puppet Enterprise.)
* If you will be running the console and puppet master on separate servers, the console server must be able to accept traffic from the puppet master (and the master must be able to send requests) on ports **443** and **8140.** The console server must also be able to send requests to the puppet master on port **8140,** both for retrieving its own catalog and for viewing archived file contents.
* PuppetDB needs to accept connections on port **8081**, and the puppet master and PE console need to be able to do outbound traffic on **8081**. 

### Dependencies and OS Specific Details

This section details the packages that are installed from the various OS repos.  Unless you do not have internet access, you shouldn't need to worry about installing these manually, they will be set up during PE installation.

#### PostgreSQL Requirement

If you will be using your own instance of PostgreSQL (as opposed to the instance PE can install) for the console and PuppetDB, it must be version 9.1 or higher.

#### OpenSSL Requirement

OpenSSL is a dependency required for PE. For Solaris 10 and all versions of RHEL, Debian, Ubuntu, Windows, and AIX nodes, OpenSSL is included with PE; for all other platforms it is installed directly from the system repositories.

***Centos***

All Nodes    | Master Nodes | Console Nodes | Console/Console DB Nodes | Cloud Provisioner Nodes
-------------|--------------|---------------|--------------------------|------------------------
pciutils     | apr          | apr           | libjpeg                  | libxslt
system-logos | apr-util     | apr-util      |                          | libxml2
which        | curl         | curl          |                          |
libxml2      | mailcap      | mailcap       |                          |
dmidecode    | libjpeg      |               |                          |
net-tools    | libtool-ltdl | libtool-ltdl  |                          |
virt-what    | unixODBC     | unixODBC      |                          |

<br>

***RHEL***

All Nodes           | Master Nodes           | Console Nodes          | Console/Console DB Nodes | Cloud Provisioner Nodes
--------------------|------------------------|------------------------|--------------------------|------------------------
pciutils            | apr                    | apr                    | libjpeg                  | libxslt
system-logos        | apr-util               | apr-util               |                          | libxml2
which               | apr-util-ldap (RHEL 6) | curl                   |                          |
libxml2             | curl                   | mailcap                |                          |
dmidecode           | mailcap                | apr-util-ldap (RHEL 6) |                          |
net-tools           | libjpeg                |                        |                          |
cronie (RHEL 6)     | libtool-ltdl (RHEL 7)  | libtool-ltdl (RHEL 7)                       |                          |
vixie-cron (RHEL 4, 5) | unixODBC (RHEL 7)   | unixODBC (RHEL 7)                       |                          |
virt-what           |                        |                        |                          |

  <br>

***SLES***

All Nodes | Master Nodes   | Console Nodes | Console/Console DB Nodes | Cloud Provisioner Nodes
----------|----------------|---------------|--------------------------|------------------------
pciutils  | libapr1        | libapr1       | libjpeg                  | libxml2
pmtools   | libapr-util1   | libapr-util1  |                          |
cron      | libxslt        | curl          |                          |
libxml2   | curl           |               |                          |
net-tools | libjpeg        |               |                          |
libxslt   | db43           | db43          |                          |
          | unixODBC       | unixODBC
        
 <br>

***Debian***

All Nodes     | Master Nodes            | Console Nodes           | Console/Console DB Nodes | Cloud Provisioner Nodes
--------------|-------------------------|-------------------------|--------------------------|------------------------
pciutils      | file                    | file                    | libjpeg62                | libxslt1.1
dmidecode     | libmagic1               | libmagic1               | libxml2-dev (Debian 7)   | libxml2
cron          | libpcre3                | libpcre3                | locales-all   (Debian 7) |
libxml2       | curl                    | curl                    |                          |
hostname      | perl                    | perl                    |                          |
libldap-2.4-2 | mime-support            | mime-support            |                          |
libreadline5  | libapr1                 | libapr1                 |                          |
virt-what     | libcap2                 | libcap2                 |                          |
              | libaprutil1             | libaprutil1             |                          |
              | libaprutil1-dbd-sqlite3 | libaprutil1-dbd-sqlite3 |                          |
              | libaprutil1-ldap        | libaprutil1-ldap        |                          |
              | libjpeg62               | libcurl3 (Debian 7)     |                          |
              | libcurl3 (Debian 7)     | libxml2-dev (Debian 7)  |                          |
              | libxml2-dev (Debian 7)  |                 

<br>

***Ubuntu***


All Nodes     | Master Nodes            | Console Nodes           | Console/Console DB Nodes | Cloud Provisioner Nodes
--------------|-------------------------|-------------------------|--------------------------|------------------------
pciutils      | file                    | file                    | libjpeg62                | libxslt1.1
dmidecode     | libmagic1               | libmagic1               |                          | libxml2
cron          | libpcre3                | libpcre3                |                          |
libxml2       | curl                    | curl                    |                          |
hostname      | perl                    | perl                    |                          |
libldap-2.4-2 | mime-support            | mime-support            |                          |
libreadline5  | libapr1                 | libapr1                 |                          |
virt-what     | libcap2                 | libcap2                 |                          |
              | libaprutil1             | libaprutil1             |                          |
              | libaprutil1-dbd-sqlite3 | libaprutil1-dbd-sqlite3 |                          |
              | libaprutil1-ldap        | libaprutil1-ldap        |                          |
              | libjpeg62               |                         |                          |

<br>

***AIX***

In order to run the puppet agent on AIX systems, you'll need to ensure the following are installed *before* attempting to install the puppet agent:

* bash
* zlib
* readline

All [AIX toolbox packages](http://www-03.ibm.com/systems/power/software/aix/linux/toolbox/alpha.html) are available from IBM.

To install the packages on your selected node directly, you can run `rpm -Uvh` with the following URLs (note that the RPM package provider on AIX must be run as root):

 * ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/bash/bash-3.2-1.aix5.2.ppc.rpm
 * ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/zlib/zlib-1.2.3-4.aix5.2.ppc.rpm
 * ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/readline/readline-6.1-1.aix6.1.ppc.rpm (AIX 6.1 and 7.1 *only*)
 * ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/readline/readline-4.3-2.aix5.1.ppc.rpm (AIX 5.3 *only*)

*Note:* if you are behind a firewall or running an http proxy, the above commands may not work. Instead, use the link above to find the packages you need.

*Note:* GPG verification will not work on AIX, the RPM version used by AIX (even 7.1) is too old. The AIX package provider doesn't support package downgrades (installing an older package over a newer package). Avoid using leading zeros when specifying a version number for the AIX provider (i.e., use `2.3.4` not `02.03.04`).

The PE AIX implementation supports the NIM, BFF, and RPM package providers. Check the [Type Reference](/references/3.4.latest/type.html#package) for technical details on these providers.

***Solaris***

Solaris support is agent only. 

For Solaris 10, the following packages are required:

  * SUNWgccruntime
  * SUNWzlib 
  * In some instances, bash may not be present on Solaris systems. It needs to be installed before running the PE installer. Install it via the media used to install the OS or via CSW if that is present on your system. (CSWbash or SUNWbash are both suitable.)

For Solaris 11 the following packages are required:

  * system/readline
  * system/library/gcc-45-runtime
  * library/security/openssl

These packages are available in the Oracle Solaris release repository (enabled by default on Solaris 11). The PE installer will automatically install them; however, if the release repository is not enabled, the packages will need to be installed manually.

* * *

Next Steps

* To install Puppet Enterprise on \*nix nodes, continue to [Installing Puppet Enterprise](./install_basic.html).
* To install Puppet Enterprise on Windows nodes, continue to [Installing Windows Agents](./install_windows.html).
