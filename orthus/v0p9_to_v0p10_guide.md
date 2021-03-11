### v0p9 to v0p10 Migration Guide

While most of the development flow is the same between v0p9 and v0p10, there are a few changes that should be noted. 
The following steps are required for v0p10.

---

#### Manual Boot

Autoboot is currently not working with the current configuration in Petalinux 2020.2. We are currently trying to fix this issue, but in the meantime, please interrupt the autoboot process or let it fail and then input the following command:

    fatload mmc 0 0x10000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 ip=dhcp rw rootwait cpuidle.off=1"; bootm 0x10000000

These 3 commands instructs U-Boot to read the kernel file on the mmc (image.ub), set the bootargs for the console, the rootfs location, and other settings, and then boot the kernel.

If the eMMC on the 250-Soc has not been partitioned in such a way, you can partition it with the following commands:

    root@udk:~# fdisk /dev/mmcblk0
    t,1,c   (set the type of partition 1 to 32bit FAT)
    root@udk:~# apt install dosfstools
    root@udk:~# mkfs.vfat -F 32 -n boot /dev/mmcblk0p1
    root@udk:~# mount /dev/mmcblk0p1 /boot
    
    (reboot)
    
    ZynqMP> fatls mmc 0:1
    
      7299528   image.ub

#### genz.conf changes

The file at /etc/modprobe.d/genz.conf has changed from what is included in the rootfs v1.2. Please replace its content with the following:

    Change genz.conf:
    # Revisit: blacklist
    blacklist genz
    #blacklist genz-blk
    blacklist orthus
    options genz    dyndbg="+pflm" \
                    req_page_grid="1G*126,2M*1000,4K*2000"
    options genz-blk dyndbg="+pflm; func genz_blk_sgl_cmpl =_"
    options orthus  dyndbg="+pflm; func orthus_local_control_read =_; func orthus_control_offset_to_base =_; func orthus_sgl_request =_"

#### New module for 5.4.0 kernel

In addition, because we are now on the 5.4.0 kernel, the lib/modules/4.19.0/ directory on the rootfs needs to be replaced with lib/modules/5.4.0
The contents come from the rootfs created by Petalinux, and it is uploaded at orthus_images/v0p10-linux5p4-modules.tar.gz.
*Please note that the genz & orthus modules in this tar will need to be replaced by the modules provided in orthus image tar*

#### New Kernel Config Options 

If you are building your own kernel, make sure you have the following options set. There are some changes and some additions since v0p9

    Search for GENZ
    Choose (1), GENZ, enable as module <M>
    Choose (2), GENZ_BLK, enable as module <M>
    Choose (3), GENZ_ORTHUS, enable as module <M>
    Do not enable GENZ_WILDCAT [it should no longer appear as an option]
    Search for and enable XFS_FS as module <M>
    Search for DYNAMIC_DEBUG and enable  [makes kernel image bigger, but well worth it]
    Search for GDB_SCRIPTS and enable
    Search for KGDB and KGDB_KDB and enable both
    Search for BPF_SYSCALL and enable
    Search for CGROUP_BPF and enable  [these 2 BPF settings are needed by Ubuntu user-space]
    Search for BINFMT_MISC and enable as module <M> [systemd wants this]
    Search for UEVENT_HELPER_PATH and set to "" [for systemd in v4.x kernels]
    Search for UEVENT_HELPER and disable [for systemd in v5.x kernels]
    Search for and enable these [for systemd]: NET_NS, SECCOMP, CHECKPOINT_RESTORE, CGROUP_SCHED, CFS_BANDWIDTH, SCHEDSTATS, SCHED_DEBUG
    Search for POSIX_ACL and enable for BTRFS,EXT4,TMPFS,XFS [for systemd]
    Search for WLAN and WIRELESS and disable both [250-SOC has no wireless]
    Search for USB and USB_PCI and disable both [250-SOC has no USB]
    Search for BT and disable [250-SOC has no Bluetooth]
    Search for CPU_IDLE and disable [workaround for Vivado HW-mgr hangs CPUs]
    Search for MEMORY_HOTPLUG and MEMORY_HOTREMOVE and enable both
    Search for SPARSEMEM_VMEMMAP and enable
    Search for ZONE_DEVICE and enable
    Search for FS_DAX and enable


#### Other notes

The genz.ko and genz-blk.ko modules need to be updated as well to work with the new orthus.ko. The updated modules can be found in v0p10-images.tar.gz or can be built from Github source. Note that because of the change in kernel versions, the new genz-linux branch is genz-xlnx_rebase_v5.4.

