---
title: Django 开发的前期准备
date: 2017-8-10
categories: Django
---
包括虚拟环境的创建、pycharm 的安装与使用以及 Navicat 的使用

<!--more-->
# 创建虚拟环境
## 安装虚拟环境所需的开发包
```
pip install virtualenvwrapper-win
```
## 创建虚拟环境
```
mkvirtualenv test
```
## 退出虚拟环境
```
deactivate
```
## 查看有哪些虚拟环境
```
workon
```
## 进入虚拟环境
```
workon test
```
## 虚拟环境安装开发包
```
pip install xxxx
```
## 卸载开发包
```
pip uninstall xxxx
```
## 查看已安装的开发包
```
pip list
```
# pycharm 的安装与使用
## 下载
>pycharm
http://pan.baidu.com/s/1gfxi3S3

## 新建项目
[文件]->[新建项目]
![](\images\django\XinJianXiangMu.PNG)

（需要先安装好 Django==1.9.8）

## 查看已安装的开发包
Ctrl+Alt+S 进入设置页面，搜索 inter
![](\images\django\YiAnZhuang.PNG)

## 运行项目
Alt+Shlft+F10 选择 Django 项目运行

# Navicat 的安装与使用
## 安装 Mysql
>mysql for windows
http://pan.baidu.com/s/1mhYBCzI

## 安装 Navicat
>Navicat for MySQL 11.0.10 32+64位(内含破解补丁).rar
http://pan.baidu.com/s/1slRWj5j

## 新建连接
[文件]->[新建连接]->[Mysql]
![](\images\django\XinJianLianJie.PNG)
## 新建数据库
![](\images\django\NewShuJuKu.PNG)








