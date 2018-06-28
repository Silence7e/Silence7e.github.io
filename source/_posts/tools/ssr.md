---
title: SSR 梯子搭建
date: 2018-06-28 16:55:25
categories: 工具
tags: [SS, 梯子]
---

CentOS ShadowsocksR单/多端口一键管理脚本
<!--more-->

环境： vultr centOS 6*64

安装SSR管理脚本

```
yum -y install wget
wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```
备用下载地址：
```
yum -y install wget
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```

安装成功之后显示如下图，如果下次还想调出此界面，输入 
```
bash ssr.sh
```

```
	---- Toyo | doub.io/ss-jc42 ----

  1. 安装 ShadowsocksR
  2. 更新 ShadowsocksR
  3. 卸载 ShadowsocksR
  4. 安装 libsodium(chacha20)
————————————
  5. 查看 账号信息
  6. 显示 连接信息
  7. 设置 用户配置
  8. 手动 修改配置
  9. 切换 端口模式
————————————
 10. 启动 ShadowsocksR
 11. 停止 ShadowsocksR
 12. 重启 ShadowsocksR
 13. 查看 ShadowsocksR 日志
————————————
 14. 其他功能
 15. 升级脚本

 当前状态: 未安装
```
输入 **1**

```
请输入要设置的ShadowsocksR账号 端口
(默认: 2333):
```

输入 **8989**

输入 **密码**

选择加密方式: **10**

选择协议插件： **2**

选择混淆插件： **1**

安装谷歌BBR加速SSR

```
yum -y install wget
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

安装完成输入 **y** 重启即可。