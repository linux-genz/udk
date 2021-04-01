### eMMC Partition Resize Guide

The 250-SoC comes with a 64MB boot partition, and it becomes a limitation when we try to use emmc boot with the BOOT.BIN on the boot partition. In this guide we will shrink the ext4 partition that holds the rootfs and add a larger boot partition, without losing the data on the rootfs.

In this guide, I'll be shrinking the rootfs partition by 1GB and created a new 500MB boot paritition. The same steps can be used for bigger or smaller partitions, as long as there is enough space after the rootfs partition to fit the boot partition.

---

#### NFS Server

To shrink the rootfs partition, it must be unmounted. We will boot with an NFS rootfs so that we can resize the rootfs partition on the eMMC.
To set up an NFS server, you can do the following:

First install nfs server and create the directory. In my case, I used /mnt/nfs_share/

    $ sudo apt install nfs-kernel-server
    $ sudo mkdir -p /mnt/nfs_share
    $ sudo chown -R nobody:nogroup /mnt/nfs_share/
    $ sudo chmod 777 /mnt/nfs_share/
    
Edit the exports file with the directory of choice.

    $ sudo echo "/mnt/nfs_share  192.168.0.0/24(rw,sync,no_root_squash,no_subtree_check)" > /etc/exports

start the export filesystem

    $ sudo exportfs -a
    $ sudo systemctl restart nfs-kernel-server

We can now boot with an NFSRoot with a rootfs image of choice. I used the smartm rootfs v1.2 in my case. 192.168.0.188 is my nfs server and I had the rootfs stored at /mnt/nfs_share/smartm/

    fatload mmc 0 0x10000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=192.168.0.188:/mnt/nfs_share/smartm,tcp,vers=4 ip=dhcp rw rootwait"; bootm 0x10000000

#### RootFS Prereqs

The tools needed are parted, resize2fs, and mkfs. I needed to install parted and mkfs on the smartm rootfs. You may also need to set up your networks.

    genzMan@udk:~$ sudo apt-get install parted      
    genzMan@udk:~$ sudo apt-get install dosfstools


#### Shrinking the EXT4 FileSystem

    genzMan@udk:~$ lsblk
    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    mmcblk0      179:0    0 29.1G  0 disk 
    \u251c\u2500mmcblk0p1  179:1    0   64M  0 part 
    \u2514\u2500mmcblk0p2  179:2    0 29.1G  0 part 
    mmcblk0boot0 179:8    0    4M  1 disk 
    mmcblk0boot1 179:16   0    4M  1 disk 

The above shows the factory partition sizes on the 250-SoC. I will shrink mmcblk0p2, the ext4 rootfs partition, by 1GB. 

First run fsck

    genzMan@udk:~$ fsck /dev/mmcblk0p2  

Then shrink the filesystem to 28GB.

    genzMan@udk:~$ sudo resize2fs /dev/mmcblk0p2 28G
    resize2fs 1.45.5 (07-Jan-2020)
    Resizing the filesystem on /dev/mmcblk0p2 to 7340032 (4k) blocks.
    The filesystem on /dev/mmcblk0p2 is now 7340032 (4k) blocks long.

#### Resizing the EXT4 Partition

Now that the filesystem is shurnk, we can resize the partition. I will be shrinking it to 28.1GB, which is 30,240,287,246B

First run parted on /dev/mmcblk0

    genzMan@udk:~$ sudo parted /dev/mmcblk0
    GNU Parted 3.3
    Using /dev/mmcblk0
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) p                                                                
    Model: MMC DG4032 (sd/mmc)
    Disk /dev/mmcblk0: 31.3GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags: 
        
    Number  Start   End     Size    Type     File system  Flags
     1      1049kB  68.2MB  67.1MB  primary  fat32
     2      68.2MB  31.3GB  31.2GB  primary  ext4
        

