---
title: CH3.4 Register Char Device
parent: CH3 Scull Driver
layout: default
nav_order: 4
---

# CH3.4 Register Char Device

## 本章學習目的 ： 
## 認識struct cdev 及 如何在kernel中註冊struct cdev

- 核心內部使用struct cdev結構代表字元裝置
- 要讓核心能夠調用驅動程式，需要註冊一或多個struct cdev

### Example code
```c
#include <linux/cdev.h>

/*run time allocate cdev and assign file operations*/
struct cdev *my_cdev = cdev_alloc();
my_cdev->ops = &my_fops;

/*when your driver embedded cdev into designed special structure
* you will need to use cdev_init() for initialize cdev
*/
void cdev_init(struct cdev *dev, struct file_operations *fops);

/*After initialization, you need to add owner and fops*/

/*Final step, use cdev_add() to add cdev into kernel*/
int cdev_add(struct cdev *dev, dev_t num, unsigned int count);

/*scull driver use scull_dev sturcture represent each type of device*/
struct scull_dev {
	struct scull_qset *data;
	int quantum;
	int qset;
	unsigned long size;
	unsigned int access_key;
	struct semaphore sem;
	struct cdev cdev;
};

/*Example of how to initialize cdev structure for scull driver*/
static void scull_setup_cdev(struct scull_dev *dev, int index)
{
		int err;
		int devno = MKDEV(scull_major, scull_minor + index);
		
		cdev_init(&dev->cdev, &scull_fops);
		dev->cdev.owner = THIS_MODULE;
		dev->cdev.ops = &scull_fops;
		err = cdev_add(&dev->cdev, devno, 1);
		
		if(err)
				printk(KERNEL_NOTICE "Error %d adding scull%d", err, index);
}
```
