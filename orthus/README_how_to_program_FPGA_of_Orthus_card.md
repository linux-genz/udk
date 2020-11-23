# Orthus Host Card

  
This document contains information about Orthus PCIe Add-in-card, which acts as Host in Gen-Z Micro Development Kit (uDK). Orthus card uses Xilinx MPSoC FPGA to communicate with Gen-Z device over QSFP28 interface. Orthus card is powered by PCIe connector.
Xilinx MPSoC FPGA on Orthus card is pre-installed with Gen-Z bitstream and pre-loaded with Linux kernel supportng Gen-Z sub-system. It comes with basic application utilities which help you discover Gen-Z devices on the fabric
All software source code is available through open-source repositories as listed below.
  
   
## References
  
### Software  
------------ 
- [linux-genz] repository to build Linux kernel from source  
- [Xilinx Zynq UltraScale+ MPSoC Software Developers Guide - UG1137](https://www.xilinx.com/support/documentation/user_guides/ug1137-zynq-ultrascale-mpsoc-swdev.pdf)   
- [Xilinx PetaLinux] Software Developmet Kit (SDK)
- [How to install Vivado/Vitis from command line ?](https://bbs.archlinux.org/viewtopic.php?pid=1918710#p1918710)  
  
### Hardware  
------------
- [Orthus-hardware-reference-user-guide]
- [Xilinx Zynq UltraScale+ MPSoC Overview - DS891](https://www.xilinx.com/support/documentation/data_sheets/ds891-zynq-ultrascale-plus-overview.pdf)  
- [Xilinx Zynq UltraScale+ MPSoC Datasheets](https://www.xilinx.com/support/documentation-navigation/silicon-devices/soc/zynq-ultrascale-plus-mpsoc.html)
     
---   
   
  
## How to program the orthus card ?  
  
### Dependencies  
----------------  
Current version of Orthus card image only works with [Vivado 2019.02](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2019-2.html). You can download this version from following links:   
-   [Vivado 2019.02 Lab edition -Linux](https://www.xilinx.com/member/forms/download/xef-vivado.html?filename=Xilinx_Vivado_Lab_Lin_2019.2_1106_2127.tar.gz)  
-   [Vivado 2019.02 Lab edition - Windows](https://www.xilinx.com/member/forms/download/xef-vivado.html?filename=Xilinx_Vivado_Lab_Win_2019.2_1106_2127.tar.gz)  
  
- [Xilinx Platform cable USB-II](https://www.xilinx.com/products/boards-and-kits/hw-usb-ii-g.html) for FPGA programming. Included in kit   
- First Stage boot loader **zynqmp_fsbl.elf**. Latest image can be downloaded from [linux-genz]/udk/orthus  
- Binary image **BOOT.BIN**. Latest image can be downloaded from [linux-genz]/udk/orthus  

     
---   
   

### Instructions  
-------------  
1. Update Switch SW1 on 250-SoC to Boot from JTAG   
   | SW1   | FPGA Signal | Left (0=Off) | Right (1=On) | 
   |-------|-------------|--------------|--------------|
   | SW1.1 | PCIE_CLK_SEL|              | On: PCIe Reference clock generate on-board |
   | SW1.2 | MOD1        |              | On           |
   | SW1.2 | MOD2        | Off          |              |
   | SW1.3 | PERST_DIR   |              | On: PERST is driven by FPGA |

- SW1 located on backside of 250-SoC card to boot from QSPI    
- Power cycle 250-SoC for above configurations to take effect    
 
 
2. Start Vivado 2019.02 Lab software. On the TCL console, type following commands to start the hardware JTAG server and connect Xilinx Programming cable  
`[open_hw_manager]`  
`[connect_hw_server]`  
`set FSBL_FILE `**_local_path_to_your_zynqmp_fsbl.elf_**  
`set BOOT_FILE `**_local_path_to_your_BOOT.BIN_**  
 
  
3. Assuming you have only local Xilinx JTAG Programming cable (USB-II) attached, if not specificy the index  
`set HW_TARGET [lindex [get_hw_targets] 0]`   **// if there are more than one Xilinx Programmers connected to same Host system, index "0" should be updated accordingly**  
`open_hw_target $HW_TARGET`  
 
  
4. Reduce the JTAG_CLK frequency to 15MHz  
** Hardware design limits maximum JTAG clock frequency to 25MHz. The next available frequency option available on Xilinx tool is 15MHz  
`set_property PARAM.FREQUENCY 1500000 $HW_TARGET`  
  
 
5. Select Bittware 250-soc card  
`set HW_DEVICE [lindex [get_hw_devices xczu19_0] 0]`  
`current_hw_device [get_hw_devices xczu19_0]`  
`refresh_hw_device -update_hw_probes false $HW_DEVICE`  
 
 
6. create/attach Flash device  
`create_hw_cfgmem -hw_device $HW_DEVICE [lindex [get_cfgmem_parts {mt25qu01g-qspi-x4-single}] 0]`    
`set_property PROGRAM.FILES [list $BOOT_FILE] [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`    **// $BOOT_FILE points to BOOT.BIN defined in Step-1**    
`set_property PROGRAM.ZYNQ_FSBL ${FSBL_FILE} [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`      **// $FSBL_FILE points to zynqmp_fsbl.elf defined in Step-1**  
`set_property PROGRAM.ADDRESS_RANGE  {use_file} [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
`set_property PROGRAM.BIN_OFFSET {0} [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
`set_property PROGRAM.BLANK_CHECK  0 [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
`set_property PROGRAM.ERASE  1 [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
`set_property PROGRAM.CFG_PROGRAM  1 [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
`set_property PROGRAM.VERIFY  0 [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`    **// setting PROGRAM.VERIFY to 1 will verify programming of bitstream to FPGA but it may double the duration of the entire sequence **    
`set_property PROGRAM.CHECKSUM  0 [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
 
 
7. Start programming (may take few hours)    
`startgroup`  
`program_hw_cfgmem -hw_cfgmem [ get_property PROGRAM.HW_CFGMEM $HW_DEVICE]`  
 
 
8. Update Switch SW1 on 250-SoC to Boot from QSPI   
   | SW1   | FPGA Signal | Left (0=Off) | Right (1=On) | 
   |-------|-------------|--------------|--------------|
   | SW1.1 | PCIE_CLK_SEL|              | On: PCIe Reference clock generate on-board |
   | SW1.2 | MOD1        | Off          |              |
   | SW1.2 | MOD2        | Off          |              |
   | SW1.3 | PERST_DIR   |              | On: PERST is driven by FPGA |
 
- SW1 located on backside of 250-SoC card to boot from QSPI    
- Power cycle the 250-SoC for changes to take effect  


9. Connect micro-USB cable to serial console located on bracket of 250-SoC  
`Serial device= /dev/ttyUSB(N)`   // 'N' is your serial console terminal  
`Baud = 115200 8N1`  
`Harware flow control = No`  
`Software flow control = No`  


    You should not see Zynq MPSoC FPGA booting the u-boot  


  
---  
  
  
[linux-genz]: https://github.com/linux-genz/linux  
[Orthus-hardware-reference-user-guide]: https://developer.bittware.com/products/250-soc.php
[Xilinx-u-boot]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842223/U-boot  
[Xilinx-linux-driver]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841873/Linux+Drivers  
[Xilinx PetaLinux]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842250/PetaLinux
[Host FPGA datasheets]: https://www.xilinx.com/support/documentation-navigation/silicon-devices/soc/zynq-ultrascale-plus-mpsoc.html
