---
title: CH4.3 Inquery Debug
parent: CH4 How to Debug
layout: default
nav_order: 2
---

- 大量使用printk() 會導致系統效率降低
    
    可以考慮 修改 /etc/proc/syslogd.conf 組態檔，將日誌檔名之前加一個減號
    
    它要求syslogd不必每次收到新訊息就立刻寫入磁碟。細節可參考 man syslog.conf(5)
    

- 對Driver Designer來說，以下幾種方式可以取得系統狀態資訊
    1. ps
    2. netstat
    3. vmstat
    4. /proc filesystem 中建立一個檔案
    5. 使用ioctl作業方法
    6. 將狀態資訊 output到sysfs

## 使用 /proc 檔案系統

> /proc 是 linux kernel 模擬出來的軟體檔案系統
是kernel 對於外界的資訊窗口
> 

### 驅動程式 實作一個可應付 read() syscall 的 read-only /proc檔

```c
#include <linux/proc_fs.h>

/*Driver need to implement a read_proc function and register it to kernel*/

/*the prototype of read_proc function*/
int (*read_proc)(char *page, char **start, off_t offset,
									int count, int *eof, void *data)									
```

```c
#include <linux/proc_fs.h>

int scull_read_procmem(char *buf, char **start, off_t offset,
												int count, int *eof, void *data)
{
	int i, j;
	int len = 0;
	int limit = count - 80; //don't print more than this
	
	for(i = 0; i < scull_nr_devs && len <= limit; i++) {
		struct scull_dev *d = &scull_devices[i];
		struct scull_qset *qs = d->data;
		if(down_interruptible(&d->sem))
			return -ERESTARTSYS;
		
		len += sprintf(buf + len, "\nDevice %i: qset %i, q %i, sz %li\n",
										i, d->qset, d->quantum, d->size);
		for(; qs && len <= limit; qs = qs->next) { /*scan the list*/
			len += sprintf(buf + len, " item at %p, qset at %p\n", qs, qs->data);
			if(qs->data && !qs->next) { /*dump only the last item*/
				for(j = 0; j < d->qset; j++) {
					if(qs->data[j])
						len += sprintf(buf + len, "   %4i: %8p\n", j, qs->data[j]);
				}
			}
		}
		up(&scull_devices[i].sem);
	}
	
	*eof = 1;
	return len;
}
```

### 註冊 /proc

- 在定義好read_proc作業函式後，還必須要為它在 /proc 下設置一個entry
- 驅動程式只要呼叫 create_proc_read_entry() 函式就可以完成這個動作
    
    ```c
    struct proc_dir_entry *create_proc_read_entry(const char *name,
    							mode_t mode, struct proc_dir_entry *base,
    							read_proc_t *read_proc, void *data);							
    /*
    * name ---> 要建立檔案的名稱
    * mode ---> 檔案的保護模式
    * read_proc ---> ptr to read_proc function
    * data ---> 核心不處理data，會傳遞給read_proc
    * base ---> prt 指向上層目錄結構，NULL表示直接在/proc目錄下建檔
    */
    ```
    
- Driver module被卸載前，需要移除相關的/proc entry
    
    ```c
    remove_proc_entry("scullmem", NULL /*上層目錄*/);
    ```
    
- 以read_proc介面實作/proc檔 會有以下許多問題
    1. 重複註冊相同名稱的問題，驅動程式不小心以相同名稱重複註冊
        - 當user-space access時，kernel無法區別是哪一個
        - 驅動程式呼叫 remove_proc_entry()，kernel無法區別是哪一個
    2. 無法確保每次都能成功移除/proc entry
        - 就算Driver中的cleanup() 在模組卸載前呼叫remove_proc_entry()
            
            也無法保證當下是否有process正在使用/proc entry
            
        - 沒成功移除 /proc entry ，可能導致read_proc意外觸發，導致kernel崩壞
- 現行已不建議Device Driver實作 /proc ，僅供debug用

### seq_file 介面

- 提供一組簡單的函式，可用來實作大型的核心虛擬檔
- 要使用seq_file，必須要先建立一個 iterator 物件，當成序列的指位器
- 在scull driver中 以seq_file 介面去建立一個 /proc檔
- 建立iterator物件的operation functions
    - start
    - next
    - stop
    - show
    
    ```c
    /*operation functions prototype*/
    void *start(struct seq_file *sfile, loff_t *pos);
    void *next(struct seq_file *sfile, void *v, loff_t *pos);
    void stop(struct seq_file *sfile, void *v);
    int show(struct seq_file *sfile, void *v);
    ```
    
    ```c
    #include <linux/seq_file.h>
    
    static void *scull_seq_start(struct seq_file *s, loff_t *pos) {
    	if(*pos >= scull_nr_devs)
    		return NULL; //no more to read
    	return scull_devices + *pos;
    }
    
    static void *scull_seq_next(sturct seq_file *s, void *v, loff_t *pos) {
    	(*pos)++;
    	if(*pos >= scull_nr_devs)
    		return NULL;
    	return scull_devices + *pos;
    }
    
    /*as for scull driver, there is no clean work for iterator stop operation function*/
    
    /*kernel will call show function to output data to user-space* during start and stop*/
    static int scull_seq_show(struct seq_file *s, void *v) {
    	struct scull_dev *dev = (struct scull_dev *) v;
    	struct scull_qset *d;
    	int i;
    	
    	if(down_interruptible(&dev->sem))
    		return -ERESTARTSYS;
    	
    	seq_printf(s, "\nDevice %i: qset %i, q %i, sz %li\n", 
    					(int)(dev - scull_devices), dev->qset, dev->quantum, dev->size);
    	
    	for(d = dev->data; d; d = d->next) {
    		seq_printf(s, " item at %p, qset at %p\n", d, d->data);
    		if(d->data && !d->next) {
    			for(i = 0; i < dev->qset; i++) {
    				if(d->data[i]) {
    					seq_printf(s, "    %4i: %8p\n", i, d->data[i]);
    				}
    			}
    		}
    	}
    	up(&dev->sem);
    	return 0;
    }
    			
    
    ```
    
- scull 驅動需要將上面製作好的作業方法包裝起來
    
    並且連結到/proc下的某個檔案
    
    ```c
    /*build and configure a seq_oeprations structure*/
    static struct seq_operations scull_seq_ops = {
    	.start = scull_seq_start, 
    	.next = scull_seq_next,
    	.stop = scull_seq_stop,
    	.show = scull_seq_show
    };
    ```
    
- 還需要再建立一個核心認識的file object
    
    ```c
    /*使用seq_file，建議使用較低的層級去連結/proc
    * 也就是建立一個 file_operations structure
    * 並提供可以讓核心 讀取 定位 這個file object的作業方法
    */
    
    /*construct a open operation function
    * which connect file object to seq_oeperations structure
    */
    
    static int scull_proc_open(sturct inode *inode, struct file *file) {
    	return seq_open(file, &scull_seq_ops);
    }
    
    /*build and configure file_oeprations structure*/
    static struct file_operations scull_proc_ops = {
    	.owner = THIS_MODULE,
    	.open = scull_proc_open,
    	.read = seq_read,
    	.llseek = seq_lseek,
    	.release = seq_release
    };
    ```
    

## 使用 iotctl 作業方法
