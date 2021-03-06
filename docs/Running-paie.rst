.. _running_paie:

Running the Watson Machine Learning (WML) Accelerator Software Install Module
=============================================================================

Overview
--------
The WML Accelerator software installation can be automated using the POWER-Up
software installer and the WML Accelerator Software Install Module. At current
time, the WMLA software installer only supports the licensed version of WMLA
running on Power hardware.

The WML Accelerator Software Install Module provides for rapid installation of
the WML Accelerator software to a homogeneous cluster of POWER8 or POWER9
servers.

The install module creates a web based software installation server on one of
the cluster nodes or another node with access to the cluster. The software
server is populated with repositories and files needed for installation of WML
Accelerator.

Once the software server is setup, installation scripts orchestrate the
software installation to one or more client nodes. Note that the software
installer node requires access to several open source repositories during the
'preparation' phase. During the preparation phase, packages which WML
Accelerator is dependent on are staged on the POWER-Up installer node. After
completion of the preparation phase, the installation requires no further
access to the open source repositories and can thus enable installation to
servers which do not have internet access.

Running POWER-Up software on one of the cluster nodes is supported. This will
"self-install" WML Accelerator on to the install along with the rest of the
cluster nodes at the same time. This eliminates the need for a dedicated
installer node but requires some additional controls to handle system reboots.
Rebooting is controlled via an Ansible variable, 'pup_reboot', that is set
automatically in the inventory. A global 'pup_reboot=True' is added to default
to original reboot behavior. If the installer node is included in the
inventory, a 'pup_reboot=True' host variable is automatically added to the
inventory (and anytime validation is called it will ensure this value is set,
preventing an override). Additional client nodes could also set
'pup_reboot=True' to prevent them from rebooting.

Support
-------
Questions regarding the WML Accelerator installation software, installation, or
suggestions for improvement can be posted on IBM's developer community forum at
https://developer.ibm.com/answers/index.html with the PowerAI tag.

Answered questions regarding PowerAI can be viewed at
https://developer.ibm.com/answers/topics/powerai/

For Advanced Users
------------------
User's experienced with the WMLA installation process may find the advanced
user instructions useful. :ref:`appendix_b`

Set up of the POWER-Up Software Installer Node
----------------------------------------------

POWER-Up Node  Prerequisites;

#. The POWER-Up software installer currently runs under RHEL 7.5 or above.

#. The user account used to run the POWER-Up software needs sudo privileges.

