---
layout: post
title: "OpCache使用"
date: 2016-03-09
backgrounds:
    - http://7xlrln.com1.z0.glb.clouddn.com/blog11773606558ad19c8eo.jpg
    -http://7xlrln.com1.z0.glb.clouddn.com/blog110539150725a08e0bo.jpg
thumb: http://7xlrln.com1.z0.glb.clouddn.com/blog110539150725a08e0bo.jpg?imageView2/1/w/200/h/200
categories: work
tags: php
---
php5.5 以上内建了OpCache，但是默认没有开启。现在需要开启OpCache缓存。

### 修改php.ini 文件
```sh
sudo vi /etc/php5/fpm/php.ini
```
修改关于OpCache部分配置.
```sh
; 开关打开
opcache.enable=1

; 可用内存, 酌情而定, 单位 megabytes
opcache.memory_consumption=256

; 对多缓存文件限制, 命中率不到 100% 的话, 可以试着提高这个值
opcache.max_accelerated_files=5000

; Opcache 会在一定时间内去检查文件的修改时间, 这里设置检查的时间周期, 默认为 2, 定位为秒
opcache.revalidate_freq=240
```
### 重启服务器，启用新配置
```sh
sudo service php5-fpm restart
sudo service nginx restart
```
### web界面工具
在某个可以对外访问目录修改访问权限.
```sh
wget https://raw.github.com/amnuts/opcache-gui/master/index.php -O op.php
```
### 参考
* [https://easyengine.io/tutorials/php/zend-opcache/](https://easyengine.io/tutorials/php/zend-opcache/)