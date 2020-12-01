## How to load Orthus kernel module
---
1. V0p8 version [orthus.ko](https://github.com/linux-genz/udk/blob/master/orthus/v0p8/orthus.ko) kernel driver is available by default in the uDK.\
   Orthus, genz, genz-blk modules are located under `/lib/modules/4.19.0/kernel/drivers/genz\`
   The orthus module can loaded as root user by the following command

        $ modprobe orthus

        $ lsmod
        Module                  Size  Used by
        orthus                225280  0
        genz                  147456  1 orthus
        uio_pdrv_genirq        16384  0


2. Default module parameters file `/etc/modprobe.d/genz.conf` present in uDK:
 
        # Revisit: blacklist
        blacklist genz
        blacklist genz-blk
        blacklist orthus
        options genz    dyndbg="+pflm" \
                req_page_grid="1G*222,2M*256,4K*456"
        options genz-blk dyndbg="+pflm"
        options orthus  dyndbg="+pflm"

3. Newer releases of orthus.ko, genz.ko and other kernel drivers will be uploaded to linux-genz [github](https://github.com/linux-genz/udk/blob/master/orthus/).\
   Modules can be fetched from github, copied over to uDK to replace existing modules, then can be loaded with modprobe / insmod commands.  
