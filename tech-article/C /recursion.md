---
title: Recursion
parent: C
layout: default
nav_order: 2
---

# Case Study

## Hanoi

- 核心concept:
    
    使用recursion 將 大問題拆解成小問題
    
    e.g 要將三個disk由 來源塔移動到目標塔，需先將 n-1個 移動到 輔助塔
    
- 主要邏輯 :
    1. 將n-1個 disk 從來源塔移動到輔助塔
    2. 將第n個disk從來源塔移動到目標塔
    3. 將n-1個disk由輔助塔移動到目標塔

```c
#include <stdio.h>

/*function to perform a single mode*/
void move(int disk, char fromRod, char toRod) {
	printf("Move disk %d from rod:%c to rod:%c\n", disk, fromRod, toRod);
}

/*function to solve Tower of Hanoi recursively*/
void towerOfHanoi(int n, char fromRod, char toRod, char auxRod) {
	//base case: no disks to move
	if(n == 0)
		return;
	
	towerOfHanoi(n-1, fromRod, auxRod, toRod);
	move(n, fromRod, toRod);
	towerOfHaoni(n-1, auxRod, toRod, fromRod);
}

int main(void) {
	int n;
	printf("Enter the number of disks: ");
	scanf("%d", &n);
	printf("Steps to solve Tower of Hanoi with %d disks: \n", n);
	towerOfHanoi(n, 'A', 'C', 'B');

	return 0;
}	
```

## 字串反轉

- 要求使用C語言實作 char *reverse(char *s) 反轉NULL結尾的字串
    
    限定 in-place與遞迴方式
    
- 先思考時作swap的技巧
- 思考可否避免使用區域變數，改用bitwise operator作法
    
    > swap 使用數值運算，當遇到一個很大的整數與負數相減，會產生overflow
    > 
    
    > bitwise operator 可以避免 integer overflow 且 不限定位元數
    > 

```c
static inline void swap(char *a, char *b)
{
	*a = *a ^ *b;
	*b = *a ^ *b;
	*a = *a ^ *b;
}

char * reverse(char *s)
{
	if((*s == '\0') || (*(s + 1) == '\0'))
			return NULL;
			
	reverse(s + 1);
	swap(s, (s + 1));
	if(*(s + 2) != '\0')
			reverse(s + 2);
			
	reverse(s + 1);
}

/*disadvantage
* 時間複雜度 T(n) = O(n平方)
*/
```

## 字串反轉 - 思考時間複雜度 改進思路

> 計算字串長度的時候，即可先交換字元
> 

```c
static inline void swap(char *a, char *b)
{
	*a = *a ^ *b;
	*b = *a ^ *b;
	*a = *a ^ *b;
}

int reverse_core(char *head, int index)
{
	if(head[index] != '\0'){
			int end = reverse_core(head, index + 1);
			if(index > end / 2)
					swap(head + index, head + end - index);
			return end;		
	}
	return index - 1;
}

char reverse(char *s)
{
	reverse_core(s, 0);
	return s;
}

/*時間複雜度為 T(n) = O(n)*/

```

## 建立目錄

- 使用程式碼建立一個模擬 mkdir功能
- -p parameter can create parent-chid directories
- use recursion skill to implement

```c
#include <dirent.h>
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

int mkdir_r(const char *path, int level)
{
		int ret = -1;
		if(!strlen(path))
					goto leave;
					
		const char *c = path;
		int cur_level = 0;
		
		while(cur_level <= level){
				/*seak for separator*/
				for(; *c && *c != '\\' && *c != '/'; c++)
							;
				if(*c)
						c++;
				cur_level++;
		}
		
		const size_t sz = c - path + 1;
		char *dir = malloc(sz);
		if(!dir)
				goto leave;
				
		memcpy(dir, path, sz - 1);
		dir[sz - 1] = '\0';
		
		DIR *d = opendir(dir);
		if(!d && mkdir(dir, S_IRWXU | S_IRWXG | S_IROTH | S_IXOTH))){
					perror("mkdir");
					goto leave;
		}
		
		if(*c)
				mkdir_r(path, cur_level);
				
		ret = 0;
		
		leave:
				free(dir);
				if(d)
					closedir(d);
				return ret;
}

int main()
{
		mkdir_r("aaa/bbb/ccc", 0);
		return 0;
}
```

## 實作模擬find的程式

- 使用 opendir(3) 與 readdir(3) syscall 函式
- 用recursion方式寫出類似find的程式
- find程式的功能：列出包含當前目錄和其所有子目錄之下的檔案名稱

