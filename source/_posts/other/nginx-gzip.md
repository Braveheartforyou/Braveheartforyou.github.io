---
title: nginx 配置前端 字段 gzip
date: 2018-11-14 17:21:14
tags: [nginx]
categories: [nginx]
description: nginx 配置gzip 压缩读取 javascript、css、json等
---
## 安装nginx
windows 直接从[nginx官网](http://nginx.org/en/download.html)下载即可
## 配置
```bash
# 运行nginx 在nginx安装目录运行nginx.exe
# 打开cmd
nginx.exe start
# 打开conf/nginx.conf添加下面几行
# gzip  on;
# gzip_comp_level 1;
# gzip_min_length 256;
# gzip_types text/plain text/css application/json application/javascript text/javascript;
nginx.exe -s reload
```
就可以在 localhost:8080中打开network 查看 js、css已经压缩