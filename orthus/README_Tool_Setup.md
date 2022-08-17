# Recommended Tools

## General Linux Utilities
All of these can be installed via a package manager like apt.

    dosfstools - This is needed for creating the vFAT filesystem.
    nfs-kernel-server - this is needed on a host for setting up a NFS Filesystem and could be useful for transferring files to Linux on the Orthus card.
    nfs-common - This is for the Orthus card for connecting to an NFS server.

## Xilinx Tools
Both tools can be installed via the unified installer which can be obtained from Xilinx. [Xilinx_Downloads].

    hw_server - This is needed for downloading the image.ub
                and for running scripts that Bittware provides.
                See below for more details.
    vitis - This is needed because it includes XSCT,
            which is required for downloading the image.ub
            and for running scripts that Bittware provides.
            See below for more details.

## Bittware scripts
Refer to Bittware's site [Orthus_Hardware_Guide] for more information, but below are some packages that might need to be installed in order for the scripts to work. While none of the guides will cover the usage of the scripts and they are not required, they may be useful.

    xvfb
    libtinfo5
    dbus-x11


[README]: https://github.com/linux-genz/udk/blob/master/orthus/README.md
[README_Tool_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/README_Tool_Setup.md
[README_Orthus_Hardware]: https://github.com/linux-genz/udk/blob/master/orthus/README_Orthus_Hardware.md
[README_Troubleshooting]: https://github.com/linux-genz/udk/blob/master/orthus/README_Troubleshooting.md
[HOWTO_Nfs_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Nfs_Setup.md
[HOWTO_Nfs_Load]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Nfs_Load.md
[HOWTO_Filesystem_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Filesystem_Setup.md
[HOWTO_Partition_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Partition_Setup.md
[Orthus_Github]: https://github.com/linux-genz/udk/blob/master/orthus/
[Orthus_Hardware_Guide]: https://developer.bittware.com/products/250-soc.php
[Xilinx_Downloads]: https://www.xilinx.com/support/download.html
[Zynq MPSoC Overview]: https://docs.xilinx.com/v/u/en-US/ds891-zynq-ultrascale-plus-overview
