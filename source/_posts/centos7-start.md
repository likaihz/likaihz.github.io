---
title: CentOS7新系统安装以及用户管理
abbrlink: 3742663798
date: 2020-09-07 13:46:58
mathjax: true
tags:
- 运维
- 记录
---

闲来无事，想做个小项目玩玩，也可以趁此机会亲身体验开发的每一个环节。第一步就是搭建环境：一个应用服务器，用于部署后台代码。

## 环境概览

* 云服务：阿里云ECS，1核/2G/40G/1Mbps带宽
* 操作系统：CentOS7.8 64位
* MySQL 5.6
* Nginx 1.18
* Tomcat 8.5

<!-- more -->

## 服务器重装系统

在阿里云控制台可以直接为服务器更换操作系统，阿里云提供各种Linux发行版可以直接安装，并且在安装新系统之前可以指定密钥，这样在新系统安装好以后就可以直接在有私钥的机器上以root用户登录了。

## CentOS 7 用户管理

* 直接用root用户登录不太安全，还是要在重装系统后新建用户用于日常运维。
* 以root登录后，新建一个用户：

```bash
adduser litiezhu
```

* 为新用户添加sudo权限：在/ect/sudoers中找到一行：

{% asset_img sudoers_raw sudoers_raw.png 550 220 %}

在最后仿照上一行的格式为新用户配置sudo权限

{% asset_img sudoers_mod sudoers_mod.png 552 235 %}

* 切换到新用户：

```bash
su litiezhu
```

* 此时仍然无法直接从远程以新用户litiezhu的身份登录，因为还没有在服务器上的该用户下配置公钥匙。切换到litiezhu用户后，在其主目录（/home/litiezhu）下新建目录“.ssh”，在该目录下新建文件authorized_keys并把公钥的内容复制到这个文件里：
{% asset_img authorized_keys authorized_keys.png 473 143 %}

* 这时再试一下从以litiezhu的身份登录到服务器，登录失败，提示Permission denied (publickey,gssapi-keyex,gssapi-with-mic)：

{% asset_img permission_denied permission_denied.png 600 64 %}
* 应该是刚刚创建的.ssh目录的权限问题，再用root登录并改一下权限：

{% asset_img chmod chmod.png 600 142 %}

* 尝试登录，成功：

{% asset_img login login.png 600 110 %}
