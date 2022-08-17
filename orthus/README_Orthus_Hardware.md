# 250 SoC Hardware Information

This document contains general usage information for the Orthus 250 SoC card. which acts as a Gen-Z host. Orthus card uses Xilinx MPSoC FPGA ([Zynq MPSoC Overview]) to communicate with Gen-Z device over QSFP28 interface. Orthus card is powered by PCIe connector.

## UART Interface

Connect micro-USB cable to serial console located on bracket of 250-SoC

Make sure to use these settings when interacting with the UART interface.  
`Serial device= /dev/ttyUSB(N)`   // 'N' is your serial console terminal location  
`Baud = 115200 8N1`  
`Harware flow control = No`  
`Software flow control = No`

## Boot Config Switches

### Switch Location

The switches are on the back of the 250 SoC near the top.
Refer to the [README_Orthus_Hardware] for more information.

### Switch configurations

There are multiple configurations, but only the necessary ones will be listed here.

Note: For SW1.1 and SW1.4, they are shown for placement in a uDK or where the PCIe is not driving PERST and the PCIe Clock. If a host is driving them, you may need to turn these switches off.

The boot mode is driven primarily by SW1.2 and SW 1.3, so make sure to take note of those below.
Refer to the [README_Orthus_Hardware] for more information.

#### JTAG only  
-------------  

   | SW1   | FPGA Signal | Left (0=Off) | Right (1=On) |
   |-------|-------------|--------------|--------------|
   | SW1.1 | PCIE_CLK_SEL|              | On: PCIe Reference clock generate on-board |
   | SW1.2 | MOD1        |              | On           |
   | SW1.3 | MOD2        | Off          |              |
   | SW1.4 | PERST_DIR   |              | On: PERST is driven by FPGA |

#### JTAG & eMMC  
-------------  

   | SW1   | FPGA Signal | Left (0=Off) | Right (1=On) |
   |-------|-------------|--------------|--------------|
   | SW1.1 | PCIE_CLK_SEL|              | On: PCIe Reference clock generate on-board |
   | SW1.2 | MOD1        | Off          |              |
   | SW1.3 | MOD2        |              | On           |
   | SW1.4 | PERST_DIR   |              | On: PERST is driven by FPGA |



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
