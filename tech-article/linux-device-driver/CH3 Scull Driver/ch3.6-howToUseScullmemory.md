---
title: CH3.6 How to use scull memory
parent: CH3 Scull Driver
layout: default
nav_order: 6
---

# CH3.6 How to use scull memory

## 本章學習目的 ： 
## 練習在scull driver中去配置&使用memory


- 此範例只for scull驅動程式的記憶體配置策略 (真實驅動程式需要另外的硬體管理技巧)
- scull裝置記憶區可隨資料量長短變化
- scull驅動程式使用的Linux核心中記憶體管理的兩個函式
    - kmalloc()
    - kfree()

```c
#include <linux/slab.h>

void *kmalloc(size_t size, int flags);
void kfree(void *ptr);
```

- scull的基本資料單位為 quanutm
- scull應該要將quantum的容量與quantum set的規模 參數化
    - 在編譯期間，可從scull.h中所定義的SCULL_QUANTUM或SCULL_QSET
    - 在模組裝載期間，可設定scull_quantum或scull_qset
    - run time期間，可透過ioctl()修改設定值

> 將問題參數化，擬定合理的預設值，並同時提供使用者修改的彈性
> 

### scull如何選擇預設值

> 假設scull裝置會被經常寫入大量資料
> 

> 真實的裝置設備會有不同的使用情境，不同的假設
> 

```c

/*structure for storing qantum and qset data*/
struct scull_qset {
	void **data;
	struct scull_qset *next;
};

/*example of how scull implement struct scull_dev and struct scull_qset to storing data*/
int scull_trim(struct scull_dev *dev)
{
		struct scull_qset *next, *dptr;
		int qset = dev->qset;
		int i;
		
		for(dptr = dev->data; dptr; dptr = next) {
				if(dptr->data) {
						for(i = 0; i < qset; i++)
								kfree(dptr->data[i]);
						kfree(dptr->data);
						dptr->data = NULL;
				}
				next = dptr->next;
				kfree(dptr);
		}
		dev->size = 0;
		dev->quantum = scull_quantum;
		dev->qset = scull_qset;
		dev->data = NULL;
		return 0;
}
```
