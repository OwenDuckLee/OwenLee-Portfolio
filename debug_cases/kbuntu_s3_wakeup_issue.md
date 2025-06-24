---
title: Kubuntu USB port and WLAN S3 wake-up issue
parent: Debug Cases
layout: default
nav_order: 2
---

# Problem Description
- Environment duplicate setup
  M/B: ADP968
  Carrier Board: Client customized
  BIOS:
  EC:
  OS: Kbuntu 20.04 / Kbuntu 22.04
- Client Carrier Board setup for external devices
  1. LVDS LCD intergrated within system
  2. Display port (DDI1_PAIR0_P/N - DDI1_PAIR3_P/N, 4 lanes) via USB-C connector
  3. keyboard and mouse through USB C also
  4. External GPU (PEG_TX0_P/N - PEG_TX3_P/N, 4 lanes)
 

  # Issue Verifying

  ## Use Linux cmd to verifying system
  - check grub cmdline
    cat /proc/cmdline
    
  - confirm pci device
    lspci
    
  - confirm pci device power status
    cat /sys/bus/pci/devices/0000:XX:YY.Z/power/runtime_status
    
  - after wakeup, confirm wifi connect
    nmcli device wifi list
    nmcli device wifi connect <SSID> password <PASSWORD>

  ## Verifying Test
  1. Confirm if system running Ubuntu can enter S3 then wakeup successfully
  2. Kbuntu 22.04 + Qualcom WLAN can enter S3 then wakeup success
  3. add grub parameter still can't help client system wake-up from S3
     GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_port_pm=off"
  4. discover problem at dmesg log "[   44.635873] mhi mhi0: Resuming from non M3 state (RESET)"
     suspect WLAN on client's carrier board did not get into sleep mode(M3 low power)
     measure 3.3V, 5V, SUS_S3# signal by scope
  5. check client's carrier board schematic of USB-hub,
     and discover one of MCU on carrier board impact USB-hub
  6. Implement below test steps to verify the MCU
     - cut MCU power (remove or mask L1501), then let system go to S3 and turn back to S0 to see if the issues still remain
     - According to schematic, when system go to S3, the MCU still remaining the power and keep transfer signal to USB hub
     - when system return from S3 to S0, the USB hub might not do correctively reset process due to MCU hold something signals which caused other devices abnormal

      ![image](https://github.com/user-attachments/assets/e858e187-fb60-4526-ab9d-8652eb2e9407)
      ![image](https://github.com/user-attachments/assets/b6a0a7c9-60a4-4b6c-a6ab-69cdf8052518)



  #Solution
  - We deliver verifying test process and suggestion to client for WLAN and MCU on their carrier board
  - Client refer to our suggestion to replace WLAN and modify MCU wire design
