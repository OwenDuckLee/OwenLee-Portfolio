---
title: NXP SoC BSP - Bring-up Image
parent: Projects Achievement
layout: default
nav_order: 6
---

# Project Description
## Project Goals
- Learn how to port new devices for SoC
- Modify U-Boot and Device Tree Source(DTS)
- Customized device drivers

## Actual Actions
- Completed NXP i.MX8 BSP Build Image using kernel imx-4.14.98-2.3.5
- Custom-Board porting DTS U-Boot

# Build Image SOP for i.MX8 (Kernel version 4.14.98-2.3.5)
## Develop Machine Environment Setup
    - Yocto version 2.5(Thud)
    - OS version: Ubuntu22.04
    - NXP i.MX Release Manifest: imx-4.14.98-2.3.5.xml

## Build Image Steps
1. Download repo tool
    ```bash
    $ mkdir ~/bin
    $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo  > ~/bin/repo
    $ chmod a+x ~/bin/repo
    $ PATH=${PATH}:~/bin
    ```
2. Create Yocto environment
    ```bash
    $ mkdir -p ~/yocto-imx8-version && cd ~/yocto-imx8-version
    ```
3. Download NXP i.MX Yocto BSP
    ```bash
    $ repo init -u https://github.com/nxp-imx/imx-manifest  -b imx-linux-sumo -m imx-4.14.98-2.3.5.xml
    $ repo sync
    ```
4. Switch to python2
    ```bash
    $ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    $ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
    $ sudo update-alternatives --config python
    ```
5. Initialize Yocto environment
    ```bash
    MACHINE=imx8mqevk DISTRO=fsl-imx-wayland source ./fsl-setup-release.sh -b bld-wayland
    ```
6. Switch back to python3
    ```bash
    $ sudo update-alternatives --config python
    ```
7. Compile Yocto Image
    ```bash
    bitbake <image recipe>
    #core-image-minimal : small console based image
    #fsl-image-gui - full image with demos and tests used for testing with graphics, no QT.
    ```
8. Flash Image into SD card
    
# Porting Custom-Board DTS U-Boot
- Take m8m051 as example
- Follow instructions of NXP UG10165 Porting guide
  
## Porting Kernel Steps

> !! Never modify .dtsi file !!
>

1. Choose to build and load kernel in Yocto Project
2. Go to build directory
    ```bash
    $ cd ~/yocto-imx/bld-wayland
    ```
3. Firstly, build a reference board kernel for associated SoC(e.g imx8mqevk)
    ```bash
    $ MACHINE=imx8mqevk bitbake linux-imx
    ```
4. Create custom layer for holding custom board kernel changes
    ```bash
    $ bitbake-layers create-layer m8m051
    $ bitbake-layers add-layer ~/yocto-imx8/sources/m8m051/
    $ bitbake-layers show-layers
    
    # remove layer can call below command
    # $ rm -rf ~/yocto-imx8/sources/m8m051/
    ```
5. Copy reference imx8mqevk.conf to custom_board layer
    ```bash
    $ mkdir -p ~/yocto-imx8/sources/m8m051/conf/machine
    $ cp ~/yocto-imx8/sources/meta-fsl-bsp-release/imx/meta-bsp/conf/machine/imx8mqevk.conf\
          ~/yocto-imx8/sources/m8m051/conf/machine/imx-m8m051.conf
    ```
6. Edit the machine configuration file with device trees listed in the KERNEL_DEVICETREE
    ```bash
    $ sudo vi ~/yocto-imx8/sources/m8m051/conf/machine/imx-m8m051.conf
    ```
    ```bash
    # modify this line for changing to use customized dtb #
    #KERNEL_DEVICETREE = "freescale/fsl-imx8mq-evk.dtb freescale/fsl-imx8mq-evk-ak4497.dtb "
    KERNEL_DEVICETREE = "freescale/fsl-imx8mq-m8m051.dtb freescale/fsl-imx8mq-evk-ak4497.dtb "
    
    # modify this line to set u-boot DTB name after bitbake linux-imx #
    #UBOOT_DTB_NAME = "fsl-imx8mq-evk.dtb"
    UBOOT_DTB_NAME = "fsl-imx8mq-m8m051.dtb"
    ```
