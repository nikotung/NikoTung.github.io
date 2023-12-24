---
title: Install sing-box on Openwrt
date: 2023-12-24 22:33:29
categories: "2023-12"
tags: [读书]
---

前段时间因为clash 删库的事情，就在网上查了一下有什么替代品之类的，发现了一个[sing-box](https://sing-box.sagernet.org/) 的工具。它支持现在机场上提供的大部分的协议，发现它也是很早前自己曾经使用的sagernet(一个Android 的代理工具)的那群开发者开发的，于是就简单的了解了一下，也在手机使用了一段时间，发现整体的体验还不错。所以就折腾了一下把全平台都迁移一下过来，这样就用一个通用的配置就全平台通用。Android 、iOS 和 Mac OS 上都有客户端可以直接安装就使用了，家里的路由器之前是装的是[immortalwrt](https://github.com/immortalwrt)，需要做一些配置才能安装好使用。


### 安装

在openwrt 的[packages](https://github.com/openwrt/packages/tree/master/net/sing-box) 中已经有了sing-box 的安装源，目前的版本是1.74。因此整个安装会比较简单用`opkg` 就能完成。

更新一下packages 然后把依赖也一并安装

```
    opkg update
    opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft
```


### References

* [How to Run](https://github.com/rezconf/Sing-box/wiki/How-to-Run)
