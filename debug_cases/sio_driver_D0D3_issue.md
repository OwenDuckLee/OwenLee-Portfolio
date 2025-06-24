---
title: Winbod SIO Driver read voltage value issue in OS power saving mode
parent: Debug Cases
layout: default
nav_order: 3
---

# Problem Description
## System Environment
- System: AVP4.0
- OS: Windows10
- Driver: Winbond SIO Driver customized by Client

## Issue Description
- When system boot-up, driver can read voltage value from SIO correctly
- OS go to power saving mode S3 about 1 min, then turn back to S0: looping this cycle and can read voltage correctly
- After 1hr: driver read voltage value is out of range
- After 4hr: driver read voltage value is out of range

# Verifying Test

## Suspection of bug in SIO driver
  1. Driver internal state machine abnormal, or queue buffer overflow/corruption
  2. driver.c or device.c , where control power management missing
  3. device.c PortIOEvtDeviceCreate()中的power management object was marked
  4. device.c D0Entry() need to remapping SIO register 

## Read Winbond SIO W83627DHG-P data sheet to confirm how to control SIO behavior in D0 D3 mode
- Below are reference screen shot from W83627DHG-P Data Sheet#
 ![image](https://github.com/user-attachments/assets/f866454d-987e-4a61-a304-d155755e6fc6) #p.28.29
 ![image](https://github.com/user-attachments/assets/dbe7e28a-f1f1-4c7b-b13a-5e538ed10798) #p.28
 ![image](https://github.com/user-attachments/assets/f459a8f5-909a-4b25-8dd6-caa3a4c74b44) #p.88
 ![image](https://github.com/user-attachments/assets/b1f7332e-70c8-4473-bd0a-928e59b78bf1) #p.78


## Measure HW signal when driver read abnormal value
  Measure HW ADC signal(12V, 5V, 3.3V) by Oscilloscope, 
  -	monitor each voltage signal if is wrong while voltage values(output from SIO driver) is out of range?
  >> if ADC signal all correct while switching between D0 – D3, we should dedicated investigate on driver level firstly.
  >> if ADC signal occurs abnormal, then we should direction to investigate on HW/BIOS firstly.
  ![image](https://github.com/user-attachments/assets/4c5cf763-82f1-4194-b823-39d31414eb9d)

## Final modification in SIO Driver device.c 
- PortIODeviceCreate() -> re-added the initialization of the power management object.
- PortIOEvtDeviceD0Entry() -> add reset SIO registers to LDN and bank0 while each time go back to D0 state.


# Solution
- submit Winbond SIO W83627DHG-P data sheet section for how to re-init SIO HWM LDN
- help client re-tracing dirver code
- submit SIO driver modified code for client(re-add power management object / add reset SIO registers to LDN and bank0)
