---
title: How to rescue corrupted BIOS ROM file (EC bit control)
parent: Debug Cases
layout: default
nav_order: 1
---

# Problem Description
- COMe motherboard module BIOS file corrupt, customer cannot boot-up the M/B

# Solution
Refer to x86 ACPI spec, EC register control, Linux scripts, 
we can implement below steps to rescue this issue without unmounting SPI ROM. 

- use COMe carrier board SPI ROM to open system
- after that, switch HW jumper to select module ROM and operate EC to re-detect SPI ROM
- use AMI BIOS AFU tool to flash module ROM under linux

## EC Bit operation
reference: https://uefi.org/htmlspecs/ACPI_Spec_6_4_html/12_ACPI_Embedded_Controller_Interface_Specification/embedded-controller-register-descriptions.html

### EC code
```c
/*
support EC Commands:
0x80 - Read Embedded Controller with SCI
0x81 - Write Embedded Controller with SCI
0x82 - Burst enable Embedded Controller
0x83 - Burst disable Embedded Controller
0x84 - Query Embedded Controller
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/io.h>
#include <unistd.h>
#include <getopt.h>

#define ECCmdPort       0x66
#define ECStatusPort    0x66
#define ECDataPort      0x62
#define ECInput_Full   0x2
#define ECOutput_Full   0x1

void Check_OBF(void)
{
    while(!(inb(ECStatusPort)&(ECOutput_Full)));
}

void Check_IBF(void)
{
    while(inb(ECStatusPort)&(ECInput_Full));
}

void SendEC_CMD(unsigned char regs)
{
    Check_IBF();
    outb(regs,ECCmdPort);
    Check_IBF();
}

void SendEC_data(unsigned char values)
{
    Check_IBF();
    outb(values,ECDataPort);
    Check_IBF();
}

unsigned char ReadEC_data(void)
{
    Check_OBF();
    return (inb(ECDataPort));
}

void help(void)
{
    printf("usage: port 62/66 EC RAM Read/Write function    \n"
                "\n"
                " -h, help\n"
                " -r, read EC register, for instance 0x3a   \n"
                " -w value, hex data to write to EC register\n"
                "\n"
    );
}

int main(int argc, char *argv[])
{
  int opt;

  //permission required for port 62/66
  if(ioperm(0x60, 8, 1))
  {
    perror("ioperm");
  }

  while((opt=getopt(argc, argv, "-:hr:w:"))!= -1)
  {
    switch(opt)
    {
        case 'h':
            help();
            break;
        case 'r':
            SendEC_CMD(0x80); //read EC RAM register
            SendEC_data(strtoul(optarg, NULL, 0));
            printf("%02x\n", ReadEC_data());
            break;
        case 'w':
            optind--;
            if (optind+2 == argc)
            {   
                unsigned char reg = strtoul(argv[2], NULL, 0);
                unsigned char value = strtoul(argv[3], NULL, 0);
                printf("write 0x%01x to register 0x%02x...\n", value, reg);
                SendEC_CMD(0x81); //write EC RAM register
                SendEC_data(reg);
                SendEC_data(value);
                usleep(500);
                SendEC_CMD(0x80); //read EC RAM register
                SendEC_data(reg);
                printf("%02x\n", ReadEC_data()); 
            }
            else
            {
                help();
                exit(1); 
            }
            break;
        case '?':
                help();
                exit(1);
        case 1:
                help();
                exit(1);
    }
  }
  ioperm(0x60, 8, 0);
  return 0;
}
```

## Linux Script
### How to design the auto-flash script:

1. verify bio_ec.c can work for flashing BIOS
2. design the script construction
    - need to show help information when user need
    - show script version
    - design script cmd rule
    - parse cmd and find the BIOS file want to open and flash
    - show instruction let user select carrier or module would like to flash
    - run bio_ec.c to ask EC to modify
    - run AMI tool to flash BIOS

### Script code

```bash
#!/bin/bash

echo "============================="
echo "*** WL9A3 BIOS auto-flash ***"
echo "============================="
echo "auto-flash starting"
echo "============================="

#Step 1: re-scan EC 
echo "*** Start to operate EC***"
sudo ./bio_ec -w 0x3a 1


#Step 2: Navigate to Afu folder and flash BIOS
echo "*** Start to Flash BIOS ***"
cd AfuLnx64
sudo ./afulnx_64 B248.21B /b /p /n

 
echo "============================="
echo "auto-flash completed success"
echo "============================="
```
