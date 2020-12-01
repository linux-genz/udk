## Preparing the kernel image to be flashed to eMMC on Orthus card.
---
### Dependencies
1. Tools / utilities
    * fallocate
    * mkfs.fat
    *

### Kernel Compilation Steps
1. Kernel source can be obtained from linux-genz [github](https://github.com/linux-genz/linux).
2. Compilation commands:
        $ make clean

        $ make CROSS_COMPILE=aarch64-linux-gnu-  ARCH=arm64  xilinx_zynqmp_defconfig

        $ make CROSS_COMPILE=aarch64-linux-gnu-  ARCH=arm64

### Creating kernel.img for flashing to eMMC
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
        
