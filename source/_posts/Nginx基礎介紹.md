---
title: Nginx基礎介紹
date: 2018-07-08 11:55:28
tags:
---
#### 什麼是Nginx?<br>
##### HTTP伺服器
作為HTTP伺服器，可以極快的速度為伺服器上的靜態文件（ex:HTML、圖片）通過HTTP協議展現給客戶端

##### 反向代理伺服器
客戶端本來可以直接通過HTTP訪問某網站的應用伺服器，如果管理員在中間加上一個Nginx，客戶端會先請求Nginx，Nginx再請求應用伺服器，並將結果返回給客戶端，此時Nginx就是反向代理伺服器。

![](Nginx基礎介紹/reproxy.jpg)
