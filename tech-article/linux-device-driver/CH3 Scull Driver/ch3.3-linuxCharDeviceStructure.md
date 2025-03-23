---
title: CH3.3 Linux char device structure
parent: CH3 Scull Driver
layout: default
nav_order: 3
---

# CH3.3 Linux char device structure

## 本章學習目的 ： 
## 認識Kernel提供給 char device的 API (驅動程式會用到的資料結構)

## 驅動程式大部分的基本操作，都會用到以下三種核心重要的資料結構

- file_operations
- file
- inode

### file_operations 結構

```c
#include <linux/fs.h>

struct file_operations {
	struct module *owenr;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*aio_read) (struct kiocb *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	ssize_t (*aio_write) (struct kiocb *, const char __user *, size_t, loff_t *);
	int (*readdir) (struct file *, void *, filldir_t);
	unsigned int (*poll) (struct file *, struct poll_table_struct *);
	
	int (*ioctl) (struct inode *, struct file *, unsigned int, unsigned long);
	int (*mmap) (struct file *, struct vm_area_struct *);
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *);
	int (*release) (struct inode *, struct file *);
	int (*fsync) (struct file *, struct dentry *, int);
	int (*aio_fsync) (struct kiocb *, int);
	.
	.
	.
	.
};
```

### Scull device driver → file_operations example

```c
struct file_operations scull_ops = {
		.owner = THIS_MODULE, 
		.llseek = scull_llseek,
		.read = scull_read,
		.write = scull_write, 
		.ioctl = scull_ioctl, 
		.open = scull_open, 
		.release = scull_release,
};
		
```

### file 結構

```c
#include <linux/fs.h>
#include <linux/fcntl.h> //where defines f_flags

struct file {
		mode_t f_mode; //file access mode [FMODE_READ | FMODE_WRITE]
		loff_t f_pos; //current read write position
		unsigned int f_flags; //driver usually check O_NONBLOCK [O_RDONLY | O_NONBLOCK | O_SYNC]
		struct file_operations *f_op;
		void *private_data;
		struct dentry *f_dentry;
};
```

### inode結構

```c
/*核心內部使用inode代表檔案，代表已開啟的檔案的FD*/
/*同一個檔案可被開啟多次，在此情況下，核心內部會有多個代表FD的file結構，但它們都只會指到同一個inode結構*/

/*inode結構含有大量檔案資訊，通常驅動程式只對其中兩個有興趣*/

dev_t i_rdev; //含有實際的裝置編號
struct cdev *i_cdev; //對於代表字元裝置的inode, 此欄位是一個指向核心字元裝置結構的指標
```
