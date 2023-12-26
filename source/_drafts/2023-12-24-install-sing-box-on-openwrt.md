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

sing-box 服务启动后不能自己代理流量的,即便你开了了`auto_route`,需要简单配置一个防火墙.

在/etc/config/firewall中添加规则：

```
config zone
        option name 'proxy'
        option forward 'REJECT'
        option output 'ACCEPT'
        option input 'ACCEPT'
        option mtu_fix '1'
        option device 'tun0'
        list network 'proxy'

config forwarding
        option name 'lan-proxy'
        option dest 'proxy'
        option src 'lan'
```

添加网络接口/etc/config/network

```
 config interface 'proxy'
        option proto 'none'
        option device 'tun0'
```

配置开机启动

```
config sing-box 'main'
        option enabled '1'
        option user 'root' # 如果你是用tun 模式的话,一定要改成root
        option conffile '/etc/sing-box/config.json'
        option workdir '/usr/share/sing-box'
```

### 配置订阅

我是自己fork 了一个[clash2singbox](https://github.com/NikoTung/clash2singbox),在原来的基础上加了地区分组.主要有:

* HongKong
* USA
* Japan
* Singapore
* Taiwan
* Others

### References

* [How to Run](https://github.com/rezconf/Sing-box/wiki/How-to-Run)
* [netlink: 'sing-box': attribute type 22 has an invalid length](https://github.com/SagerNet/sing-box/issues/100)
* [nftables](https://en.wikipedia.org/wiki/Nftables)
