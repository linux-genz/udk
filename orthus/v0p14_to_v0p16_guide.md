### 0p14 to 0p16 Migration Guide

Get new orthus images from this repo in orthus/orthus_images/v0p16-images.tgz

After unpacking on your 250-soc, make sure your /boot partition is mounted, and copy BOOT.BIN and image.ub to /boot.
Then "cd /lib/modules" and untar modules-v5.10-genz-0p16.tgz

---
#### genz.conf

The file at /etc/modprobe.d/genz.conf needs to be modified with additional settings for genz, genz-blk, and orthus.
The new param is listed below.

    req_page_grid="256M*1784,2M*1022,4K*1024"

#### Zephyr

The updated zephyr (v0.4) requires a compatible version of llamas (v0.5).

#### Llamas

Don't forget to reinstall llamas after updating, by running the setup.py: "sudo python3 setup.py install"

After setup.py runs successfully, you will need to setup running llamas as a service. Start by copying llamas.service from the top of the llamas repo to /etc/systemd/system/llamas.service

Now create a new shell script at the path /usr/local/bin/llamas and make sure it is owned by root:root with permission 755. The script should contain the following, but make any necessary changes to make sure the paths point to where the llamas and genz-utils repos are checked out.

    #!/bin/bash

    cd /home/udk/src/linux/llamas
    PYTHONPATH=$PYTHONPATH:/home/udk/src/linux/genz-utils exec ./llamas/llamas_api.py $*

Now you can run the following to start the llamas service and ensure it runs on every boot.

    sudo systemctl start llamas
    sudo systemctl enable llamas
