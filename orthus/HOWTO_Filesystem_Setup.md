# Filesystem setup

## vFAT filesystem

This partition is used for programming the FPGA and the initial boot process and contains 3 files. BOOT.BIN, boot.scr, and image.ub. These are available here: [Orthus_Github]. Either in a NFS Filesystem ([HOWTO_Nfs_Load]) or while booted into Linux on the card, you can just copy the files into the partition. You just need to make sure to mount the partition (assuming the vFAT partition is /dev/mmcblk0p1, but replace with your own if it is different). You can do this with:

    $ sudo mkdir partition1_mnt
    $ mount -t vfat /dev/mmcblk0p1 partition1_mnt

You can then copy the above files into partition1_mnt.

WARNING:: Ensure the copy completes before rebooting. You can ensure this by unmounting /dev/mmcblk0p1

## EXT4 filesystem

This partition stores the Linux filesystem from which the card will end up booting from. To write this, you will want to have booted from a NFS Filesystem ([HOWTO_Nfs_Load]). This portion of the guide will assume nothing was written to this partition yet. You will need the rootfs_ubuntu_22.04.img.gz, or the latest img, ([Orthus_Github]) to be accessible by the NFS Filesystem for this as well.

Just run the following in the NFS filesystem if unzipped:

    $ sudo dd if=/path/to/rootfs.img of=/dev/mmcblk0p2

If zipped as .gz you can run the following instead:

    $ zcat /path/to/rootfs.img.gz | sudo dd of=/dev/mmcblk0p2

Once the copy is completed the filesystem needs to be expanded, so run the following:

    $ sudo e2fsck -f /dev/mmcblk0p2
    $ sudo resize2fs /dev/mmcblk0p2

The e2fsck is required by resize2fs before it can be ran and resize2fs ensures the filesystem uses the whole partition.


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
