# SMESH

This repository provides a stand-alone, packaged library build of the
mesh framework tracking upstream sources the [Salome
Platform](www.salome-platform.org).

The repo, cmake setup and patches are based on Trevor Laughlin's work
in <a href="https://github.com/trelau/SMESH">SMESH packaging
repository</a>, though rearranged a bit to ease rpkg / mock builds for
Fedora.  Right now it is tracking from and hardwired to building the
bleeding edge of SMESH from Salome git against the bleeding edge of
OpenCascade from OCC git.  Stable maintenance build support against
tagged releases may appear at some point.

If you are looking to use SMESH functionality in a Python application,
please see trelau's [pySMESH](https://github.com/trelau/pySMESH).

## Build from source

This repository makes use of submodules. To build from sources you
must clone this repository and the submodules with:

    git clone --recurse-submodules https://github.com/montylab3d/smesh.git

OR

    git clone https://github.com/montylab3d/smesh.git
    git submodule init
    git submodule update

The submodules should automatically checkout on the 'packaging'
branch; this branch already includes the patches and changes needed.
rpkg is then able to build Fedora packages in a single step:

     rpg local --spec packages/fedore/smesh-bleed.spec

RPMS and SRPM are then built into rpkg's default buildroot (typically
under /tmp/rpkg/smesh-{version}...)