7. Change the preferred version for kernel to build with linux-imx
    ```bash
    $ cd ~/yocto-imx8/bld-wayland/conf
    $ sudo vi local.conf
    ```
    
    ```bash
    #add this line to conf/local.conf
    #this forces the linux-imx version to be used
    ## PREFERRED_PROVIDER_virtual/kernel_<custom_name> = "linux-imx"
    PREFERRED_PROVIDER_virtual/kernel_imx-m8m051 = "linux-imx"
    PREFERRED_PROVIDER_virtual/bootloader_imx-m8m051 = "u-boot-imx"
    ```
8. Build custom machine first time for having work directory
    
    ```bash
    $ cd ~/yocto-imx8
    $ MACHINE=imx-m8m051 bitbake linux-imx
    ```
9. Copy reference .dts file for custom layer
    ```bash
    $ cd ~/yocto-imx8/bld-wayland/tmp/work-shared/imx-m8m051/kernel-source/arch/arm64/boot/dts/freescale
    $ cp fsl-imx8mq-evk.dts fsl-imx8mq-m8m051.dts
    ```
10. Modify .dts device tree for custom board
    
    > go to kernel Documentation/devicetree/bingings for further information
    $cd ~/yocto-imx8/bld-wayland/tmp/work-shared/imx-m8m051/kernel-source/Documentation/devicetree/bindings/fsl-board.txt
    > 
    
    ```bash
    $ sudo vi fsl-imx8mq-m8m051.dts
    ```
    
    ```c
    //modify the hardware configurations for custom board
    //for this example, we adjust the memory for LPDDR 4GB capacity
    
    /dts-v1/;
    #include "fsl-imx8mq.dtsi"
    
    / {
    		model = "Freescale i.MX8MQ EVK";
    		compatible = "fsl, imx8mq-evk", "fsl, imx8mq";
    		
    		/*add memory node configuration for overlapping .dtsi*/
    		memory@40000000 {
    				device_type = "memory";
    				reg = <0x0 0x40000000 0x1 0x0>;
    		};
    		
    		/*another original nodes in code*/
    		
    		/* ... */
    
    ```
12. Build the custom machine
    ```bash
    $ cd ~/yocto-imx8
    $ MACHINE=imx-m8m051 bitbake linux-imx
    ```
13. Check build kernel and device tree
    ```bash
    $ ls ~/yocto-imx/bld-wayland/tmp/work/imx_m8m051-poky-linux/linux-imx/4.14.08-r0
    #can find the build output
    
    $ ls ~/yocto-imx/bld-wayland/tmp/deploy/images/imx-m8m051
    #then can find fsl-imx8mq-m8m051.dtb and build kernel image for flashing
    ```
14. Check the yocto project patch and .bbappend
    > create .bbappedn / .patch / custom_defconfig
    >
    
    ```bash
    $ cd ~/yocto-imx/sources/m8m051/recipes-kernel/linux-imx/files
    ```
    
## Porting U-Boot Steps
1. Follow UG10165 porting U-Boot section
2. Utilize porting kernel built m8m051 layer
3. Build custom machine first time for having work directory
   ```bash
   $ cd ~/yocto-imx8
   $ MACHINE=imx-m8m051 bitbake linux-imx
   ```
4. Modify U-Boot .dtsi and .dts file for porting
   ```bash
    # go to temp work folder
    # modify fsl-imx8mq-m8m051.dts for memory uart…etc
   
    $ cd ~/yocto-imx8/bld-wayland/tmp/work/imx_m8m051-poky-linux/u-boot-imx/2018.03-r0/git/arch/arm/dts
    $ cp fsl-imx8mq-evk.dts fsl-imx8mq-m8m051.dts
   ```
5. Go to below folder modify Makefile
    ```bash
    $ cd ~/yocto-imx8/bld-wayland/tmp/work/imx_m8m051-poky-linux/u-boot-imx/2018.03-r0/build/imx8mq_evk_config/source/arch/arm/dts/
    $ sudo vi Makefile
    # add fsl-imx8mq-m8m051.dtb
    ```
6. Compile U-Boot in Yocto project by bitbake
7. Build .bbappend file and .patch file for Yocto project
    ```bash
    $ mkdir -p ~/yocto-imx8/sources/m8m051/recipes-bsp/uboot-imx/files
    $ sudo vi patch1.patch
    ```

    ```bash
    $ cd ..
    $ sudo vi u-boot-imx_%.bbappend
    ```
    
    ```c
    //add below content
    //noted sumo will be different with latest UG command
    FILESEXTRAPATH_prepend := "${THISDIR}/files:"
    SRC_URI += "file://patch1.patch"
    ```
