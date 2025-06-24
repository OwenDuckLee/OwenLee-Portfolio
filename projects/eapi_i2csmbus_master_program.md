---
title: EAPI SDK - I2C/SMBus master program
parent: Projects Achievement
layout: default
nav_order: 2
---
# Description
- Here are two verifying-test program
- First one is: TransmitHugeFile
- Second one is: WriteReadCompare
- both of these programs are implemented by EAPI SDK

# TransmitHugeFile.c
## Program features
- select which protocol(I2C/SMBus) want to test
- select which speed want to test(100KHz / 400KHz)

## Program Code
- include "EApi.h"
- need to link libEAPI_Library.so dynamic library
- compiling and testing cmd
  ```bash
    $ cd project_folder
    $ sudo chmod 777 *
    $ gcc -o runTest TransmitHugeFile.c -I -L libEAPI_Library.so
    $ sudo ./runTest
  ```

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <math.h>
#include <time.h>
#include "EApi.h" 

/*define the testing parameter*/
#define I2C_BUS_ID 	3 
#define I2C_SLAVE_ADDR 	0x42  	
/*
 * I2C slave address
 * (e.g stm32 7bits address 0x21 -> EAPI need to use 8bit address: 0x42)
 * */

#define FILE_SIZE 	270000 	//huge file size
#define COMMAND 	0	

/*I2C transmition abstraction function*/
void sendFileOverI2C() {
    /*huge file simulation*/
    uint32_t dataLength = FILE_SIZE;
    uint8_t * testFileBuffer = (uint8_t *)malloc((size_t)dataLength * sizeof(uint8_t));

    /*initialize huge file content*/
    for (uint32_t i = 0; i < FILE_SIZE; i++) {
        testFileBuffer[i] = (uint8_t)(i % 256);
    }

    uint32_t bytesSent = 0; //the count of bytes has wrote
    EApiStatus_t ret;

    struct timespec start_time, end_time;

    printf("Starting I²C file transfer...\n");
    if(clock_gettime(CLOCK_MONOTONIC, &start_time) == -1) {
    	perror("clock_gettime"); 
    	free(testFileBuffer);
    	return;
    }

    while (bytesSent < FILE_SIZE) {
      //count of bytes for current need to write
	    uint32_t bytesToSend = 32;

      //use EAPI I2C write function write to slave
      ret = EApiI2CWriteTransfer(
              I2C_BUS_ID,
              I2C_SLAVE_ADDR,
	            COMMAND,
              testFileBuffer + bytesSent,
	            bytesToSend
            );

      if (!EAPI_TEST_SUCCESS(ret)) {
        fprintf(stderr, "Error: I²C Write failed at byte %u\n", bytesSent);
        printf("%s(%d): EApiI2CWriteTransfer error ... (error code 0x%X)\n", __FUNCTION__, __LINE__, ret);
        return;
      }

      printf("Transferred %u bytes to I²C Slave.\n", bytesToSend);
      bytesSent += bytesToSend;
    }

    if(clock_gettime(CLOCK_MONOTONIC, &end_time) == -1) {
	    perror("clock_gettime");
	    free(testFileBuffer);
	    return;
    }

    double elapsed_time = (end_time.tv_sec - start_time.tv_sec) +
	    		                (end_time.tv_nsec - start_time.tv_nsec) / 1e9;

    printf("File transfer completed. Total bytes sent: %u\n", FILE_SIZE);
    printf("Elapsed time: %.6f seconds\n", elapsed_time);
    free(testFileBuffer);
    return;
}

