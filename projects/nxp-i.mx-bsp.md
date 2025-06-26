
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
    2. Create Yocto environment
    3. Download NXP i.MX Yocto BSP
    4. Switch to python2
    5. Initialize Yocto environment
    6. Switch back to python3
    7. Compile Yocto Image
    8. Flash Image into SD card


# Porting Custom-Board DTS U-Boot
