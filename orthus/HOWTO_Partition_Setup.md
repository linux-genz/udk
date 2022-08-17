# Partition setup

This document will walk through a couple different partition setup examples that you may choose to use.

WARNING: If there is any existing data, it may be lost when changing partitions. If there is any data you want to keep, be sure to take steps to back it up before continuing.

## Partition map

The below map is an example of what the partitions should look like when you are done. If you make changes to this, make sure the requirements listed in the next section are met.

    Disk /dev/mmcblk0: 29.12 GiB, 31268536320 bytes, 61071360 sectors
    Disk Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x14d460e3

    Device         Boot   Start      End  Sectors  Size Id Type
    /dev/mmcblk0p1         2048  1026047  1024000  500M 83 Linux
    /dev/mmcblk0p2      1028096 61071359 60043264 28.6G 83 Linux

-------------  


## How to create/resize partition map

WARNING: There is a potential for lost data when doing anything with partitions and the listed example below will result in lost data. Be sure to back up anything that needs to be saved before proceeding with either the example method or your own method.

The fdisk guide and the Linux resize guide below assume the board is booted from a NFS filesystem. Check out [HOWTO_Nfs_Load] for information on how to do that if you have not done so already. The below methods are not the only method that can be used, so you can choose a preferred method, just pay attention to the below NOTEs.

Before working on the partitions, here are some important notes:

    NOTE 1: The EXT4 partition needs to start on a 2MiB aligned sector.
    NOTE 2: The vFAT partition needs to start at sector 2048.
    NOTE 3: The vFAT partition needs to be 500M in size.

### Partition create fdisk example

WARNING: This will destroy all data on /dev/mmcblk0!

To start using fdisk, you first want to see all available disks. To do so, run the following (that is a lower-case L):

    $ sudo fdisk -l

The disk we will use is /dev/mmcblk0. If the name for your disk ends up different, just ensure to target the eMMC device. To look at what partitions may be available on that disk, run the same command as above, but also targeting the disk.

    $ sudo fdisk -l /dev/mmcblk0

If any partitions require resizing, you will have to delete whatever partition is there and start over. To start the process run:

    $ sudo fdisk /dev/mmcblk0

To access the help menu you will want to press m. Each command will have prompts to help guide you so this guide will just describe the specific ones you might need.

    d - If resizing any partition this will be required.
        The prompt will have you select which partition to delete.
        If you need to resize the first partition, you will need to delete both.
    n - Add a new partition. The guided prompts will require a start and an end sector.
        You can use the map above or set your own, but be sure to keep the NOTEs in mind.
    w - Apply the changes to the target device and quit.
    q - Quite without saving changes.

The flow will be to use the (d)elete option to clear any partitions, then the (n)ew option to create both the vFAT and EXT4 partitions. The actual formatting will happen later. Once everything is as desired, press w to save the changes and exit fdisk. rerun the sudo fdisk -l command on this device to ensure the partitions are correct.

Now the partitions need to be formatted. The first partition should be a vFAT partition and the second should be EXT4. To do this, you need to make sure dosfstools is installed. Then you can run the following command to finish setting up the first partition:

    mkfs.vfat /dev/mmcblk0p1

Now the second partition needs to be formatted and that can be done with the following command:

    mkfs.ext4 /dev/mmcblk0p2

The partitions are now created and formatted, now you can refer to [HOWTO_Filesystem_Setup]


### Linux Partition resize parted example

Below is the lsblk output of the factory partition sizes while booted to an NFS filesystem. The following guide will be based off of this.

    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    mmcblk0      179:0    0 29.1G  0 disk
    \u251c\u2500mmcblk0p1  179:1    0   64M  0 part
    \u2514\u2500mmcblk0p2  179:2    0 29.1G  0 part
    mmcblk0boot0 179:8    0    4M  1 disk
    mmcblk0boot1 179:16   0    4M  1 disk

The first step is to run the following command

    fsck /dev/mmcblk0p2

After this, you will want to shrink the filesystem to 28GB with the following command

    sudo resize2fs /dev/mmcblk0p2 28G

With the filesystem shrunk, you can now shrink the partition itself. To do so, run parted with the following command.

    sudo parted /dev/mmcblk0

In parted you can resize the image. The commands you need are described below.

    Command    - Description
    p          - List the partitons, used to verify changes
    unit B     - Convert the units to bytes
    resizepart - Resizes the partition, used to do the actual resize

Now here are the full commands to use while within parted. You are first changing the unit to bytes. Then the first p indicates what the partitions look like. the resizepart command changes partition 2 to the given size. The final p verifies that the change applied.

    unit B
    p
    resizepart 2 30240287246B
    p

You can exit, then use lsblk afterwards to confirm. You can boot using the following.

    fatload mmc 0 0x20000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 ip=dhcp rw rootwait cpuidle.off=1"; bootm 0x20000000


### Boot Partition creation example

WARNING: Ensure to finish before rebooting, otherwise the card will fail to boot or program successfully.

This portion will assume you are booted into Linux on the card already and are wanting to expand to a 500MB.

Start parted with the following command.

    sudo parted /dev/mmcblk0

Below is the description of the parted commands that will be used. Be sure to refer to parted documentation for more information.

    Command  - Description
    p        - List the partitons, used to verify changes
    unit B   - Convert the units to bytes
    mkpart   - Makes a partition
    rm       - Delete a partition

Below is the commands that need to be ran. unit B changes the units to Bytes. p shows the list of current partitions. rm 1 removes partition 1. mkpart primary creates a primary partition, then it runs through a guided setup. The first asks the filesystem type, you will want to use fat32. It will then ask for the start and end. You can use the examples or your own, with the caveats being that they need to be outside of any existing partitions. The final p is used to verify the changes are as desired.

    unit B
    p
    rm 1
    mkpart primary
        fat32 - When prompted for filesystem type
        30240931840B - When prompted for start
        30657216512B - When prompted for end
    p

After this you can run the following to create the filesystem on the partition

    sudo mkfs.vfat -F 32 /dev/mmcblk0p1

Now you can mount it and copy the necessary files into it.

    sudo mount /dev/mmcblk0p1 newboot/
    sudo cp BOOT.BIN newboot/
    sudo cp boot.scr newboot/
    sudo cp image.ub newboot/

Now you can test this new partition while booting into an NFS filesystem or while booting into the shrunken filesystem from the previous section. Here are the 2 ways to do it.

NFS:

Replace nfs_server_ip with the IP of the NFS server. If a static IP is required for the card while booting, replace the dhcp portion of ip=dhcp with the desired static IP.

    fatload mmc 0:3 0x2000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=nfs_server_ip:/mnt/nfs_share/shared_fs,tcp,vers=4 ip=dhcp rw rootwait"; bootm 0x20000000

Shrunken Filesystem:

    fatload mmc 0:3 0x20000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 ip=dhcp rw rootwait cpuidle.off=1"; bootm 0x20000000





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
