---
title: STM32 Slave device(simulate ROM) - I2C/SMBus transmission
parent: Projects Achievement
layout: default
nav_order: 3
---

# Description
- use dev board STM32F103C8T6 to simulate ROM for I2C/SMBus transmission

# STM32 HW setup
- enable pin PB7 as I2C SDA
- enable pin PB6 as I2C SCL
- find internal LED pin PC13 and select as GPIO_Output

# Program Features
- Simulate ROM as data buffer for read/write
- control STM32 GPIO internal LED
- control STM32 I2C
- transmit data by I2C to control on/off internal LED
- transmit data by I2C to control on/off FAN(TBD)

# Program code
- Need to follow master transmition speed (100KHz or 400KHz)
- then set speed in IDE to re-generate stm32 code

## main.c
- main.c file was generated after you completed setup in stm32Cube IDE

```c
/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Main program body
  ******************************************************************************
  * @attention
  *
  * Copyright (c) 2024 STMicroelectronics.
  * All rights reserved.
  *
  * This software is licensed under terms that can be found in the LICENSE file
  * in the root directory of this software component.
  * If no LICENSE file comes with this software, it is provided AS-IS.
  *
  ******************************************************************************
  */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */

/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/
I2C_HandleTypeDef hi2c1;

/* USER CODE BEGIN PV */

/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_I2C1_Init(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{

  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_I2C1_Init();
  /* USER CODE BEGIN 2 */
  if(HAL_I2C_EnableListen_IT(&hi2c1) != HAL_OK )
  {
	  Error_Handler();
  }
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief I2C1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_I2C1_Init(void)
{

  /* USER CODE BEGIN I2C1_Init 0 */

  /* USER CODE END I2C1_Init 0 */

  /* USER CODE BEGIN I2C1_Init 1 */

  /* USER CODE END I2C1_Init 1 */
  hi2c1.Instance = I2C1;
  hi2c1.Init.ClockSpeed = 100000;
  hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
  hi2c1.Init.OwnAddress1 = 66;
  hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
  hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
  hi2c1.Init.OwnAddress2 = 0;
  hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
  hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
  if (HAL_I2C_Init(&hi2c1) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN I2C1_Init 2 */

  /* USER CODE END I2C1_Init 2 */

}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};
/* USER CODE BEGIN MX_GPIO_Init_1 */
/* USER CODE END MX_GPIO_Init_1 */

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOC_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(FAN_GPIO_Port, FAN_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin : LED_Pin */
  GPIO_InitStruct.Pin = LED_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(LED_GPIO_Port, &GPIO_InitStruct);

  /*Configure GPIO pin : FAN_Pin */
  GPIO_InitStruct.Pin = FAN_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(FAN_GPIO_Port, &GPIO_InitStruct);

/* USER CODE BEGIN MX_GPIO_Init_2 */
/* USER CODE END MX_GPIO_Init_2 */
}

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */
```

## i2c-slave.h

```c
/*
 * i2c_slave.h
 *
 *  Created on: Dec 13, 2024
 *      Author: owen.lee
 */

#ifndef INC_I2C_SLAVE_H_
#define INC_I2C_SLAVE_H_

#include "stm32f1xx_hal.h"
#include <stdbool.h>

bool process_received_gpio_group_task;

/*call back function*/
void HAL_I2C_ListenCpltCallback(I2C_HandleTypeDef *hi2c);
void HAL_I2C_AddrCallback(I2C_HandleTypeDef *hi2c, uint8_t TransferDirection, uint16_t AddrMatchCode);
void HAL_I2C_SlaveRxCpltCallback(I2C_HandleTypeDef *hi2c);
void HAL_I2C_SlaveTxCpltCallback(I2C_HandleTypeDef *hi2c);
void HAL_I2C_ErrorCallback(I2C_HandleTypeDef *hi2c);


/*execution function*/
void control_gpio_group_task1(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value);
void control_gpio_group_task2(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value);
void control_gpio_group_task3(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value);
void process_gpio_group_task(void);
bool is_i2cdump_tool_active(void);


#endif /* INC_I2C_SLAVE_H_ */
```
## i2c-slave.c