```c
#include <sys/types.h>
#include <dirent.h>

void list_dir(const char *dir_name)
{
		DIR *d = opendir(dir_name);
		if(!d) return; //fail to open directory
		
		while(1){
				struct dirent *entry = readdir(d);
				if(!entry) break;
				
				const char *d_name = entry->d_name;
				printf("%s/%s\n", dir_name, d_name);
				
				if((entry->d_type & DT_DIR) && strcmp(d_name, "..")
						&& strcmp(d_nname, ".")) {
						char path[PATH_MA];
						int path_length = snprintf(path, PATH_MAX, 
																		"%s/%s", dir_name, d_name);
						printf("%s\n", path);
						if(path_length >= PATH_MAX) return;
						
						/*recursively call "list_dir" with new path*/
						list_dir(path);
				}
		}
		if(closedir(d)) return;
}		
														
```

# Functional Programming in C

## Functional Programming for Concurrency environment
### reference: https://hackmd.io/@jserv/H10MXXoT?type=view

### Functional Programming C

> Functional Programming 是種開發典範(programming apradigm)，
> 不是design patter 也不是 framework
> 是種以數學函數為中心的"思考方式” 與 “程式風格”
> 
- 運算用的function 大多只包含 條件式及遞迴呼叫
- higher-order functions兩個特點
    - function 可以當作參數傳入funciton 之中
    - function 可以回傳 function
- no implicit Side Effect (沒有隱含的副作用)
- lazy evaluation

- FP 中常見的三種操作
    - map() : 配合給定的行為，回傳一個陣列或 linked list
    - filter() : 用來檢索 或 篩選
    - reduce() : 將素材混合在一起，得到最終結果

## Case: reverse elements in linked list

- 整體函式執行順序
    
    ```c
    main -->
    		make_list -->
    				reverse_toarray -->
    						reverse -->
    								list2array -->
    										void_map_array	
    ```
    
- 建立 linked list 的部分
    - 由陣列的尾端開始建立起
    - 建立linked list過程中，lst會將下一個結構的next儲存起來
    - lst將記憶體位址 和 數值 複製到下一個結構中

```c
#include <stdio.h>
#include <stdlib.h>

/*integer linked list type*/
typedef struct __int_list const *const int_list_t;
static int_list_t const Nil = NULL; //empty list

/*singly linked list element*/
typedef struct __int_list {
	int_list_t next;
	int32_t const val;
} node_t;

/*a pointer to where we store the result of the computation*/
typedef void *const CPS_Result;

/*prototypes for the continuation - call back functions*/
typedef void (*MakeListCallback)(int_list_t list, CPS_Result res);
void make_list(uint32_t const arr_size,
								int32_t const array[], 
								int_list_t lst,
								MakeListCallback const cb,
								CPS_Result res);
						
typedef void (*ReversedListCallback)(int_list_t list, CPS_Result res);
void reverse(int_list_t list,
							int_list_t rlist,
							ReversedCallback const cb,
							CPS_Result res);
							
typedef void (*VoidMappable)(int32_t const val);
void void_map_array(VoidMapple const cb,
										uint32_t const size,
										int32_t const *const arr);
										
void list2array(int_list_t list, CPS_Result res);

/*reverse a list and store it in an array*/
void reverse_toarray(int_list_t list, CPS_Result res) {
	reverse(list, Nil, list2array, res);
}

static void print_val(int32_t const val) { printf("%d ", val); }

#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof(arr[0]))

int main(int argc, char *argv[]) {
	int32_t arr[] = {99, 7, 5};
	uint32_t const arr_size = ARRAY_SIZE(arr);
	
	int32_t res[arr_size];
	
	/*call make_list and pass reverse_toarray as the "continuation"*/
	make_list(arr_size, arr, Nil, reverse_toarray, res);
	
	/*print out the reversed array*/
	void_map_array(print_val, arr_size, res);
	printf("\n");
	return 0;
}

/*construct a linked list from an array*/
void make_list(uint32_t const arr_size,
								int32_t const arr[], 
								int_list_t lst,
								MakeListCallback const cb,
								CPS_Result res) {
	if(!arr_size) {
		cb(lst, res);
		return;
	}
	
	make_list(arr_size - 1, arr, 
						&(node_t){.val = arr[arr_size - 1], .next = lst}, cb, res);
}								

/*transform a linked list into an array*/
void list2array(int_list_t list, CPS_Result res) {
	if(Nil == list) return;
	int32_t *array = res;
	array[0] = list->val;
	list2array(list->next, array + 1);
}

void reverse(int_list_t list,
							int_list_t rlist, //reversed list
							ReversedListCallback const cb,
							CPS_Result res) {
	if(Nil == list) {
		cb(rlist, res);
		return;
	}
	reverse(list->next, &(node_t){.val = list->val, .next = rlist}, cb, res);
}

void void_map_array(VoidMappable const cb,
										uint32_t const size,
										int32_t const *const arr) {
	if(!size) return;
	cb(arr[0]);
	void_map_array(cb, size -1, arr + 1);
}
			
```

## Case: merge sort elements in linked list

