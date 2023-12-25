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
    opkg install sing-box
```

安装完后,`/etc/init.d/sing-box` 和 `/etc/config/sing-box` 会被创建,主要是用来开机启动和配置管理,后面会用到.

为了验证安装是否成功以及配置文件是否有问题,用`check` 和 `run` 命令分别测试一下.

```
    sing-box check -c {your_config_path}
    sing-box run -c {your_config_path} -D {your_work_dir} # 这个会在你运行的地方下载geoip ,geosite 以及cash dashboard 的UI 
```

如果上面的检测没有问题说明安装就正常了. 下面是要配置开机启动和防火墙.

### 配置防火墙

通过`opkg install sing-box` 自动创建的`/etc/init.d/sing-box` 脚本,运行是没有问题的,但是我自己通过`/etc/init.d/sing-box start` 启动后`sing-box` 进程也就退出了,不知道为何.于是我就参考[这里](https://github.com/rezconf/Sing-box/wiki/How-to-Run) 换了一个脚本.由于路由的防火墙是用的`nft-table`,因此防火墙这边做了一下调整.

```
// 在start 中增加
nft add table inet filter
nft add chain inet filter FORWARD { type filter hook forward priority 0\; }
nft insert rule inet filter FORWARD oifname "tun*" counter accept # 这里的tun要根据sing-box 配置文件中的网卡名字决定.

//在stop 中删除
nft delete table inet filter
```

`sing-box` 运行后会在路由器的`Network` -> `Interface` 中创建一个虚拟网卡,名字是和你的配置文件中的网卡名字一致的.

### References

* [How to Run](https://github.com/rezconf/Sing-box/wiki/How-to-Run)
* [netlink: 'sing-box': attribute type 22 has an invalid length](https://github.com/SagerNet/sing-box/issues/100)
* [nftables](https://en.wikipedia.org/wiki/Nftables)
