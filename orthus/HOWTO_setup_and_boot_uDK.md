## How to load Orthus kernel module
---
**Setup**
*  Insert the ZMM in the fan cage   
*  Connect QSFP-to-1C cable from Orthus card (Bittware 250-SoCi) QSFP connector to backplane 1C connectors. Choose the 1C connector which are opposite to where ZMM is plugged-in   
*  Connect Micro-USB serial console to Orthus card for viewing console logs
*  Power On the kit.   
    
    

**Instructions**
1. Login to uDK (Orthus) using the credentials provided  (Username: root, Password: root)   

2. Update the module parameters file `/etc/modprobe.d/genz.conf`if not already updated. Following configurations are valid from version 0p12 onwards   
        
        # Revisit: blacklist   
        blacklist genz   
        #blacklist genz-blk   
        blacklist orthus   
        options genz    dyndbg="+pflm" \   
                        req_page_grid="1G*126,2M*1000,4K*2000"   
        options genz-blk dyndbg="+pflm; func genz_blk_sgl_cmpl =_" \   
                         hw_queues=4 \   
                         queue_depth=1 \   
                         qfactor=1  
        options orthus  dyndbg="+pflm; func orthus_local_control_read =_; func orthus_control_offset_to_base =_;" \   
       			cdma_thresh=512   
        
3. Insert the driver modules.   
     
        $ cd /lib/modules/`uname -r`/kernel/drivers/genz/   
        $ sudo modprobe orthus    
         
   You can individually load the modules, but need to follow the order
        
        $ sudo insmod genz.ko
        $ sudo insmod genz-blk.ko
        $ sudo insmod iprop/orthus.ko


4  Test the Gen-Z link
        
        $ sudo /home/udk/genz-utils/lsgenz
        

You can fetch the newer releases of driver modules (orthus.ko, genz.ko, genz-blk.ko) from linux-genz [github](https://github.com/linux-genz/udk/blob/master/orthus/).\
   Modules can be fetched from github, copied over to uDK to replace existing modules, then can be loaded with modprobe / insmod commands.  
