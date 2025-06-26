---
title: NXP SoC BSP - Bring-up Image
parent: Projects Achievement
layout: default
nav_order: 6
---

# Project Description
## Project Goals
- learn how to port new devices for SoC
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

## Steps
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