#. Enable access to the Extra Packages for Enterprise Linux (EPEL) repository.
   (https://fedoraproject.org/wiki/EPEL#Quickstart)

#. Enable the common, optional and extras repositories.

    # On POWER8::

       $ sudo subscription-manager repos --enable=rhel-7-for-power-le-rpms --enable=rhel-7-for-power-le-optional-rpms --enable=rhel-7-for-power-le-extras-rpms

    # On POWER9::

       $ sudo subscription-manager repos --enable=rhel-7-for-power-9-rpms --enable=rhel-7-for-power-9-optional-rpms --enable=–enable=rhel-7-for-power-9-extras-rpms

#. Insure that there is at least 16 GB of available disk space in the partition
   holding the /srv directory::

    $ df -h /srv

#. Install the version of POWER-Up software appropriate for the version of WML
   Accelerator you wish to install. The versions listed in the table below are
   the versions tested with the corresponding release of WML Accelerator or
   prior release of PowerAI Enterprise;

.. csv-table::
   :header: "WML Accelerator Release", "POWER-Up software installer vs", "Notes", "EOL date"

   "1.1.2", "software-install-b2.12", "Support for installation of PAIE 1.1.2"
   "1.2.0", "wmla120-1.0.0", "Support for installation of WMLA 1.2.0"
   "1.2.0", "wmla120-1.0.1", "Support for installation of WMLA 1.2.0"
   "1.2.0", "wmla120-1.0.2", "Validation checks. Install WMLA to installer node. Operating system install."
   "1.2.1", "wmla121-1.0.0", "Support for installation of WMLA 1.2.1"

From your home directory install the POWER-Up software and initialize the
environment. For additional information see :ref:`installing`::

    $ sudo yum install git

    $ git clone https://github.com/ibm/power-up -b wmla121-1.0.0

    $ cd power-up

    $ ./scripts/install.sh

    $ source scripts/setup-env

**NOTES:**

- The latest functional enhancements and defect fixes can be obtained by
  cloning the software installer without specifying the branch release.
  Generally, you should use a release level specified in the table above unless
  you are experiencing problems.::

    git clone https://github.com/ibm/power-up

- Multiple users can install and use the WMLA installer software, however there
  is only one software server created and there are no safeguards built in to
  protect against concurrent modifications of the software server content, data
  files or client nodes.

- Each user of the WMLA installer software must install the POWER-Up software
  following the steps above.


Installation of WML Accelerator
----------------------------------

Installation of the WML Accelerator software involves the following steps;

#. Preparation of the client nodes

#. Preparation of the software server

#. Initialization of the cluster nodes

#. Installation of software on the cluster nodes


Preparation of the client nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before beginning automated installation, you should have completed the 'Setup
for automated installer steps' at
https://www.ibm.com/support/knowledgecenter/SSFHA8_1.2.1/wmla_auto_install_setup.html
PowerUp includes a simple to use operating system installation utility which
can be used to install operating systems if needed. See :ref:`running_os`

Before proceeding with preparation of the POWER-Up server, you will need to
gather the following information;

- Fully qualified domain name (FQDN) for each client node

- Userid and password or private ssh key for accessing the client nodes. Note
  that for running an automated installation, the same user id and password
  must exist on all client nodes and must be configured with sudo access. The
  PowerUp software installer uses passwordless ssh access during the install.
  If an ssh key is not available one will be generated and distributed to all
  the cluster nodes.

Copy or Extract the WMLA software packages onto the PowerUp installation node.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Before beginning installation of WML Accelerator, the binary file containing
the licensed or eval version of the wmla software needs to be copied or
downloaded onto the installer node.
The files can be copied anywhere, but the POWER-Up software can locate them
quicker if the files are under a subdirectory of one of the /home/ directories
or the /root directory.

-  WML Accelerator binary file. (ibm-wmla-\*_\*.bin)

Extract WMLA. Assuming the WMLA binary is in /home/user/wmla121bin::

    cd /home/user/wmla121bin
    bash ibm-wmla-1.2.1_ppc64le.bin

In addition to the Red Hat and EPEL repositories, the POWER-Up software server
needs access to the following repositories during the preparation phase;

-  IBM AI repo
-  Cuda driver
-  Anaconda

These can be accessed using the public internet (URL's are 'built-in') or from
an alternate web site such as an intranet mirror repository, another POWER-Up
server or from a mounted USB key.

**NOTES:**

- Extraction and license acceptance of WML Accelerator must be performed on the
  same hardware architecture as the intended target nodes. If you are running
  the POWER-Up installer software on an x_86 node, you must first extract the
  files on an OpenPOWER node and then copy all of the extracted contents to the
  POWER-Up installer node.

- Red Hat dependent packages are unique to Power8, Power9 and x86 and must be
  downloaded on the target architecture. If you are running the WML Accelerator
  installer on a different architecture than the architecture of your cluster
  nodes, you must download the Red Hat dependent packages on a node of the same
  architecture as your cluster and then copy them to a directory on the
  installer node. A utility script is included to facilitate this process. To
  use the script, insure you have ssh access with sudo privileges to an
  appropriate node which has a subscription to the Red Hat 'common', 'optional'
  and 'extras' channels. (One of the cluster nodes or any other suitable node
  can be used for this purpose). To run the script from the power-up directory
  on the installer node::

    ./software/get-dependent-packages.sh userid hostname arch

The hostname can be a resolvable hostname or ip address. The
get-dependent-packages script will download the required packages on the
specified Power node and then move them to the ~/tempdl directory on the
installer node. After running the script, run/rerun the --prep phase of
installation. For dependent packages, choose option D (Create from files in a
local Directory) and enter the full absolute path to the tempdl/ directory. To
run the WMLA installer and refresh just the dependencies repo, execute the
following::

    pup software --step dependency_repo --prep wmla*

**Status of the Software Server**

At any time, you can check the status of the POWER-Up software server by
running::

    $ pup software --status wmla*


To use the automated installer with the evaluation version of WML Accelerator,
include the --eval switch in all pup commands. ie::

    $ pup software --status --eval wmla*

Note: The POWER-Up software installer runs python installation modules.
Inclusion of the '.py' in the software module name is optional. ie For WML
Accelerator version 1.2.1, wmla121 or wmla121.py are both acceptable.

**Hint: The POWER-Up command line interface supports tab autocompletion.**

Preparation is run with the following POWER-Up command::

    $ pup software --prep wmla*

Preparation is interactive and may be rerun if needed. Respond to the prompts
as appropriate for your environment. Note that the EPEL, Cuda, dependencies
and Anaconda repositories can be replicated from the public web sites or from
alternate sites accessible on your intranet environment or from local disk (ie
from a mounted USB drive). Most other files come from the local file system.


Initialization of the Client Nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
During the initialization phase, you will need to enter a resolvable hostname
for each client node in a cluster inventory file. If installing WMLA to the
installer node, it also must be entered in the cluster inventory file.
Optionally you may select from an ssh key in your .ssh/ directory. If one is
not available, an ssh key pair will be automatically generated. You will also
be prompted for a password for the client nodes. Initialization will set up all
client nodes for installation.  Optionally during init clients you may run
validation checks against all cluster nodes. Validation checks validate the
following;

-  hostnames are resolvable to FQDN for all nodes in the cluster
-  Firewall ports are enabled (or firewall is disabled)
-  Shared storage directories are properly mounted and appropriate permission
   bits set
-  Time is synchronizes across the cluster nodes
-  Storage and memory resources are adequate on all cluster nodes
-  Appropriate OS is installed on all cluster nodes


To initialize the client nodes and enable access to the POWER-Up software
server::

    $ pup software --init-clients wmla*

NOTES:

- During the initialization phase you will be required to create an inventory
  list of the nodes being installed. An editor window will be opened
  automatically to enable this.
- During the initialization phase you will be required to provide values for
  certain environment variables needed by Spectrum Conductor with Spark and
  Spectrum Deep Learning Impact. An editor window will be automatically opened
  to enable this.
- The CLUSTERADMIN variable will be automatically populated with the cluster
  node userid provided during the cluster inventory creation.
- The DLI_SHARED_FS environment variable should be the full absolute path to
  the shared file system mount point. (eg DLI_SHARED_FS: /mnt/my-mount-point).
  The shared file system and the client node mount points need to be configured
  prior to installing WML Accelerator.
- If left blank, the DLI_CONDA_HOME environment variable will be automatically
  populated. If entered, it should be the full absolute path of the install
  location for Anaconda. (ie DLI_CONDA_HOME: /opt/anaconda3)
- Initialization of client nodes can be rerun if needed.

Installation
~~~~~~~~~~~~
To install the WML Accelerator software and prerequisites::

    $ pup software --install wmla*

NOTES:

-  Installation of WML Accelerator can be rerun if needed.

After completion of the installation of the WML Accelerator software, you must
configure Spectrum Conductor Deep Learning Impact and apply any outstanding
fixes.
Go to https://www.ibm.com/support/knowledgecenter/SSFHA8, choose your version
of WML Accelerator and then use the search bar to search for ‘Configure IBM
Spectrum Conductor Deep Learning Impact’.

Additional Notes
~~~~~~~~~~~~~~~~

You can browse the content of the POWER-Up software server by pointing a web
browser at the address  of POWER-Up installer node. Individual files can be
copied to client nodes using wget or curl if desired.

**Dependent software packages**
The WML Accelerator software is dependent on additional open source software
that is not shipped with WML Accelerator. Some of these dependent packages are
downloaded to the POWER-Up software server from enabled yum repositories during
the preparation phase and are subsequently available to the client nodes during
the install phase. Additional software packages can be installed in the
'dependencies' repo on the POWER-Up software server by listing them in the
power-up/software/dependent-packages.list file. Entries in this file can be
delimited by spaces or commas and can appear on multiple lines. Note that
packages listed in the dependent-packages.list file are not automatically
installed on client nodes unless needed by the PowerAI software. They can be
installed on a client node explicitly using yum on the client node (ie yum
install pkg-name). Alternatively, they can be installed on all client nodes at
once using Ansible (run from within the power-up directory)::

    $ ansible all -i playbooks/software_hosts --become --ask-become-pass -m yum -a "name=pkg-name"

or on a subset of nodes (eg the master nodes) ::

    $ ansible master -i playbooks/software_hosts --become --ask-become-pass -m yum -a "name=pkg-name"

Uninstalling the POWER-Up Software
----------------------------------
To uninstall the POWER-Up software and remove the software repositories, follow
the instructions below;

#. Identify platform to remove::

    $ PLATFORM="ppc64le"

#. Stop and remove the nginx web server::

    $ sudo nginx -s stop
    $ sudo yum erase nginx -y

#. If you wish to remove the http service from the firewall on this node::

    $ sudo firewall-cmd --permanent --remove-service=http
    $ sudo firewall-cmd --reload

#. If you wish to stop and disable the firewall service on this node::

    $ sudo systemctl stop firewalld.service
    $ sudo systemctl disable firewalld.service

#. Remove the yum.repo files created by the WMLA installer::

    $ sudo rm /etc/yum.repos.d/cuda.repo
    $ sudo rm /etc/yum.repos.d/nginx.repo

#. Remove the software server content and repositories (replace
   'wmla121-ppc63le' with current software module and architecture)::

    $ sudo rm -rf /srv/pup/wmla121-ppc64le/anaconda
    $ sudo rm -rf /srv/pup/wmla121-ppc64le/wmla-license
    $ sudo rm -rf /srv/pup/wmla121-ppc64le/spectrum-dli
    $ sudo rm -rf /srv/pup/wmla121-ppc64le/spectrum-conductor
    $ sudo rm -rf /srv/pup/wmla121-ppc64le/repos

#. Remove the yum cache data depending on Computer Architecture::

    $ sudo rm -rf /var/cache/yum/${PLATFORM}/7Server/cuda/
    $ sudo rm -rf /var/cache/yum/${PLATFORM}/7Server/nginx/


#. Uninstall the PowerUp Software
    - Assuming you installed from your home directory, execute::

        $ sudo rm -rf ~/power-up
