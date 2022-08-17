# NTFS File System Load flow

This document contains information on how to boot from Linux from a network based file system. This guide assumes setup was already completed ([HOWTO_Nfs_Setup]). This can be useful for resizing or creating partitions, writing a new rootfs, and changing BOOT.BIN, boot.scr, and image.ub files ([Orthus_Github]) in the vfat partition.


## Boot Sequence

The following section will describe how to boot to the NFS file system in 2 different scenarios.
For both paths, make sure the UART terminal is set up ([README_Orthus_Hardware]).

### With working image.ub and working boot process

Interrupt the U-Boot sequence by pressing any key.

Type the following line into the UART interface. Replace nfs_server_ip with the IP of the NFS server. If a static IP is required for the card while booting, replace the dhcp portion of ip=dhcp with the desired static IP.

    $ setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=nfs_server_ip:/mnt/nfs_share/shared_fs,tcp,vers=4 ip=dhcp rw rootwait"; boot

The bootargs portion set bootargs for the image.ub.
The boot portion proceeds with the boot process which loads the image.ub in memory

### With working image.ub but some failure in the boot process

Interrupt the U-Boot sequence by pressing any key.

Type the following line into the UART interface. Replace nfs_server_ip with the IP of the NFS server. If a static IP is required for the card while booting, replace the dhcp portion of ip=dhcp with the desired static IP.

    $ fatload mmc 0 0x20000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=nfs_server_ip:/mnt/nfs_share/shared_fs,tcp,vers=4 ip=dhcp rw rootwait"; bootm 0x2000000

The bootargs portion set bootargs for the image.ub.
The boot portion proceeds with the boot process which loads the image.ub in memory

### Without working image.ub

Interrupt the U-Boot sequence by pressing any key.

Now a working image.ub needs to be provided via JTAG. The general flow will be described here and assumes a working hw_server is already set up and assumes a working XSCT terminal. Refer to tools setup ([README_Tool_Setup]) and Xilinx Documentation ([Zynq MPSoC Overview]) for more information.

From a working XSCT you will want to run the following commands.
Replace hardware_server_ip with the IP of the host the hw_server is hosted on.

    $ connect -host hardware_server_ip
    $ targets

You should see output similar to below:

     1  PS TAP
     2  PMU
     3  PL
        4 Legacy Debug Hub
     5  PSU
        6  RPU
           7  Cortex-R5 #0 (Halted)
           8  Cortex-R5 #1 (Lock Step Mode)
        9  APU
          10  Cortex-A53 #0 (Running)
          11  Cortex-A53 #1 (Power On Reset)
          12  Cortex-A53 #2 (Power On Reset)
          13  Cortex-A53 #3 (Power On Reset)

Make note of the number for Cortex-A53 #0 under APU. These numbers may not always be the same. This example will use 10, but replace with the correct value.

    $ target 10
    $ dow -data image.ub 0x20000000

Type the following line into the UART interface. Replace nfs_server_ip with the IP of the NFS server. If a static IP is required for the card while booting, replace the dhcp portion of ip=dhcp with the desired static IP.

    $ setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=nfs_server_ip:/mnt/nfs_share/shared_fs,tcp,vers=4 ip=dhcp rw rootwait"; bootm 0x20000000

The bootargs portion set bootargs for the image.ub.
The bootm portion boots from 0x2000000



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
