---
title: Bitwise Operation
parent: C
layout: default
nav_order: 1
---

# Refernce Article
Reference article: https://hackmd.io/@sysprog/c-bitwise

# Bitwise 方式

- Logical shift : 左側補上0
- Arithmetic shift : 補上號數 (sign bit) 也就是最高有效位元的值在左側

## 注意位移運算的兩種未定義狀況

- 左移超過變數長度，其結果未定義
    
    ```c
    int i=0xFFFFFFFF;
    i = i << 32; // 此結果未定義
    ```
    
- 右移一個負數時，可能是邏輯位移或是算術位移，C 語言標準未定義
    
    在 C99 規格書 6.5.7 Bitwise shift operators 第 5 節指出
    
    > The result of E1 >> E2 is E1 right-shifted E2 bit positions. 
    If E1 has a signed type and a negative value, the resulting value is implementation-defined.
    > 
    
- 當右移一個負數時，有可能變成正數or負數，主要會依據編譯器如何實作的
- 編譯器可以有選項可改變此語意，gcc的實作就是使用arithmetic shift (ex. 也就是右移一個負數時 會補上sign bit在左側 得到一個負數)

> arithmetic shift的應用
若要判斷一個int type 的變數n是否為正數，可以使用 n >> 31 等同於 n ≥ 0 ? 0 : -1
> 

```c
int n;

/*how to identify if variable n is positive*/
(n >> 31); //option 1
n >= 0 ? 0 : -1; //option 2
```

## !需特別注意! 無號數 與 有號數 在C語言混合在單一表示時，有號數會被轉換為無號數

須注意混用時的實際運算狀況

```c
/*int 與 unsigned int 運算 產生無窮迴圈的案例*/
int n = 10; 
for (int i = n - 1 ; i - sizeof(char) >= 0; i--)
    printf("i: 0x%x\n",i);
    
//sizeof() will return unsigned int
//i will be turned to unsigned
//when unsigned 0 minus 1 will be 0xFFFFFFFF
//cause infinite loop
```

> 若 n 是 32-bit 整數，那麼 abs(n) 等同於 ((n >> 31) ^ n) - (n >> 31)
> 
- 當 n 是正數時:
    - n >> 31 是 0
    - n ^ 0 仍是 n
    - n - 0 仍是 n

- 當 n 是負數時:
    - n >> 31 是 -1
    - -1 以 2 補數表示為 0xFFFFFFFF
    - n ^ (-1) 等同於 1 補數運算
    - 最後再減 -1，得到 2 補數運算的值
 

## How to use and constructing Bitmasks

```c
//test if mine has either of two lowest bits on 	
(mine & 0x3) != 0

//test if mine has both of two lowest bits on 	
(mine & 0x3) == 0x3

//set lowest 8 bits of mine 	
mine |= 0xff

//clear every other bit in mine 	
mine &= 0x55555555 //clear bits in position 1/3/5/etc.
or
mine &= 0xaaaaaaaa //clear bits in position 2/4/5/etc. 
```

## Bitwise usual mask sample

```c
/*mask for 32-bit unsigned integer*/
/*we count starting from the least sinificant bit at position 0*/

//all bits on 	
~0 or -1

//one bit on in position n, all others off 	
1 << n

//n least significant bits on, all others off 	
(1 << n) - 1

//most significant bit on, all others off 	
(1 << 31)

//k most significant bits on, all others off 	
(~0 << (32 - k)) 
or
~(~0U >> k)
```

```c
/*Bitwise Manipulation*/
1 << x 	          //2 to the power x
~x + 1 	          //-x, arithmetic negation
x >> 31 	        //-1 if x is negative, 0 otherwise
x &= (x - 1) 	    //clears lowest "on" bit in x
(x ^ y) < 0 	    //true if x and y have opposite signs

```

## Bitwise usual sample

```c
/*Bitwise usual sample*/

//set a bit at position n
unsigned char a |= (1 << n);

//clear a bit at position n
unsigned char b &= ~(1 << n);

//write a macro for clear specific bit
#define CLEARBIT(a, pos) (a &= ~(1 << pos))

//toggle a bit
unsigned char c ^= (1 << n);

//test a bit
unsigned char e = d & (1 << n);//d has the byte value

//the right and left most byte
//assuming 16 bit, 2-byte short integer
unsigned char right = val & 0xff; //right most(least significant) byte
unsigned char left = (val >> 8) & 0xff; //left most(most significant) byte

//sign bit
//assuming 16 bit, 2-byte short integer, two's complement
bool sign = val & 0x8000;
```

## Setting and Clearing a Bit example
```c
#include <stdio.h>
#include <stdbool.h>

void binary(unsigned int n){
	for (int i = 256; i > 0; i /= 2){
		if(n & i)
			printf(" 1");
		else
			printf(" 0");
	}
	printf("\n");
}
			
bool getBit(int n, int index){
	return ((n & (1 << index)) > 0);
}

int setBit(int n, int index, bool b){
	if(b)
		return (n | (1 << index));
	int mask = ~(1 << index);
	return n & mask;
}

int main(void){
	int num = 16, index;
	
	printf("Input\n");
	for (int i = 7; i >= 0; i--)
		printf("%d ", getBit(num, i));
	printf("\n");
	
	//set bit
	index = 6;
	printf("#Setting $d-th bit\n", index);
	num = setBit(num, index, true);
	for (int i = 7; i >= 0; i--)
		printf("%d ", getBit(num, i));
	printf("\n");
	
	//unset(clear) bit
	index = 4;
	printf("#Unsetting %d-th bit\n", index);
	num = setBit(num, index, false);
	for (int i = 7; i >= 0; i--)
		printf("%d ", getBit(num, i));
	printf("\n");
	
	return 0;
}
```
