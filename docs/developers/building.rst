.. Copyright (C) 2021 Alex Doyle <adoyle@nvidia.com>
   Copyright (C) 2014,2015,2016,2017,2018 Curt Brune <curt@cumulusnetworks.com>
   Copyright (C) 2014 Pete Bratach <pete@cumulusnetworks.com>
   SPDX-License-Identifier:     GPL-2.0

Build Instructions
==================


Branches and Build Environment Compatibility
--------------------------------------------

Be aware: the master branch, from release
`2021.11 <https://github.com/opencomputeproject/onie/tree/2021.11>`_
on contains upgraded software components that break backwards compatibility
with most machine builds and require a Debian 10 build environment.
For clarity, a list of ONIE branches and build environments for all machine targets is maintained here:
`onie/build-config/scripts/onie-build-targets.json <https://github.com/opencomputeproject/onie/blob/master/build-config/scripts/onie-build-targets.json>`_

For some context about the upgrade process, see the `2021 ONIE Status Update <https://www.youtube.com/watch?v=8H4S7eSZWJ4>`_

Machine targets that have not yet been upgraded for the master branch, can still be built
from the `2021.08 release <https://github.com/opencomputeproject/onie/tree/2021.08>`_
branch or the Debian9BuildEnvironment tag by using a Debian 9 build environment.
This process can be made much easier by using a Docker container - see below.


Preparing an ONIE Build Environment
-----------------------------------
ONIE build environments can be either configured directly on the
host machine (see "Preparing a New Build Machine" below) or run
in `Docker <http://www.docker.com>`_ containers for systems that have
either docker.io (from Debian) or docker-ce (from Docker) installed.

The latter approach is preferred, as building ONIE will require build
environments based off of Debian 10 Buster for the 2021.11 release
on, or Debian 9 Stretch for backwards compatible builds before the
`2021.08 release <https://github.com/opencomputeproject/onie/tree/2021.08>`_,

Dedicated User Environment Docker Images
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since developing while logged in to a Docker container can be frustrating,
in that a developer's configuration isn't present, host file systems
have to be explicitly mounted to save files, etc, ONIE build images
are available with Dedicated User Environment `DUE <https://github.com/cumulusnetworks/DUE>`_

DUE is the recommended environment for building ONIE as
it has been used for debug and release generation since 2019.

This MIT licensed open source project can create Docker (or Podman) ONIE
build environments from different Debian releases, and preserves the
user's identity and home directory while running them, creating the
experience of very little changing from a developers native environment.
Additionally, users will be building in the same environment that has
been used to build ONIE releases, vastly simplifying the
process of debugging build issues.

While DUE is available via apt in Debian 11 Bullseye, the source code
`https://github.com/cumulusnetworks/DUE <https://github.com/cumulusnetworks/DUE>`_
can be run from its checkout directory on Red Hat and Ubuntu systems as well.

To get started with DUE, configuration instructions can be found at
`ONIE build environment creation example <https://github.com/CumulusNetworks/DUE/tree/master/templates/onie>`_, and there is a video tutorial available at this `YouTube link <https://www.youtube.com/watch?v=evzkiiRRIvw>`_.

Traditional Docker Image
^^^^^^^^^^^^^^^^^^^^^^^^

If you want to use Docker containers without DUE, the ONIE project
provides a Debian build environment as a Docker image.  For more
info, see the `Docker image README
<https://github.com/opencomputeproject/onie/tree/master/contrib/build-env>`_.

That is a good starting point, but for long term development you may
want to install the development packages natively on your system.
Have a look at the top-level ``Makefile`` and the
``$(DEBIAN_BUILD_HOST_PACKAGES)`` variable.  Then install the
corresponding packages for your distribution that provide the same
tools.


Preparing a New Build Machine
-----------------------------

The ONIE build environment requires a number of standard development
packages.

For a `Debian-based system <http://www.debian.org/>`_, a Makefile
target exists that installs the required packages on your build machine.

For other distributions, a Debian Docker image is available to use as
a starting point.

Debian Systems
^^^^^^^^^^^^^^

The ONIE project maintains a Makefile target for the current stable
version of Debian.  This target requires the use of ``sudo(8)``, since
package installation requires root privileges::

  $ cd build-config
  $ sudo apt-get update
  $ sudo apt-get install build-essential
  $ make debian-prepare-build-host


Preparing a New Build User Account
----------------------------------

