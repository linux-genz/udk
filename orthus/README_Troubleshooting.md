# Troubleshooting

## JTAG UART setup

### hw_server jtag poll delay

If there are any JTAG UART issues or if there are any unexplained hangs while a device that uses JTAG UART is connected to a hw_server, it may be useful to open the hw_server with as is shown below:

    $ sudo hw_server -e "set jtag_poll_delay 1000"

## Networking Setup

### Specifying a DNS Server
Using your favorite editor, such as vim, open /etc/systemd/resolved.conf with sudo. Once open, find the #DNS= line and uncomment it. After uncommenting it, write the desired IP after the = then save the changes. Once the edit has been made you will need to apply the change. To do so, run the following:

    $ sudo systemctl restart systemd-resolved

### Missing ethernet interface
Check to see if /etc/netplan/01-network-manager-all.yaml exists. If it does not, create it then fill it with the following:

    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp4: yes

Make sure the indents are in increments of 2 spaces. Afterwards you may need to run the following for changes to apply:

    $ sudo systemctl restart systemd-resolved

[README]: https://github.com/linux-genz/udk/blob/master/orthus/README.md
[README_Tool_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/README_Tool_Setup.md
[README_Orthus_Hardware]: https://github.com/linux-genz/udk/blob/master/orthus/README_Orthus_Hardware.md
[README_Troubleshooting]: https://github.com/linux-genz/udk/blob/master/orthus/README_Troubleshooting.md
[HOWTO_Nfs_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Nfs_Setup.md
[HOWTO_Nfs_Load]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Nfs_Load.md
[HOWTO_Filesystem_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Filesystem_Setup.md
[HOWTO_Partition_Setup]: https://github.com/linux-genz/udk/blob/master/orthus/HOWTO_Partition_Setup.md
[Orthus_Github]: https://github.com/linux-genz/udk/blob/master/orthus/
[Orthus_Hardware_Guide]: https://developer.bittware.com/products/250-soc.php
[Xilinx_Downloads]: https://www.xilinx.com/support/download.html
[Zynq MPSoC Overview]: https://docs.xilinx.com/v/u/en-US/ds891-zynq-ultrascale-plus-overview
