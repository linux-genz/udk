# NTFS File System setup flow

This document contains information on how to set up an NFS for booting to a Linux filesystem.

## Host setup

### NFS Server

First install NFS server and create the directory. In this case, /mnt/nfs_share/ is used.

    $ sudo apt install nfs-kernel-server
    $ sudo mkdir -p /mnt/nfs_share
    $ sudo chown -R nobody:nogroup /mnt/nfs_share/
    $ sudo chmod 777 /mnt/nfs_share/

Edit the exports file with the directory of choice.

    $ sudo echo "/mnt/nfs_share  192.168.0.0/24(rw,sync,no_root_squash,no_subtree_check,crossmnt)" > /etc/exports

Start the export filesystem(any changes to /etc/exports requires this step)

    $ sudo exportfs -a
    $ sudo systemctl restart nfs-kernel-server

### Filesystem setup

The zipped filesystem image provided on the [Orthus_Github] should be placed in the NFS share. Should be rootfs_ubuntu_22.04.img, but if not replace it in the line below with the correct name.
Then run the following in the location it was placed:

    $ sudo mkdir shared_fs
    $ sudo mount -o loop,rw,sync rootfs_ubuntu_22.04.img shared_fs/



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