The user account for compiling ONIE must have ``/sbin`` and ``/usr/sbin``
in the ``$PATH`` environment variable.  As an example, when using the
``bash`` shell add the following near the end of ``$HOME/.bashrc``::

  export PATH="/sbin:/usr/sbin:$PATH"

Machine Definition Files
------------------------

In order to compile ONIE for a particular platform, you need the
platform's machine definition, located in
``$ONIE_ROOT/machine/<vendor>/<vendor>_<model>``.

See the INSTALL file in ``machine/<vendor>/<vendor>_<model>`` for
additional information about a particular platform.

Cross-Compiler Toolchain
------------------------

The ONIE build process generates and uses a cross-compiling toolchain
based on `gcc <http://gcc.gnu.org/>`_ and `uClibc
<http://www.uclibc.org/>`_.  The `crosstool-NG
<http://crosstool-ng.org/>`_ project is used to manage the build
of the toolchain.

A number of packages are downloaded from the Internet by the ONIE
build process and cached for subsequent builds. You can set up your
own local mirror for these packages by setting up
``onie/build-config/local.make``.  See the sample file,
``onie/build-config/local.make.example``, and the ``ONIE_MIRROR`` and
``CROSSTOOL_ONIE_MIRROR`` variables for examples.

Also see the FAQ entry :ref:`cache_packages`.

Cross-Compiling ONIE
--------------------

To compile ONIE, first change directories to ``build-config`` and then
type ``make MACHINEROOT=../machine/<vendor> MACHINE=<vendor>_<model>
all``, specifying the target machine.  For example::

  $ cd build-config
  $ make -j4 MACHINEROOT=../machine/<vendor> MACHINE=<vendor>_<model> all

When complete, the following ONIE binaries are created in the ``build/images``
directory:

.. _onie_build_products:

.. csv-table:: ONIE Build Products
  :header: "File", "Purpose"
  :delim: |

  onie-<platform>-<revision>.bin | Raw binary, suitable for NOR flash programming
  onie-updater-<platform>-<revision> | ONIE updater, for use with the ONIE update mechanism

Installing the ONIE Binary
--------------------------

See the ``INSTALL`` file in ``machine/<platform>`` for additional information
about how to install the ONIE binary on a particular platform.

Source Code Description
=======================

Source Code Layout
------------------

The ONIE source code layout is as follows::

  onie
  ├── build
  │   └── docs
  │       ├── doctrees
  │       └── html
  ├── build-config
  │   ├── arch
  │   ├── conf
  │   ├── make
  │   └── scripts
  ├── contrib
  │   └── onie-server
  ├── demo
  ├── docs
  ├── installer
  ├── machine
  │   └──<platform> 
  │       ├── demo
  │       ├── kernel
  │       ├── test
  │       └── u-boot
  ├── patches
  │   ├── busybox
  │   ├── crosstool-NG
  │   ├── e2fsprogs
  │   ├── kernel
  │   └── u-boot
  ├── rootconf
  │   └── default
  │       ├── bin
  │       ├── etc
  │       │   ├── init.d
  │       │   ├── rc3.d
  │       │   └── rcS.d
  │       ├── root
  │       ├── sbin
  │       └── scripts
  ├── test
  │   ├── bin
  │   ├── lib
  │   └── tests
  └── upstream

====================  =======
Directory             Purpose
====================  =======
build/docs            The final documentation is placed here.
build-config          Builds are launched from this directory.  The main Makefile is here.
build-config/arch     Contains configurations for CPU architectures.
build-config/conf     Contains configurations common to all platforms.
build-config/make     Contains makefile fragments included by the main Makefile.
build-config/scripts  Scripts used by the build process.
contrib/onie-server   A standalone DHCP+HTTP Python-based server for simple installs.
demo                  A sample ONIE-compliant installer and OS. See ``README.demo`` for details.
docs                  What you are reading now.
installer             Files for building an ONIE update installer.
machine               Contains platform-specific machine definition files. More details below.
patches               Patch sets applied to upstream projects, common to all platforms.
rootconf              Files copied into the final ``sysroot`` image. The main ONIE discovery
                      and execution application lives here.  More details below.
test/bin              Contains the ONIE testing harness (Python unit test-based).
test/lib              Common Python classes for writing ONIE tests.
test/tests            ONIE tests.
upstream              Local cache of upstream project tarballs.
====================  =======


Machine Definition Directory
----------------------------

The ``machine`` directory layout is as follows::

  onie/machine
  └── <vendor>
      └── <vendor>_<model>
          ├── demo
          │   └── platform.conf
          ├── INSTALL
          ├── kernel
          │   ├── config
          │   ├── platform-<platform>.patch
          │   └── series
          ├── machine.make
          ├── onie-rom.conf
          └── u-boot
              ├── platform-<platform>.patch
              └── series

