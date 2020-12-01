## Steps to prepare a RootFS 
---
### Dependencies / Prerequisites

1. Base image for rootfs used in this example is [Ubuntu 20.04](http://cdimage.ubuntu.com/ubuntu-base/releases/20.04/release/ubuntu-base-20.04.1-base-arm64.tar.gz). Download the archive and save it.

2. A linux host is needed to prepare the RootFS, Ubuntu 18.04 is used in this example. On the host, install the following package which makes chroot into arm64 system possible under an x86_64 host.

        $ sudo apt-get install qemu qemu-user-static binfmt-support debootstrap

3. Tools / utilities needed:
    * fallocate
    * mkfs.ext4
    * tar
    * Package manager like apt / yum depending on host OS
    * gzip

### Preparing RootFS on the host

1. Create a 1GB (or more as needed) file, install ext4 fs and mount it, 

        $ fallocate -l 1000M rootfs.img
        $ sudo mkfs.ext4 -F -L ROOTFS rootfs.img
        $ mkdir mnt
        $ sudo mount rootfs.img mnt

2. Use tar to decompress and extract the Ubuntu 20.04 base image, if the same is done on Windows, then all softlinks will be lost and chrooting will fail.

        $ sudo tar -xzvf ubuntu-base-20.04.1-base-arm64.tar.gz -C mnt/

3. Copy qemu-aarch64-static as below

        $ sudo cp -a /usr/bin/qemu-aarch64-static mnt/usr/bin/

4. chroot to new filesystem

        $ sudo chroot mnt/

5. Now initialization can be done as follows

        # Change the setting here
        $ USER=genzMan
        $ HOST=udk

        # Create User (If failure because of /dev/null not being present, then create /dev/null as follows - mknod -m 0666 /dev/null c 1 3)

        $ useradd -G sudo -m -s /bin/bash $USER
        $ passwd $USER
        # enter user password

        # Hostname & Network
        $ echo $HOST > /etc/hostname
        $ echo "127.0.0.1    localhost.localdomain localhost" > /etc/hosts
        $ echo "127.0.0.1    $HOST" >> /etc/hosts
        $ echo "auto eth0" > /etc/network/interfaces.d/eth0
        $ echo "iface eth0 inet dhcp" >> /etc/network/interfaces.d/eth0
        $ echo "nameserver 127.0.1.1" > /etc/resolv.conf

        # Install packages (if installing python fails because it cannot get random numbers, then create /dev/random and /dev/urandom as follows - mknod -m 0666 /dev/random c 1 8; mknod -m 0666 /dev/urandom c 1 9)

        $ apt-get update
        $ apt-get upgrade
        $ apt-get install ifupdown net-tools network-manager iputils-ping ethtool 
        $ apt-get install udev sudo ssh
        $ apt-get install vim nano

6. Exit out of chroot by pressing Ctrl + D, then copy your kernel's `/lib/modules/kernel_version/*` files to `mnt/lib/`\
   The kernel shipping by default in the uDK is 4.19.0, its `/lib/modules/4.19.0/` files are in the linux-genz [github](https://github.com/linux-genz/udk/tree/master/orthus).

7. If just rootfs.img needed, then umount and rootfs.img is ready

        $ sudo umount mnt/

8. If compressed rootfs.tar.gz needed, then do

        $ sudo tar -cv mnt/* -f rootfs.tar
        $ gzip -k rootfs.tar

### Notes

1. Additional packages to install as needed

        * $ apt install ubuntu-minimal whiptail perl systemd systemd-sysv openssh-client openssh-server openssh-sftp-server iproute2 less rsyslog bind9-host zip unzip acl bash-completion bc build-essential
	
        * $ apt install ubuntu-standard perl sudo locales fake-hwclock ntp ntpstat haveged tree autoconf automake libtool flex bison gdb git cloc device-tree-compiler apt-utils at btrfs-progs cmake cscope curl dh-python dmeventd dns-root-data emacs-gtk facter libruby finalrd finger fio fping gawk hiera htop hwloc initramfs-tools iperf kpartx landscape-common libdaxctl-dev libpmem-dev libpython3-dev libsysfs2 lvm2 mdadm netcat-openbsd ocfs2-tools python3-flask python3-pip resolvconf sysfsutils tcsh trace-cmd uuid-dev uuid xfsprogs libncurses-dev libssl-dev
