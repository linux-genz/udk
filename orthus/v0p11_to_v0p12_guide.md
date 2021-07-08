### 0p11 to 0p12 Migration Guide

There are updates to the genz.conf file and the genz-utils to run 0p12

---
#### genz.conf

The file at /etc/modprobe.d/genz.conf has changed with additional settings for genz-blk and orthus
    
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
    
#### Zephyr

The updated Zephyr requires a new argument in zephyr_fabric.conf
Please refer to the updated example_zephyr_fabric.conf
    
#### Llamas

Don't forget to reinstall llamas after updating, by running the setup.py: "python3 setup.py install"

After setup.py runs successfully, you can run llamas with: "sudo ./llamas_api.py -vvvvv". 

