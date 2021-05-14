## HOWTO_flash_kernel_from_uboot

Following instructions provide various methods on how to program Linux kernel and Root file-system through u-boot.

---

### Setting up TFTP server

#### TFTP Server on Host machine   
1. Setup TFTP server to any external host which is accessible through wired network   
	* Refer [How to setup TFTP on Ubuntu ?](https://linuxhint.com/install_tftp_server_ubuntu/)   
	* Use TFTP_ADDRESS="IP_ADDRESS_OF_THIS_SERVER:PORT_NUMBER"   
	* Use TFTP_OPTIONS="--secure --create"   

2. Once tftp server is running on host, copy kernel and root file-system binaries to the tftp root directory   
	**Remember to change the ownership of files like kernel.img rootfs.img using following command**
	LINUX> chown tftp:tftp  image.ub rootfs.img   

#### TFTP Client on Orthus Card    
1. Serial console configurations for Orthus card  

        Serial device= /dev/ttyUSB0   
        Baud = 115200 8N1   
        Harware flow control = No   
        Software flow control = No   


2. Connect Orthus card to same LAN network as TFTP Server above, using RJ45 connector mounted on Ethernet daughter card along full-height bracket   

3. Using serial console, configure following variables on Orthus card  

        ZynqMP> setenv serverip 10.0.31.150  // This is $TFTP_ADDRESS as configured on TFTP host server   
        ZynqMP> setenv gateway 10.0.31.1     // This is IP address of your local lab or home network. If there is no gateway then you can set it same as $TFTP_ADDRESS   
        ZynqMP> setenv ipaddr 10.0.31.147    // This is static IP address of your kit, ensure that this does not conflict with any other machine on the subnet. You can also enable DHCP in u-boot to get dynamic IP.  

   *  U-Boot also supports DHCP, please look into the dhcp tool on U-Boot for more details.   

4. Verify if Orthus card can ping TFTP Server   

        ZynqMP> ping $serverip
        Using ethernet@ff0c0000 device
        host 10.0.31.150 is alive

5. Save these settings for future reuse   
        ZynqMP> saveenv


### Program Linux kernel to eMMC on Orthus card   

1. Fetch kernel.img from TFTP server to kit memory at address 0x1000000 note down the addresses and # of bytes received printed at end of command   

        ZynqMP> mw.b 0x1000000 0x0 0x4000000           <--- clear the memory segment where kernel.img will be copied

	ZynqMP> tftpboot 0x1000000 kernel.img
        Using ethernet@ff0c0000 device
        TFTP from server 10.0.31.150; our IP address is 10.0.31.147
        Filename 'kernel.img'.
        Load address: 0x1000000                        <--- ** Location in memory where kernel.imge is copied (Note this)**
        Loading: *#################################################################
                 #################################################################
                 #################################################################
                 #################################################################
                 ...
                 ######################
                 3.1 MiB/s
        done
        Bytes transferred = 67108864 (4000000 hex)    <--- ** Total size of kernel image in bytes (NOTE this)**


2. Identify partitions on the eMMC   

        ZynqMP> mmc info
        Device: mmc@ff160000
        Manufacturer ID: 45
        OEM: 100
        Name: DG403 
        Bus Speed: 52000000
        Mode : MMC High Speed (52MHz)
        Rd Block Len: 512
        MMC version 5.1
        High Capacity: Yes
        Capacity: 29.1 GiB
        Bus Width: 4-bit
        Erase Group Size: 512 KiB
        HC WP Group Size: 8 MiB
        User Capacity: 29.1 GiB WRREL
        Boot Capacity: 4 MiB ENH
        RPMB Capacity: 4 MiB ENH   

        ZynqMP> mmc dev 0
        switch to partitions #0, OK
        mmc0(part 0) is current device  <--- This is your eMMC device (MMC 0)   

        ZynqMP> mmc part   
        Partition Map for MMC device 0  --   Partition Type: DOS

        Part	Start Sector	Num Sectors	UUID		Type
          11	2048      	131072    	9684eaa1-01	83    <--- This partition is used for Linux kernel
          2	133120    	60938240  	9684eaa1-02	83    <--- And next partition for Root File-system 


3. Erase partition 1 and flash kernel from memory address 0x1000000 to MMC   

        ZynqMP> mmc dev 0  
        switch to partitions #0, OK
        mmc0(part 0) is current device

        ZynqMP> mmc erase 0x800 0x20000     
        MMC erase: dev # 0, block # 2048, count 131072 ... 131072 blocks erased: OK

        ZynqMP> mmc write 0x1000000 0x800 0x20000
        MMC write: dev # 0, block # 2048, count 131072 ... 131072 blocks written: OK

    *  0x800 = 2048 (decimal) is first sector of this Partition
    *  0x20000 = 131072 (decimal) which is Total size of kernel in bytes / sector size = (67108864 / 512) = 131072 (decimal)  
    *  0x1000000 = memory address where kernel.img was copied using TFTP  



---
---

### Program Root File-System image to eMMC on Orthus Card   


1. Fetch Root File-system image from TFTP   

        ZynqMP> mw.b 0x1000000 0x0 0x20000000          <--- clear the memory segment where rootfs.img will be copied
        ZynqMP> tftpboot 0x2000000 rootfs.img
        Using ethernet@ff0c0000 device
        TFTP from server 10.0.31.150; our IP address is 10.0.31.147
        Filename 'rootfs.img'.
        Load address: 0x2000000                      <--- ** Location in memory where rootfs.img is copied (Note this)**
        Loading: #################################################################
                 #################################################################
                 #################################################################
                 #################################################################
                 ...
                 ######################
                 5.3 MiB/s
        done
        Bytes transferred = 524288000 (1f400000 hex) <--- ** Total size of Root file-system image in Bytes (Note this) **   


2. Erase partition 2 and flash rootfs from memory address 0x2000000 to MMC,

        ZynqMP> mmc dev 0
        switch to partitions #0, OK
        mmc0(part 0) is current device

        ZynqMP> mmc erase 0x20800 0x400000
        MMC erase: dev # 0, block # 133120, count 4194304 ... 4194304 blocks erased: OK

        ZynqMP> mmc write 0x2000000 0x20800 0xfa000
        MMC write: dev # 0, block # 133120, count 1024000 ... 1024000 blocks written: OK

    *  Partition 2 is ~29G in size, so to save time you may erase only the space which is required. For example we erase only 2GB in above command      
    *  0x20800 = 133130 (decimal) is first sector of this Partition
    *  0x400000 = (size of partition in bytes) / (sector size) = (2 x 1024 x 1024 x 1024 / 512) = 4194304 (decimal) = 0x400000
    *  0x2000000 = memory address where root.img was copied using TFTP    
    *  0xfa000 = (size of the RootFS image) / (sector size) = (1024000 / 512) = 1024000 (decimal) = 0xfa000


3. Update bootargs variable to select eMMC as Root device.   

        ZynqMP> setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait"
        ZynqMP> saveenv

    *  By default bootargs variable should use root=/dev/mmcblk0p2 which is eMMC partition-2 as Root file-system    


4. Power cycle and kernel, rootfs should boot from eMMC now.  
        ZynqMP> bootm 0x1000000
    * Note: do NOT change the Switch SW1 on Othus card. The setting of the switch determines where the FSBL (u-boot) is located which is QSPI.