int main() {
    EApiStatus_t ret;
    //EAPI initialize
    if ((ret = EApiLibInitialize()) != EAPI_STATUS_SUCCESS) {
	      printf("%s(%d): EApiLibInitialize error ... (error code 0x%X)\n", __FUNCTION__, __LINE__, ret);
	      return -1;
    }
    //start to test i2c transmition of huge file
    sendFileOverI2C();

    return 0;
}
```


# WriteReadCompare.c
## Program features
- select which protocol(I2C/SMBus) want to test
- select which speed want to test(100KHz / 400KHz)

## Program Code
- include "EApi.h"
- need to link libEAPI_Library.so dynamic library
- compiling and testing cmd
  ```bash
    $ cd project_folder
    $ sudo chmod 777 *
    $ gcc -o runTest WriteReadCompare.c -I -L libEAPI_Library.so
    $ sudo ./runTest
  ```

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <math.h>
#include <time.h>
#include "EApi.h" 

/*define the testing parameter*/
#define COMMAND 	0	
#define I2C_BUS_ID 	4
#define I2C_SLAVE_ADDR 	0x42  	
/*
 * I2C slave address
 * (e.g stm32 7bits address 0x21 -> EAPI need to use 8bit address: 0x42)
 * */

#define BUFFER_SIZE 256
#define TXRX_CHUNK_SIZE 16
#define CYCLE_COUNT 4

uint8_t fullBuffer[BUFFER_SIZE];

void initBuffer(uint8_t *buffer, size_t size) {
	srand(time(NULL));
	for(size_t i = 0; i < size; i++) {
    buffer[i] = rand() % 256;
  }
}

int compareRxTxBuffer(const uint8_t *rxBuffer, const uint8_t *txBuffer, size_t size) {
	for(size_t i = 0; i < size; i++){
		if(rxBuffer[i] != txBuffer[i])
			return 0;
	}

	return 1;
}


int performTestCycle() {
  EApiStatus_t ret;
	uint32_t bufferSize = TXRX_CHUNK_SIZE; 
	uint8_t * txBuffer = (uint8_t *)malloc((size_t)bufferSize * sizeof(uint8_t)); 
	uint8_t * rxBuffer = (uint8_t *)malloc((size_t)bufferSize * sizeof(uint8_t));

	for(size_t offset = 0; offset < BUFFER_SIZE; offset += TXRX_CHUNK_SIZE) {
		//init txBuffer
		memcpy(txBuffer, fullBuffer + offset, TXRX_CHUNK_SIZE);

		//write txBuffer data to i2c slave
		ret = EApiI2CWriteTransfer(
            		I2C_BUS_ID,
            		I2C_SLAVE_ADDR, 
			          COMMAND,
            		txBuffer,
            		TXRX_CHUNK_SIZE	
        	);
    if (!EAPI_TEST_SUCCESS(ret)) {
          fprintf(stderr, "Error: I2C Write failed at offset %zu\n", offset);
          printf("%s(%d): EApiI2CWriteTransfer error ... (error code 0x%X)\n", __FUNCTION__, __LINE__, ret);
          return -1;
    }

		//read i2c slave data to rxBuffer
		ret = EApiI2CReadTransfer(
                I2C_BUS_ID,
                I2C_SLAVE_ADDR,
                COMMAND,
                rxBuffer,
            		TXRX_CHUNK_SIZE,
            		TXRX_CHUNK_SIZE	
          );
    if (!EAPI_TEST_SUCCESS(ret)) {
          fprintf(stderr, "Error: I2C Read failed at offset %zu\n", offset);
          printf("%s(%d): EApiI2CReadTransfer error ... (error code 0x%X)\n", __FUNCTION__, __LINE__, ret);
          return -1;
    }

		//compare rxBuffer and txBuffer
		if(!compareRxTxBuffer(rxBuffer, txBuffer, TXRX_CHUNK_SIZE)) {
        printf("Buffer mismatch at offset %zu\n", offset);
			  return -1;
		}

		printf("Chunk at offset %zu passed validation.\n", offset);
  }

	free(rxBuffer);
	free(txBuffer);
	return 0;
}

int main() {
  EApiStatus_t ret;
  //EAPI initialize
  if ((ret = EApiLibInitialize()) != EAPI_STATUS_SUCCESS)
  {
      printf("%s(%d): EApiLibInitialize error ... (error code 0x%X)\n", __FUNCTION__, __LINE__, ret);
      return -1;
  }

	//perform verifying test for cycle count
	for(int cycle = 1; cycle <= CYCLE_COUNT; cycle++) {
		//init fullBuffer
		initBuffer(fullBuffer, BUFFER_SIZE);

		printf("Starting Cycle %d\n", cycle);	
		if(performTestCycle() != 0) {
			printf("Cycle %d failed.\n", cycle);
			return -1;
		}
		printf("Cycle %d completed successfully.\n", cycle);
	}

	printf("All cycles completed successfully.\n");
	return 0;
}
```