```c
/*
 * i2c_slave.c
 *
 *  Created on: Dec 13, 2024
 *      Author: owen.lee
 */

#include <i2c-slave.h>
#include "main.h"

extern I2C_HandleTypeDef hi2c1;

/*store data*/
#define BUFFER_SIZE 1024
static uint8_t DataBuffer[BUFFER_SIZE] = {0};
uint16_t buffer_index = 0;

uint8_t register_address = 0; //default register address is 0x00

int rx_count = 0;
int tx_count = 0;

uint16_t bytesToSend = 16; //I2C -> 32, SMBus -> 16

/*GPIO group control*/
#define LED_GROUP_START 0x10
#define FAN_GROUP_START 0x00
#define MOTOR_GROUP_START 0x20
#define GPIO_GROUP_SIZE 8 // Assume each group controls up to 8 GPIO pins

void control_gpio_group_task1(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value) {
    // Iterate over each pin in the pin_mask and set/reset based on the value
//    for (uint16_t pin = 0x01; pin <= GPIO_PIN_15; pin <<= 1) {
//        if (pin_mask & pin) {
//            if (value == 0x01) {
//                HAL_GPIO_WritePin(GPIOx, pin, GPIO_PIN_SET);
//            } else if (value == 0x00) {
//                HAL_GPIO_WritePin(GPIOx, pin, GPIO_PIN_RESET);
//            }
//        }
//    }
	  if (value == 0x01) {
//			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_SET);
			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_RESET); //reverse the LED light logic
		} else if (value == 0x00) {
//			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_SET); //reverse the LED light logic
		}

}

void control_gpio_group_task2(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value) {
	  if (value == 0x01) {
//			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_SET);
			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_RESET);
		} else if (value == 0x00) {
//			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(GPIOx, pin_mask, GPIO_PIN_SET);
		}

}

void control_gpio_group_task3(GPIO_TypeDef *GPIOx, uint16_t pin_mask, uint8_t value) {

}


void process_gpio_group_task(void){
	  uint8_t value = DataBuffer[register_address];

			if (register_address >= LED_GROUP_START && register_address < LED_GROUP_START + GPIO_GROUP_SIZE) {
				control_gpio_group_task1(GPIOC, LED_Pin, value);
			} else if (register_address >= FAN_GROUP_START && register_address < FAN_GROUP_START + GPIO_GROUP_SIZE) {
				control_gpio_group_task2(GPIOC, FAN_Pin, value);
			} else if (register_address >= MOTOR_GROUP_START && register_address < MOTOR_GROUP_START + GPIO_GROUP_SIZE) {
//				control_gpio_group_task3(GPIOC, MOTOR_Pin, value);
			}
}


bool is_i2cdump_tool_active(void);


void HAL_I2C_ListenCpltCallback(I2C_HandleTypeDef *hi2c)
{
	HAL_I2C_EnableListen_IT(hi2c);
}

void HAL_I2C_AddrCallback(I2C_HandleTypeDef *hi2c, uint8_t TransferDirection, uint16_t AddrMatchCode)
{

		if(TransferDirection == I2C_DIRECTION_TRANSMIT)
		{
			HAL_I2C_Slave_Sequential_Receive_IT(hi2c, &register_address, 1, I2C_FIRST_FRAME);
		}
		else if (TransferDirection == I2C_DIRECTION_RECEIVE)
		{
			//HAL_I2C_Slave_Sequential_Transmit_IT(hi2c, &DataBuffer[register_address], 1, I2C_FIRST_AND_LAST_FRAME);
			if (register_address < BUFFER_SIZE)
			{
				if (is_i2cdump_tool_active()) // for handling tools like i2cdump
				{
					bytesToSend = BUFFER_SIZE - register_address;
				}

				if (bytesToSend > BUFFER_SIZE - register_address) {
					bytesToSend = BUFFER_SIZE - register_address; // Ensure you don't read past the buffer size
				}

				HAL_I2C_Slave_Sequential_Transmit_IT(hi2c, &DataBuffer[register_address], bytesToSend, I2C_FIRST_AND_LAST_FRAME);
			}else{
				uint8_t dummyAck = 0; // Dummy data to acknowledge but send nothing
				HAL_I2C_Slave_Sequential_Transmit_IT(hi2c, &dummyAck, 1, I2C_FIRST_AND_LAST_FRAME); // ACK only, no data
			}
		}
	//}
}

void HAL_I2C_SlaveRxCpltCallback(I2C_HandleTypeDef *hi2c)
{
		rx_count ++;
    if (register_address < BUFFER_SIZE)
    {
        uint16_t remaining_buffer_size = BUFFER_SIZE - register_address;

        // 決定實際接收的資料量
        uint16_t bytes_to_receive = (bytesToSend < remaining_buffer_size) ? bytesToSend : remaining_buffer_size;

        // 接收 Master 傳來的多 byte 資料並儲存到 buffer
        HAL_I2C_Slave_Sequential_Receive_IT(hi2c, &DataBuffer[register_address], bytes_to_receive, I2C_NEXT_FRAME);
    }

	process_received_gpio_group_task = true;

	HAL_I2C_EnableListen_IT(hi2c);
}

void HAL_I2C_SlaveTxCpltCallback(I2C_HandleTypeDef *hi2c)
{
	tx_count ++;
	HAL_I2C_EnableListen_IT(hi2c);
}

void HAL_I2C_ErrorCallback(I2C_HandleTypeDef *hi2c)
{
	HAL_I2C_EnableListen_IT(hi2c);
}

bool is_i2cdump_tool_active(void)
{
    static uint8_t previous_register_address = 0;
    static uint8_t consecutive_reads = 0;

    if (register_address == previous_register_address + 1)
    {
        consecutive_reads++;
    }
    else
    {
        consecutive_reads = 0; // Reset if not sequential
    }

    previous_register_address = register_address;

    // If master is reading sequential data, return true (i.e., i2cdump)
    return consecutive_reads > 10; // Threshold to detect consecutive reads (adjustable)
}
```

# Reference
- how to setup stm32 I2C → https://medium.com/閱益如美/stm32-20-i2c-c4c9ce6d1c3a
- https://github.com/tunerok/stm32_i2c_slave_examlpe/blob/main/stm32_i2c_slave_example/Core/Src/i2c_slave.c
