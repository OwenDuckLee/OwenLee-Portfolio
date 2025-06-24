---
title: I2C master program
parent: Projects Achievement
layout: default
nav_order: 1
---

# Reference
## Linux Kernel Documentation 
- i2c dev-interface → https://www.kernel.org/doc/Documentation/i2c/dev-interface
- slave-interface → https://docs.kernel.org/i2c/slave-interface.html

## Linux 
Linux i2c-tools opensource code (i2cdetect / i2cset / i2cget .etc)
https://github.com/mozilla-b2g/i2c-tools


# Program features
- this program will running on i2c master device
- let i2c master device read/write bytes to i2c slave devices

# Program Code

## C program gcc compile cmd
```bash
gcc program.c -o program -li2c
```

## i2c-testtool-owen.c
```c
// Author: Owen.lee
// Date: 2024-09-19

#include <linux/i2c-dev.h>
#include <linux/i2c.h>
// #include <i2c/smbus.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <fcntl.h>
#include <error.h>
#include <unistd.h>

#define I2C_BUS_TYPE_PCH 1
#define VERSION_MAJOR 1
#define VERSION_MINOR 0
#define VERSION_PATCH 0
#define VERSION_BUILD 1

static void helpMessage(void);
void init();

static int i2c_smbus_access(int file, char read_write, unsigned char command,
                    int size, union i2c_smbus_data *data)
{
    struct i2c_smbus_ioctl_data args;
    int err;

    args.read_write = read_write;
    args.command = command;
    args.size = size;
    args.data = data;

    err = ioctl(file, I2C_SMBUS, &args);
    if(err == -1)
        err = -errno;

    return err;
}

/*read relevant functions*/
static int i2c_smbus_read_byte_data(int file, unsigned char command, unsigned char *ucData){
    union i2c_smbus_data data;

    if(i2c_smbus_access(file, I2C_SMBUS_READ, command, 
                            I2C_SMBUS_BYTE_DATA, &data)){
            printf("i2c_smbus_access error\n");
            return -1;
    }else{
        *ucData = data.byte;
    }

    return 0;
}

static int i2c_smbus_read_word_data(int file, unsigned char command, unsigned char *ucData){
	union i2c_smbus_data data;

	if(i2c_smbus_access(file, I2C_SMBUS_READ, command,
			                 I2C_SMBUS_WORD_DATA, &data)){
            printf("i2c_smbus_access error\n");
            return -1;
    }else{
        *ucData = data.word;
    }

    return 0;
}

static int i2c_smbus_read_block_data(int file,unsigned char command){

	union i2c_smbus_data data;
	int i = 0;
	data.block[0] = 16;
	
	if(i2c_smbus_access(file,I2C_SMBUS_READ,command,
                            I2C_SMBUS_I2C_BLOCK_DATA,&data)){             
        	printf("i2c_smbus_access error\n");
        	return -1;
	}else{
		
		for(i = 0;i < 16;i++){
			printf("[%d]: 0x%02X\n",i,data.block[1 + i]);
		}
	}
	
	return 0;
}

/*write relevant functions*/
static int i2c_smbus_write_byte_data(int file,unsigned char command,unsigned char ucData){

	union i2c_smbus_data data;
	data.byte = ucData;
	
	if(i2c_smbus_access(file,I2C_SMBUS_WRITE,command,
                            I2C_SMBUS_BYTE_DATA,&data)){             
        	printf("i2c_smbus_access error\n");
        	return -1;
	}else{
		printf("i2c_smbus_access success\n");
	}
	return 0;
}

static int i2c_smbus_write_word_data(int file, unsigned char command, unsigned char ucData){

	union i2c_smbus_data data;
	data.word = ucData;

    if(i2c_smbus_access(file,I2C_SMBUS_WRITE,command,
                            I2C_SMBUS_WORD_DATA,&data)){             
        	printf("i2c_smbus_access error\n");
        	return -1;
	}else{
		printf("i2c_smbus_access success\n");
	}
	return 0;
}

static int i2c_smbus_write_block_data(int file,unsigned char command){

	union i2c_smbus_data data;
	int i = 0;
	data.block[0] = 16;

	for(i = 0;i < 16;i++){
		data.block[1 + i] = 0x50 + i;
	}
	
	
	if(i2c_smbus_access(file,I2C_SMBUS_WRITE,command,
                            I2C_SMBUS_I2C_BLOCK_DATA,&data)){             
        	printf("i2c_smbus_access error\n");
        	return -1;
	}else{
		printf("i2c_smbus_access success\n");
	}
	
	return 0;
}

int main(int argc, char *argv[]){
    /*variable for i2c bus and slave device*/
    int file, addr, result, func_select;
    const char *i2c_bus_path = argv[1];
    unsigned char ucTemp, ucCmd;

    init();
    if(argc < 5){
        helpMessage();
        return -1;
    }

    file = open(i2c_bus_path, O_RDWR);
    if(fd < 0){
        printf("open %s failed", i2c_bus_path);
        exit(1);
    }

    addr = strtol(argv[3], NULL, 16);
    ucCmd = strtol(argv[4], NULL, 16);

    /*access dedicated i2c bus salve device address*/
    result = ioctl(file, I2C_SLAVE, addr);
    if(result < 0){
        printf("ioctl i2c slave address:%d fail, result: %d\n", addr, result);
    }
    printf("success to detect address: %d, return result: %d", addr, result);

    func_select = atoi(argv[2]);

    switch(func_select){
	
		case 1:
			i2c_smbus_read_byte_data(file,ucCmd,&ucTemp);
			printf("DATA: 0x%02X\n",ucTemp);
		break;
           
		case 2:
             i2c_smbus_read_word_data(file, ucCmd);
		break;
	
		case 3:
			i2c_smbus_read_block_data(file,ucCmd);		
		break;
       
        case 5:
            ucTemp = strtol(argv[5],NULL,16);
			i2c_smbus_write_byte_data(file,ucCmd,ucTemp);
		break;
		
		case 6:
            i2c_smbus_write_word_data(file, ucCmd, ucTemp);
		break;
	
		case 7:
		    i2c_smbus_write_block_data(file,ucCmd);
		break;
		
		default:
			helpMessage();
			return -1;
		break;
	}
	

	return 0;
}

static void helpMessage(void){

	printf("\n=========================================Help==========================================\n");
	printf(" ./a.out <i2c-dev> testcase DeviceAddr Cmd DATA                                          \n");
	printf(" example: ./a.out /dev/i2c-2 1 0x50 0x03 : Read Device:0x50 cmd:0x03 1 Byte from i2c-2.  \n");
	printf("                                                                                         \n");
	printf(" ./a.out <i2c-dev> 1 DeviceAddr Cmd      : read 1 byte from Cmd offset                   \n");
	printf(" ./a.out <i2c-dev> 2 DeviceAddr Cmd DATA : read 4 byte word DATA from Cmd offset         \n");
	printf(" ./a.out <i2c-dev> 3 DeviceAddr Cmd      : read 16 byte block data from Cmd offset.      \n");
	printf(" ./a.out <i2c-dev> 5 DeviceAddr Cmd      : write 1 byte to Cmd offset                    \n");
	printf(" ./a.out <i2c-dev> 6 DeviceAddr Cmd DATA : write 4 byte word DATA to Cmd offset          \n");
	printf(" ./a.out <i2c-dev> 7 DeviceAddr Cmd      : write 16 byte block data to Cmd offset.       \n");
	printf("\n=========================================Help==========================================\n");

	return;
}

void init(){
    
    //show software version
    printf("[ I2C control test software version: %d.%d.%d.%d]\n", 
           VERSION_MAJOR, VERSION_MINOR, VERSION_PATCH, VERSION_BUILD);

    //set I2C bus type. right now is only PCH, might add EC option for selected in future.
	printf("******** <Bus Type ID> ********\n");
    printf("This tool is for testing I2C Bus Type:PCH read/write.\n");
    printf("*******************************\n");
}
```
