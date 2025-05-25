---
title: GOLANG
date: 2023-12-29 14:38:57
categories: 
- GOLANG
tags: 
- [GO]
- [study]
---

go lang 学习记录，以及一些遇到的问题
![logo](logo.jpg)

<!-- more -->

## 安装
主页：  https://go.dev/  下载
下载后直接安装。

安装后查询： 
``` shell
go version
```
若出现版本展示，则视为安装成功

### 配置代理 proxy
``` shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```
