---
title: CH3.1 How to design a Scull Driver
parent: CH3 Scull Driver
layout: default
nav_order: 1
---

# CH3.1 How to design a Scull Driver

# 本章學習目的: 練習建構一個完整的 字元裝置驅動程式(char device driver / char driver)

> 大多數簡單的硬體裝置，都可以歸類為 “字元裝置”
> 

- 參考實際驅動程式 - scull (Simple Character Utility for Loading Localities)
- scull的硬體基礎是核心內部配置而來的一塊記憶體
- 電腦一定有這個硬體，能跑Linux就能跑scull
- scull沒有任何實際的功能，唯一的作用：展示核心與char driver之間的軟體介面

# 設計驅動程式

- 定義 驅動程式要提供哪些功能(機制facilities)給user-space程式
  
- 實作多個抽象層，分別包裝不同特性的目標裝置
    - scull0 ~ scull3 → 記憶體構成的device，具備 共通性global & 持續性persistent
    - scullpipe0 ~ scullpipe3 → FIFO device，實作 blocking and nonblocking 的read() write()
    - scullsingle → 一次只服務一個process 的 open()
    - scullpriv → 只專屬於每個虛擬操控台
    - sculluid → 允許被重複多次開啟，但只屬於同一位使用者，會告訴新的使用者device busy
    - scullwuid → 允許被重複多次開啟，但只屬於同一位使用者，會推延open()直到舊使用者關閉
      
- 這幾種不同特性的scull驅動程式
  在mechanism(機制)上都完全一樣，只是各自加上不同的policy(操作法則)

- 練習透過這些設計，熟悉各種可能的裝置管理辦法
