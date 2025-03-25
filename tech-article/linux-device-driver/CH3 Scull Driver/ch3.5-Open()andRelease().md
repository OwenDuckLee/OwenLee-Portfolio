---
title: CH3.5 open() and release()
parent: CH3 Scull Driver
layout: default
nav_order: 5
---

# CH3.5 open() and release()

## 本章學習目的 ： 
## 實作char driver中的 open() 及 release()


## 大部分驅動程式中的open()需要實作以下功能:
- 檢查裝置特有的錯誤(etc. 硬體方面的問題)
- 若裝置是首次被開啟，應進行初始化程序
- 更新 f_op 指標
- 配置 並 裝填 任何要在 file_p->private_data的資料結構

### open()會先找出目前被開啟的是哪一個裝置

  > 找出含有該cdev結構的sculll_dev結構
  > 

  > 使用<linux/kernel.h>中定義的container_of macro
  > 

  ```c
  #inlcude <linux/kernel.h>
  
  container_of(pointer, container_type, container_field);
  
  /*example of how to find cdef in scull_dev*/
  struct scull_dev *dev;
  dev = container_of(inode->i_cdev, struct scull_dev, cdev);
  filp->private_data = dev;
  
  /*example of simple scull_open()*/
  int scull_open(struct inode *inode, struct file *filp)
  {
  		struct scull_dev *dev;
  		dev = container_of(inode->i_cdev, struct scull_dev, cdev);
  		filp->private_data = dev;
  		
  		if((filp->f_flags & O_ACCMODE) == O_WRONLY){
  				scull_trim(dev);
  		}
  		
  		return 0;
  }
  ```

## release()需要實作以下功能:

- 釋放open()儲存於filp→private_data中的任何東西
- 在最後一次關閉時，將目標裝置關機
