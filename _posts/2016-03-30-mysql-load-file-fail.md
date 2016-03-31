---
layout: post
title: "使用mysql load data infile 导入数据失败"
date: 2016-03-30
backgrounds:
    - http://7xlrln.com1.z0.glb.clouddn.com/blog11773606558ad19c8eo.jpg
    - http://7xlrln.com1.z0.glb.clouddn.com/blog110539150725a08e0bo.jpg
thumb: http://7xlrln.com1.z0.glb.clouddn.com/blog11773606558ad19c8eo.jpg?imageView2/1/w/200/h/200
categories: work
tags: php
---

在使用mysql -e 查询出一个表的完整内容后，使用load data infile命令失败， 提示找不到文件 error code 13.
load data infile '/home/a' into table `tablename` fields terminated by '\t' optionally enclosed by '"' escaped by '"' lines terminated by '\n';


### 解决方法

```
* 1 sudo vi /etc/apparmor.d/usr.sbin.mysqld
* 2 添加目录权限 /data/\* rw,
* 3  sudo /etc/init.d/apparmor reload
```