Then change the units to bytes and print the partition information

    (parted) unit B                                                           
    (parted) p                                                                
    Model: MMC DG4032 (sd/mmc)
    Disk /dev/mmcblk0: 31268536320B
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags: 
        
    Number  Start      End           Size          Type     File system  Flags
     1      1048576B   68157439B     67108864B     primary  fat32
     2      68157440B  31268536319B  31200378880B  primary  ext4

Then resize the partition and print the information again to confirm the change

    (parted) resizepart 2 30240287246B
    (parted) p                                                                
    Model: MMC DG4032 (sd/mmc)
    Disk /dev/mmcblk0: 31268536320B
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags: 

    Number  Start      End           Size          Type     File system  Flags
    1      1048576B   68157439B     67108864B     primary  fat32
    2      68157440B  30240287743B  30172130304B  primary  ext4

lsblk will also confirm the change

        
    genzMan@udk:~$ lsblk                                                      
    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    mmcblk0      179:0    0 29.1G  0 disk 
    \u251c\u2500mmcblk0p1  179:1    0   64M  0 part 
    \u2514\u2500mmcblk0p2  179:2    0   28.1G  0 part 
    mmcblk0boot0 179:8    0    4M  1 disk 
    mmcblk0boot1 179:16   0    4M  1 disk 
    
Reboot the 250-SoC and use /dev/mmcblk0p2 as the rootfs to confirm it still works.

    fatload mmc 0 0x10000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 ip=dhcp rw rootwait cpuidle.off=1"; bootm 0x10000000

#### Creating a new boot partition

I will be creating a new 500MB boot partition. I will start the partition at the next sector, which is at 30,240,931,840B. 
The end will be at 30,240,931,840B + 524,288,000B = 30,765,219,840B

    genzMan@udk:~$ sudo parted /dev/mmcblk0
    GNU Parted 3.3
    Using /dev/mmcblk0
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) unit B
    (parted) p                                                                
    Model: MMC DG4032 (sd/mmc)
    Disk /dev/mmcblk0: 31268536320B
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags: 

    Number  Start      End           Size          Type     File system  Flags
     1      1048576B   68157439B     67108864B     primary  fat32
     2      68157440B  30240287743B  30172130304B  primary  ext4

First I remove the old partition
    
    (parted) rm 1               

Then I create a new primary fat32 partition, with the above start and end bytes.

    (parted) mkpart primary                                                   
    File system type?  [ext2]? fat32                                          
    Start? 30240931840B                                                       
    End? 30657216512B                                                         
    (parted) p                                                                
    Model: MMC DG4032 (sd/mmc)
    Disk /dev/mmcblk0: 31268536320B
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags: 

    Number  Start         End           Size          Type     File system  Flags
     2      68157440B     30240287743B  30172130304B  primary  ext4
     1      30240931840B  30657217023B  416285184B    primary  fat32        lba

Then I create a fat32 filesystem on the partition

    genzMan@udk:~$ sudo mkfs.vfat -F 32 /dev/mmcblk0p1
    mkfs.fat 4.1 (2017-01-24)

Now we can mount and put the necessary images on it

    genzMan@udk:~$ sudo mount /dev/mmcblk0p1 newboot/                                                                                                                                                                                             
    genzMan@udk:~$ sudo cp BOOT.BIN newboot/
    genzMan@udk:~$ sudo cp boot.scr newboot/
    genzMan@udk:~$ sudo cp image.ub newboot/

#### Testing the new partition

We can test a new partition before deleting the old one by loading a kernel image on it. The 250-SoC will only load the BOOT.BIN from partition 1, so we cannot try a boot on the new partition unless we delete the old partition. 

We can pick the 3rd partition to load the kernel from to test to confirm that the partition settings are correct.

    fatload mmc 0:3 0x10000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/nfs nfsroot=192.168.0.188:/mnt/nfs_share/smartm,tcp,vers=4 ip=dhcp rw rootwait"; bootm 0x10000000

We can also use this command with the shrunken rootfs partition to make sure that also works.

    fatload mmc 0:3 0x10000000 image.ub; setenv bootargs "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait"; bootm 0x10000000

If it boots, we know the settings are good, and we can delete both boot partitions and make a new one with the same settings.