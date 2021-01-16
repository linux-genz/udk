## Preparing the kernel image to be flashed to eMMC on Orthus card.
---
### Dependencies
1. Tools / utilities
    * fallocate
    * mkfs.fat
    *
2. Kernel source can be obtained from linux-genz [github](https://github.com/linux-genz/linux).

---

### Configure kernel options
1.  Remove old compilation files and sanitize your environment
        $ make clean
2.  Create default defconfig. xilinx_zynqmp_defconfig is default configuration maintained by Xilinx for their MPSoC FPGA
        $ make CROSS_COMPILE=aarch64-linux-gnu-  ARCH=arm64  xilinx_zynqmp_defconfig
3.  Enable following KConfig as modules:
	- GENZ=M
	- GENZ_BLK=M
	- GENZ_ORTHUS=M 
	- BINFMT_MISC=M
	- Do not enable GENZ_WILDCAT=N

4.  Enable following KConfig with compiled in options
	- XFS_FS=y
	- DYNAMIC_DEBUG=y // (optional, for debugging purposes)
	- BPF_SYSCALL=y
	- CGROUP_BPF=y
	- NET_NS=y
	- SECCOMP=y
	- CHECKPOINT_RESTORE=y
	- CGROUP_SCHED=y
	- CFS_BANDWIDTH=y
	- SCHEDSTATS=y
	- SCHED_DEBUG=y

5. enable POSIX_ACL=y for:
	- TMPFS
	- EXT4
	- XFS
	- BTRFS

6. Optionally you may disable following KConfigs to reduce compilation time and reduce image size

	- WLAN=n
	- WIRELESS=n
	- USB=n
	- USB_PCI=n
	- BT=n


7. Set UEVENT_HELPER_PATH to ""

---

### Device Tree modifications

1.  Device tree addition for ethernet
*  For Petalinux, make following changes in file project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
```
  &gem0 {
      status = "disabled";
  };
  &gem1 {
      phy-mode = "sgmii";
      phy-handle = <&phy0>;
      status = "okay";
      is-internal-pcspma;
      xlnx,ptp-enet-clock = <0x0>;
      phy0: phy@0 {
       compatible = "ethernet-phy-ieee802.3-c22";
       device_type = "ethernet-phy";
       ti,rx-internal-delay = <0x8>;
       ti,tx-internal-delay = <0xa>;
       ti,fifo-depth = <0x1>;
       reg = <0>;
      };
  };
```

2.  Additions for PTE Table and Raw CB
*  For Petalinux, make following changes in file project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
```
  &GenZReqrSS_pte_table_cntrl {
    compatible = "xlnx,iprop-genz-pte-table";
  };
  &GenZReqrSS_raw_cb_ctrl {
    compatible = "xlnx,iprop-genz-raw-cb-table";
  };
```


#### Compile Kernel
```
        $ make CROSS_COMPILE=aarch64-linux-gnu-  ARCH=arm64
        $ make CROSS_COMPILE=aarch64-linux-gnu-  ARCH=arm64 modules
```

---

## Preparing Kernel image for eMMC
1. Have the compiled kernel image 'image.ub' ready.\
   Kernel 4.19.0 used by default in the uDK is available in the linux-genz [github](https://github.com/linux-genz/udk/blob/master/orthus/v0p8/image.ub).

2. On a linux host, do the following

    a. Partition 1 on eMMC device is used for storing kernel, it is of size 64MiB 

        $ fallocate -l 64M kernel.img
        $ sudo mkfs.fat -F 32 kernel.img
        $ mkdir mnt
        $ sudo mount kernel.img mnt

    b. Now copy the image.ub kernel 

        $ sudo cp image.ub mnt/

    c. Unmount now and kernel.img is ready

        $ sudo umount mnt/
        
