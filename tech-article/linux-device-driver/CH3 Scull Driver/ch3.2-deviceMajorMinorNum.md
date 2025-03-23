---
title: CH3.2 Device Major-num and Minor-num
parent: CH3 Scull Driver
layout: default
nav_order: 2
---

# Device Major-num and Minor-num

# 本章學習目的 ： 理解如何 配置,存取,釋放 裝置編號 

## 字元裝置的存取，是透過裝置在檔案系統上的名稱

- special file → 在 /dev 目錄下
- device file
- node

> major number 代表裝置所配合驅動程式(驅動程式編號)
> 

> minor number 代表你的驅動程式所實作的裝置(裝置編號)
> 

- 主編號與次編號 兩者合稱 “裝置編號”
- 在核心中以 dev_t 型別 表示裝置編號 (定義於 <linux/types.h>)
- dev_t 為一個 unsigned 32-bit
    - 12bits 為主編號
    - 20bits 留給次編號
- 驅動程式中，應該使用<linux/kdev_t.h> 提供的一對巨集 來設定裝置編號
    
    ```c
    #include <linux/types.h>
    #include <linux/kdev_t.h>
    
    /*dev_t is defined in <linux/types.h>*/
    
    /*這兩個巨集可以從一個裝置編號中，抽離出主編號與次編號
    * 展開後的型別依然是dev_t
    */
    MAJOR(dev_t dev);
    MINOR(dev_t dev);
    
    /*將主編號與次編號 合併為一個裝置編號dev_t*/
    MKDEV(int major, int minor);
    ```
    

## 裝置編號的配置&釋放

> 當驅動程式要設定字元裝置時，第一件事就是要先取得一個或多個裝置編號
> 

```c
/*處理這件事 需要使用 register_char_dev_region()函式*/

#include <linux/fs.h>

int register_char_dev_region(dev_t first, 
														unsigned int count, char *name);
														
/*parameter:
* first --> 想配置的裝置編號的範圍之起點
* count --> 想申請的連裝置編號的總數
* name  --> 獲得此編號範圍的裝置之名稱，名稱會出現在 /proc/devices & sysfs
*/

/*當不需要裝置編號時，記得要釋放它們，通常會在模組的clean function中*/
void unregister_chardev_region(dev_t first, unsigned int count);
```

## 如何動態配置主編號 ( **驅動程式建議使用此種方式** )

```c
#include <linux/fs.h>

int alloc_chardev_region(dev_t *dev, unsigned int firstminor, unsigned int count, cahr *name);

/*當不需要裝置編號時，記得要釋放它們，通常會在模組的clean function中*/
void unregister_chardev_region(dev_t first, unsigned int count);
```

- 在核心原始程式碼中 Documentation/devices.txt檔案中
有紀錄該版本核心預定的"裝置-主編號”對照表
- 動態配置也有缺點，無法事先建立好 /dev/裝置節點
- 可自己建立一個shell script “scull_load”

```bash
## 讀取裝置編號->建立裝置節點->載入模組
## 裝置列表的來源 可從 /proc/devices or sysfs

#!bin/sh
module="scull" 
device="scull"
mode="664"

#use insmod to laod target module
/sbin/insmod ./$module.ko $* || exit 1

#remove previous loaded device nodes
rm -f /dev/${device}[0-3]

#/proc/devices $1 is major number, $2 is device name
major=$(awk "\$2= =\"$module\" {print \$1}" /proc/devices)

mknod /dev/$(device)0 c $major 0
mknod /dev/$(device)1 c $major 1
mknod /dev/$(device)2 c $major 2
mknod /dev/$(device)3 c $major 3

#setting node owener group and permision
group="staff"
grep -q '^staff:' /etc/group || group="wheel"
chgrp $group /dev/${device}[0-3]
chmod $mode /dev/${device}[0-3]
```