- 使用merge sort減少程式碼中的side effect
- 將每個sub function 變成 pure function ( each function only used region variables)
- non-intrusive linked list —> linked list的結構與資料物件本身是分離的

```c
#include <stdiod.h>
#include <stdint.h>
#include <stddef.h>
#include <stdlib.h>

#define container_of(ptr, type, member) \
		((type *) ((char *) (ptr) - offsetof(type, member)))

#define list_entry(node, type, member) container_of(node, type, member)

typedef struct __list * list_t;
typedef struct __list {
	list_t next;
} node_t;

typedef struct __ele {
	int32_t const val;
	list_t const list;
} ele_t;

static const ele_t Nil = {.val = 0, .list = &(node_t){.next = NULL}};

typedef void *const CPS_Result;
typedef void (*MakeListCallBack)(ele_t *e, CPS_Result res);
void make_list(uint32_t const arr_size, 
								int32_t const array[],
								ele_t *e, 
								MakeListCallBack const cb,
								CPS_Result res);
							
void mergesort_toarray(ele_t *e, CPS_Result res);

/*Merge Sort*/
void mergesort(ele_t **source);
void partition(ele_t *head, ele_t **front, ele_t **back);
ele_t *mergeLists(ele_t *a, ele_t *b);

typedef void(*VoidMappable)(int32_t const val);
void void_map_array(VoidMappable const cb,
										uint32_t const size,
										int32_t const *const arr);
										
void list2array(ele_t *e, CPS_Result res);

static void print_val(int32_t const val) {
	printf("%d ", val);
}

#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof(arr[0]))

int main(int argc, char *argv[]) {
	int32_t arr[] = {99, 95, 90, 85, 80, 20, 75, 15, 10, 5};
	uint32_t const arr_size = ARRAY_SIZE(arr);
	
	int32_t res[arr_size];
	make_list(arr_size, arr, (ele_t *)(&Nil), mergesort_toarray, res);
	
	void_map_array(print_val, arr_size, res);
	printf("\n");
	return 0;
}

void make_list(uint32_t const arr_size, 
								int32_t const array[],
								ele_t *e, 
								MakeListCallBack const cb,
								CPS_Result res)
{
	if(!arr_size) {
		cb(e, res);
		return;
	}
	
	make_list(arr_size - 1, arr,
						&(ele_t){.val = arr[arr_size - 1],
										 .list = &(node_t){.next = (list_t)(&(e->list))}},
										 cb, res);
}

void list2array(ele_t *e, CPS_Result res) {
	if(!e->list->next) return;
	
	int32_t *array = res;
	array[0] = e->val;
	list2array(list_entry(e->list->next, ele_t, list), array + 1);
} 

void mergesort_toarray(ele_t *e, CPS_Result res) {
	if(e->list->next)
		mergesort(&e);
	list2array(e, res);
}

void void_map_array(VoidMappable const cb,
										uint32_t const size, 
										int32_t const *const arr)
{
	if(!size) return;
	cb(arr[0]);
	void_map_array(cb, size - 1, arr + 1);
}

/*Algorithm for mergesort*/
void mergesort(ele_t **source) {
	ele_t *head = *source;
	ele_t *a = NULL, *b = NULL;
	
	if(!head || head->list->next == (list_t)(&(Nil.list))) return;

	partition(head, &a, &b); //partition source list then seperately put into front-list and back-list
	
	mergesort(&a);
	mergesort(&b);
	
	*source = mergeLists(a, b);
}

void partition(ele_t *head, ele_t **front, ele_t **back) {
	if(!head || head->list->next == (list_t)(&(Nil.list))) {
		*front = head;
		*back = NULL;
	} else {
		ele_t *slow = head;
		ele_t *fast = list_entry(head->list->next, ele_t, list);
		
		while(fast->list->next) {
			fast = list_entry(fast->list-next, ele_t, list);
			
			if(fast->list->next) {
				slow = list_entry(slow->list->next, ele_t, list);
				fast = list_entry(fast->list-next, ele_t, list);
			}
		}
		
		*front = head;
		*back = list_entry(slow->list->next, ele_t, list);
		slow->list->next = (list_t)(&(Nil.list));
	}
}
	
ele_t *mergeLists(ele_t *a, ele_t *b) {
	ele_t *mergedList = NULL;
	
	if(!a->list->next) return b;
	if(!b->list->next) return a;
	
	if(a->val <= b->val) {
		mergedList = a;
		ele_t *tmp = mergeLists(list_entry(a->list->next, ele_t, list), b);
		mergedList->list->next = (list_t)(&(tmp->list));
	} else {
		mergedList = b;
		ele_t *tmp = mergeLists(a, lsit_entry(b->list->next, ele_t, list));
		mergedList->list->next = (list_t)(&(tmp->list));
	}
	return mergedList;
}
```
