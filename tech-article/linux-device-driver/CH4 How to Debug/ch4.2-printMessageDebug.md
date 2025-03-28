---
title: CH4.2 Print message debug
parent: CH4 How to Debug
layout: default
nav_order: 1
---


## printk() function

> 驅動程式研發初期的偵錯好工具
> 

### How to set console loglevel

- run program which call syscall sys_syslog() e.g. setlevel.c
- modify the console_level current value in /proc/sys/kernel/printk
    
    ```c
    $ sudo echo 8 > /proc/sys/kernel/printk
    ```
    

### 製作printk()訊息開關

- PDEBUG巨集範例
    
    ```c
    #undef PDEBUG
    #ifdef SCULL_DEBUG
    	#ifdef __KERNEL__
    		/*for kernel-space debug*/
    		#define PDEBUG(fmt, args...) printk( KERN_DEBUG "scull: " fmt, ## args)
    	#else
    		/*for user-space debug*/
    		#define PDEBUG(fmt, args...) fprintf(stderr, fmt, ## args)
    	#endif
    #else
    	#define PDEBUG(fmt, args...) /*empty macro*/
    #endif
    
    #undef PDEBUGG
    #define PDEBUGG(fmt, args...) /*for disable indicated debug message*/
    ```
    
- 進一步簡化控制程序，Makefile中做設定
    
    ```makefile
    ##Makefile example##
    
    #if want to disable all debug message, mark below
    DEBUG = y
    
    #decide if debug flag will add to CFLAGS
    ifeq ($(DEBUG), y)
    	DEBFLAGS = -O -g -DSCULL_DEBUG # "-O" is necessary for expand inline macro
    else
    	DEBFLAGS = -O2
    endif
    
    CFLAGS += $(DEBFLAGS)
    
    ```
    

### 節制訊息產生速率

- kernel提供了一個有調節作用的工具函式
    
    ```c
    int printk_ratelimit(void);
    
    /*printk_ratelimit() sue example*/
    if(printk_ratelimit())
    	printk(KERN_NOTICE "The printer is still on fire\n");
    ```
    
- 可以在user-space中透過/proc檔案系統，調整printk_ratelimit()的工作參數
    - /proc/sys/kernel/printk_ratelimit
        
        → 調整"間隔秒數”，訊息量超過零界值後須等待多少秒，才可恢復訊息傳送
        
    - /proc/sys/kernel/printk_ratelimit_burst
        
        → 控制“零界值”，計算的單位為列數
        

### 如何印出裝置編號

> 驅動程式印出訊息時，需要顯示裝置編號，識別是哪個硬體裝置的訊息
> 
- kernel提供一組macro ， 定義於 <linux/kdev_t.h>
    
    ```c
    #include <linux/kdev_t.h>
    
    int print_dev_t(char *buffer, dev_t dev);
    char *format_dev_t(char *buffer, dev_t dev);
    ```
