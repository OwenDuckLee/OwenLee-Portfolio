---
title: CH2.4 Module Compile and Install
parent: CH2 Module
layout: default
nav_order: 1
---

## 編譯模組

- 要編譯模組，需先準備以下工具:
    1. 編譯器 !!注意編譯器版本!!
    2. 模組管理工具
    3. 相關工具的版本

- 還需要準備材料:
    1. 系統上有一個完整的核心源碼樹
    2. 使用目標核心源碼樹 建構好的目標系統的專屬核心
    
- 為模組寫一個Makefile
    
    ```makefile
    #makefile content
    obj-m := hello.o
    
    #上述指令使用了GNU的擴充語法
    #宣稱有一個模組要從目的碼檔 hello.o建構出來
    #所產生的模組檔會是 hello.ko
    	
    #example
    obj-m := module.o
    module-objs := file1.o file2.o
    ```
    
- 如何使用Makefile
    
    ```bash
    #假設 核心源碼樹位址 ~/kernel-6.8 目錄 
    #假設 模組source位址 ~/mymodule 目錄 
    $cd ~/mymodule
    $make -C ~/kernel-6.8 M=$(shell pwd) modules
    
    ##先切換工作目錄到 -C選項所指的目錄(核心源碼的頂層目錄)
    ##在該目錄找到核心源碼頂層的makefile
    
    ##M= 選項會使該個makefile回到模組的原始程式目錄
    ##開始建構modules目標，此目標代表obj-m 變數所列的模組
    
    ```
    
- 優化減少輸入make指令的makefile 範本
    
    ```makefile
    #如果有定義KERNELRELEASE，就表示make是從核心建構系統去處理本檔案，所以我們可使用他的語言
    
    ifneq ($(KERNELRELEASE),)
    	obj-m := module.o
    
    #否則，我們是直接被命令列使用，所以要先呼叫核心建構系統
    
    else
    
    	KERNELDIR ?= /lib/modules/$(shell uname -r)/build
    	PWD := $(shell pwd)
    	
    default:
    	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
    	
    endif
    
    #完整makefile還需要加上 clean, install等目標
    ```
    

## 模組的裝載 與 卸載

### 使用 insmod 將模組載入核心

- inmod會做的工作
    - 將模組碼 與 資料載入核心
    - 執行ld功能，將模組中所有的unresolved symbol 連結到核心內部的符號表
    - inmod只會改變記憶體中的副本

- how linux kernel support insmod
    - it depend on kernel/module.c defined systemcall sys_init_module()
    - sysy_init_module() will arrange a kernel space memory( arrange by vmalloc() ) for storing module
        
        then will duplicate module .txt sector into this memory
        
        find  module unresolved symbol form kernel symbol table
        
        finally call module init function to complete the whole module installation
        

### 使用 modprobe

- modprobe能自動載入目標模組所需的其他模組
- modprobe只會搜尋已經安裝到 /lib/modules中的模組

### 使用 rmmod

## 核心版本的相依性

因為核心版本的相容性問題，可以利用 KERNEL_VERSION 與 LINUX_VERSION_CODE巧妙避開

```c
<linux/version.h>中所定義的巨集

UTS_RELEASE //展開後為一個描述核心版本的字串

LINUX_VERSION_CODE //展開後為一個代表核心版本的數值

KERNEL_VERSION(major, minor, release) //產生 LINUX_VERSION_CODE的巨集
```