The machine/nxp/nxp_p2020rbdpca directory contains all the files
necessary to build ONIE for the NXP P2020RBD-PCA reference platform.

================================   =======
File                               Purpose
================================   =======
demo/platform.conf                 Platform-specific codes for creating the demo OS.
INSTALL                            Platform-specific ONIE installation instructions.
kernel/config                      Additional kernel config appended to the core kernel config.
kernel/platform-<platform>.patch   Kernel platform-specific patch(es).
kernel/series                      List of kernel platform-specific patch(es) in order.
machine.make                       Platform-specific makefile.
onie-<platform>-rom.conf           Layout of the ONIE binary image(s).
u-boot/platform-<platform>.patch   U-Boot platform-specific patch(es).
u-boot/series                      List of U-Boot platform-specific patch(es) in order.
================================   =======


``rootconf`` Directory
----------------------

The ``rootconf`` directory layout is as follows::

  onie/rootconf
  ├── default
  │   ├── bin
  │   │   ├── discover
  │   │   ├── exec_installer
  │   │   ├── onie-nos-install
  │   │   ├── onie-console
  │   │   ├── support
  │   │   ├── uninstaller
  │   │   ├── onie-self-update
  │   │   └── onie-stop
  │   ├── etc
  │   │   ├── init.d
  │   │   │   ├── discover.sh
  │   │   │   ├── dropbear.sh
  │   │   │   ├── makedev.sh
  │   │   │   ├── networking.sh
  │   │   │   ├── rc
  │   │   │   ├── rc.local
  │   │   │   ├── syslogd.sh
  │   │   │   └── telnetd.sh
  │   │   ├── inittab
  │   │   ├── issue
  │   │   ├── issue.null
  │   │   ├── mtab
  │   │   ├── passwd
  │   │   ├── profile
  │   │   ├── rc3.d
  │   │   │   ├── S10dropbear.sh -> ../init.d/dropbear.sh
  │   │   │   ├── S10telnetd.sh -> ../init.d/telnetd.sh
  │   │   │   └── S50discover.sh -> ../init.d/discover.sh
  │   │   ├── rcS.d
  │   │   │   ├── S01makedev.sh -> ../init.d/makedev.sh
  │   │   │   ├── S05rc.local -> ../init.d/rc.local
  │   │   │   ├── S10networking.sh -> ../init.d/networking.sh
  │   │   │   └── S20syslogd.sh -> ../init.d/syslogd.sh
  │   │   └── syslog.conf
  │   ├── root
  │   ├── sbin
  │   │   └── boot-failure
  │   └── scripts
  │       ├── functions
  │       ├── udhcp4_net
  │       └── udhcp4_sd
  └── install

The contents of the ``default`` directory are copied to the ``sysroot``
verbatim during the build process.

==========================  =======
File                        Purpose
==========================  =======
bin/discover                Image discovery script. Feeds into ``exec_installer``.
bin/exec_installer          Downloads and executes an installer image.
bin/onie-nos-install        CLI for explicitly specifying an NOS URL to use for the install.
bin/support                 CLI that generates a tarball of useful system information.
bin/uninstaller             Executed during uninstall operations.
bin/onie-self-update        CLI for explicit specifying an ONIE update URL to use for the install.
bin/onie-stop               CLI for disabling discovery mode.  Terminates the discovery process.
etc/init.d                  Various initialization scripts.
etc/inittab                 Standard Linux initialization script.
etc/issue                   Standard Linux logon customization file.
etc/mtab                    Standard Linux file listing mounted file systems.
etc/passwd                  Standard Linux database file listing users authorized to access the system.
etc/profile                 Standard Linux file listing users of the system.
etc/rcS.d/S01makedev.sh     Creates the usual Linux kernel devices and file systems.
etc/rcS.d/S05rc.local       Standard Linux script to start ``rc.local``.
etc/rcS.d/S10networking.sh  Brings up the Ethernet management interface.
etc/rcS.d/S20syslogd.sh     Starts the ``syslogd`` service.
etc/rc3.c/S10dropbear.sh    Starts the ``dropbear`` SSH service.
etc/rc3.d/S10telnetd.sh     Starts the ``telnet`` service.
etc/rc3.d/S50discover.sh    Starts the ONIE discovery service.
install                     The installer file.                     
scripts                     General helper scripts, sourced by other scripts.
==========================  =======
