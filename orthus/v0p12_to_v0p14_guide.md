### 0p12 to 0p14 Migration Guide

Get new orthus images from this repo in orthus/orthus_images/v0p14-images.tgz

After unpacking on your 250-soc, make sure your /boot partition is mounted, and copy BOOT.BIN and image.ub to /boot.
Then "cd /lib/modules" and untar modules-v5.10-genz-0p14.tgz

Optional (but recommended): Update ubuntu to 21.10 via the usual ubuntu tool (do_release_upgrade).
This updates userspace to match the new v5.10 kernel, for example, allowing per-inode control of DAX for files
on ext4 and xfs filesystems.

---
#### genz.conf

The file at /etc/modprobe.d/genz.conf needs to be modified with additional settings for genz, genz-blk, and orthus.
The blacklist lines that might be in your current file (for 0p12) are no longer required and should be removed.
    
    options genz    dyndbg="+pflm" \
                    req_page_grid="128M*1008,2M*1000,4K*2000"
    options genz-blk dyndbg="+pflm; func genz_blk_sgl_cmpl =_; func genz_blk_queue_rq =_" \
                     hw_queues=4 \
                     queue_depth=1 \
                     qfactor=1
    options orthus  dyndbg="+pflm; func orthus_local_control_read =_; func orthus_control_offset_to_base =_; \
                func iprop_genz_complex_intf_sts_queue_handler =_; func iprop_genz_complex_poll_status_threaded =_; \
                func orthus_rsp_raw_cb_isr =_; func iprop_raw_cb_pop_pkt =_; func iprop_genz_resp_pkt_decode =_; \
                func iprop_genz_resp_pkt_handler_control =_; func iprop_genz_set_pkt_gen_header =_; \
                func iprop_raw_cb_push_pkt =_; func iprop_genz_resp_pkt_do_ctrl_read =_; \
                func iprop_genz_resp_pkt_do_ctrl_write =_; func iprop_genz_build_ctrl_standalone_ack_pkt =_; \
                func iprop_raw_cb_push_pkt =_; func iprop_genz_resp_pkt_do_dr =_; \
                func orthus_local_control_write =_"
#### Zephyr

The updated zephyr (v0.3) requires a compatible version of llamas (v0.4).

#### Llamas

Don't forget to reinstall llamas after updating, by running the setup.py: "sudo python3 setup.py install"

After setup.py runs successfully, you can run llamas with: "sudo ./llamas_api.py -vvvvv". 

