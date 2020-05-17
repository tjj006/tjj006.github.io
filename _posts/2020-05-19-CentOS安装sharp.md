---
layout: post
title: "CentOS安装sharp"
subtitle: '从nvm，node，cnpm到sharp'
author: "陶叔"
header-style: text
tags:
    - NodeJS
---
> 最近用到了一个身份证图片生成器的node应用，其中有一个sharp模块安装不上，尝试了很多方法终于搞定。记录一下前后的过程，防止下次换一台服务器又忘记。

- 安装nvm
![-w538](https://tjj006-1302037511.cos.ap-shanghai.myqcloud.com/2020/05/17/15896019676547.jpg)

- 安装Node
```
nvm install v13.6.0
nvm use v13.6.0
```
![-w568](https://tjj006-1302037511.cos.ap-shanghai.myqcloud.com/2020/05/17/15896020703393.jpg)

- 安装Sharp
```
npm install sharp
```
> 问题：若Node版本太高，会导致安装失败（可通过nvm安装、使用其他Node版本）。也有可能下载不到安装包，停在那边卡死也不超时。。。
![](https://tjj006-1302037511.cos.ap-shanghai.myqcloud.com/2020/05/17/15896831367019.jpg)

- 解决卡死问题：安装cnpm，重装sharp
```
npm install --global cnpm
cnpm install sharp
```

![-w817](https://tjj006-1302037511.cos.ap-shanghai.myqcloud.com/2020/05/17/15896298860699.jpg)

![-w1005](https://tjj006-1302037511.cos.ap-shanghai.myqcloud.com/2020/05/17/15896298374794.jpg)

