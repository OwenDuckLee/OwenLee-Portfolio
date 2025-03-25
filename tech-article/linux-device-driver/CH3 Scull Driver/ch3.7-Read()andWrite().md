---
title: CH3.7 read() and write()
parent: CH3 Scull Driver
layout: default
nav_order: 7
---

# CH3.7 read() and write()

## 本章學習目的 ： 
## 實作 char device driver中的 read() 及 write()

- read() 跟 write() 的工作類似，都是在user-space與kernel-space之間傳遞資料
- 驅動程式必須要能夠存取位於user-space的記憶區
- kernel-space的程式要可安全存取位於user-space的資料

### Kernel 中提供的 read() and write() prototype
```c
ssize_t read(struct file *filp, char __user *buff, size_t count, loff_t *f_pos);
ssize_t write(struct file *filp, const char __user *buff, size_t count, loff_t *f_pos);

//f_pos is a pointer point to an long offset type object
//this object represent the position where user is accesing in the file
```

### 核心提供一組函式，確保驅動程式能夠安全的存取，傳輸資料
```c
//scull driver的read需要可將整段資料寫到user-space的能力
//scull driver的write需要可從user-space讀取一整段資料的能力

#include <asm/uaccess.h>

/*可傳輸任何型別，任意規模的陣列*/
unsigned long copy_to_user(void __user *to, const void *from, unsigned long count);
unsigned long copy_from_user(void *to, const void __user *from, unsigned long count);

//皆會檢查user-space傳入的指標是否有效，若無效，則不會有傳輸動作
//如果是傳輸資料期間才遇到無效指標位址，則只有部分資料會完成傳輸
//以上兩種情況，return value 都是"尚未完成傳輸的資料量"

//return value 0 表示成功
```

- 驅動程式裡任何會存取user-space的函式，都必須符合reentrant的條件
- 驅動程式裡任何會存取user-space的函式，必須要能夠與其他函式concurrently執行


## Scull 字元驅動程式 read() operation
```c
ssize_t scull_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
{
		struct scull_dev *dev = filp->private_data;
		struct scull_qset *dptr; //the first listitem
		
		int quantum = dev->quantum, qset = dev->qset;
		int itemsize = quantum * qset; //how many bytes in the listitem
		int item, s_pos, q_pos, rest;
		ssize_t retval = 0;
		
		/*error handling*/
		if(down_interruptible(&dev->sem))
				return -ERESTARTSYS;
				
		if(*f_pos >= dev->size)
				goto out;
		
		if(*f_pos + count > dev->size)
				count = dev->size - *f_pos;
				
		/*find listitem, qset index, and offset in the quantum*/
		item = (long) *f_pos / itemsize;
		rest = (long) *f_pos % itemsize;
		s_pos = rest / quantum;
		q_pos = rest % quantum;
		
		/*follow the list up to the right position*/	
		dptr = scull_follow(dev, item);
		if(dptr == NULL || !dptr->data || !dptr->data[s_pos])
				goto out;
				
		/*read only up to the end of this quantum*/
		if(count > quantum - q_pos)
				count = quantum - q_pos;
		
		if(copy_to_user(buf, dptr->data[s_pos] + q_pos, count)){
				retval = -EFAULT;
				goto out;
		}
		
		*f_pos += count;
		retval = count;
		
		out:
			up(&dev->sem);
			return retval;
}
```

## Scull 字元驅動程式 write() operation

> !!Noted!! 真實世界很少採取部分寫入的作法
> 

> 太多工程師設計的程式都不接受部分傳輸，如果write()沒成功，就直接認定為失敗…
> 

```c
/*解除 scull_write() 一次最多只能寫滿一單位quantum的限制*/
ssize_t sull_write(struct file *filp, const char __user *buf, size_t count, loff_t *f_pos)
{
		struct scull_dev *dev = filp->private_data;
		struct scull_qset *dptr;
		int quantum = dev->quantum, qset = dev->qset;
		int itemsize = quantum * qset;
		int item, s_pos, q_pos, rest;
		ssize_t retval = -ENOMEM;
		
		if(down_interruptible(&dev->sem))
					return -ERESTARTSYS;
					
		/*find listitem, qset index and offset in the quantum*/
		item = (long)*f_pos / itemsize;
		rest = (long)*f_pos % itemsize;
		s_pos = rest / quantum;
		q_pos = rest % quantum;
		
		/*follow the list up to the right position*/
		dptr = scull_follow(dev, item);
		if(dptr == NULL)
				goto out;
				
		if(!dptr->data) {
				dptr->data = kmalloc(qset * sizeof(char *), GFP_KERNEL);
				if(!dptr->data)
						goto out;
				memset(dptr->data, 0, qset * sizeof(char *));
		}
		
		if(!dptr->data[s_pos]) {
				dptr->data[s_pos] = kmalloc(quantum, GFP_KERNEL);
				if(!dptr->data[s_pos])
						goto out;
		}
						
		/*write only up to the end of this quantum*/
		if(count > quantum - q_pos)
				count = quantum - q_pos;
				
		if(copy_from_user(dptr->data[s_pos] + q_pos, buf, count)) {
					retval = -EFAULT;
					goto out;
		}
		*f_pos += count;
		retval = count;
		
		/*udpate the size*/
		if(dev->size < *f_pos)
				dev->size = *f_pos;
				
		out:
				up(&dev->sem);
				return retval;
}
```
