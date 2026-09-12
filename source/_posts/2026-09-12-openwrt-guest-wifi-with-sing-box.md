---
title: OpenWrt 访客网络配置，以及与 sing-box auto_redirect 的冲突排查
tags:
  - OpenWrt
  - 网络
categories: 2026-09
date: 2026-09-12 15:30:00
---

家里的路由器（TP-Link TL-XDR6088，刷的 ImmortalWrt）一直是单网段扁平网络，所有设备都在 `192.168.8.0/24` 里。最近想给来家里的客人单独开一个访客 Wi-Fi，要求很简单：能上网，但摸不到内网设备和路由器管理后台。配置本身不复杂，但因为路由器上跑着 sing-box（`auto_redirect` 模式全局接管流量），踩了一个值得记录的坑：访客网络配好后，国内网站正常，国外全部不通。

### 目标拓扑

```
主网  lan    192.168.8.0/24  原有 SSID        可互访内网、可管理路由器
访客  guest  192.168.9.0/24  新 SSID(2.4G)    只能上网，禁内网、禁管理
```

只动新增配置，不修改任何现有的 lan/wan/无线配置，把风险降到最低。动手前先 `sysupgrade -b` 备份一份。

### 基础配置

**① 新建 guest 网桥和接口**（`/etc/config/network`）：

```
config device 'br_guest'
	option type 'bridge'
	option name 'br-guest'

config interface 'guest'
	option proto 'static'
	option device 'br-guest'
	option ipaddr '192.168.9.1'
	option netmask '255.255.255.0'
```

**② 访客 DHCP**（`/etc/config/dhcp`）：独立地址池，租期缩短到 1 小时：

```
config dhcp 'guest'
	option interface 'guest'
	option start '100'
	option limit '100'
	option leasetime '1h'
```

**③ 防火墙隔离**（`/etc/config/firewall`）：核心思路是 guest 区域 `input=REJECT`（禁止访问路由器本身）、`forward=REJECT`（禁止转发到其他内网区域），只放行到 wan 的转发，再单独开 DHCP 和 DNS 两个小口子：

```
config zone 'guest'
	option name 'guest'
	option network 'guest'
	option input 'REJECT'
	option output 'ACCEPT'
	option forward 'REJECT'

config forwarding 'guest_wan'
	option src 'guest'
	option dest 'wan'

config rule 'guest_dhcp'
	option name 'Guest-DHCP'
	option src 'guest'
	option proto 'udp'
	option dest_port '67-68'
	option target 'ACCEPT'

config rule 'guest_dns'
	option name 'Guest-DNS'
	option src 'guest'
	option proto 'tcp udp'
	option dest_port '53'
	option target 'ACCEPT'
```

**④ 访客 SSID**（`/etc/config/wireless`）：在 2.4G 射频（radio0）上多开一个 AP，绑到 guest 网络，`isolate` 开启客户端隔离让访客之间也互相不可见：

```
config wifi-iface 'guest_2g'
	option device 'radio0'
	option network 'guest'
	option mode 'ap'
	option ssid 'your-guest-ssid'
	option encryption 'psk2'
	option key 'your-password'
	option isolate '1'
```

生效：

```
uci commit
service network reload
service firewall reload
service dnsmasq restart
wifi reload
```

一个小坑：`service network reload` 用 `nohup ... &` 后台跑时被 SSH 会话退出连带杀掉了，导致新接口没加载。在前台重新执行一次 `service network reload` 后 `br-guest` 才起来。涉及生效步骤时还是前台跑、盯着输出比较稳。

### 问题：访客网国内通、国外不通

配完后手机连访客 Wi-Fi 测试：拿到 `192.168.9.x` 地址 ✅，访问内网被拒 ✅，国内网站正常 ✅，但国外网站全部超时 ❌。

主网设备完全正常，说明问题只出在 guest 区域的特殊性上。路由器的代理是 sing-box，tun 入站配了 `auto_route + auto_redirect`，它会自动生成 nftables 规则全局拦截 TCP/UDP 流量。打开它的 prerouting 链看（有计数器，非常方便排查）：

```
chain prerouting {
	type nat hook prerouting priority dstnat + 2; policy accept;
	iifname "singbox-tun" return
	ip daddr { 192.168.0.0/16 } return              # 私网绕行
	ip daddr @route_exclude_address_set return      # geoip-cn 绕行
	meta l4proto tcp redirect to :33903 return      # 其余 TCP 重定向到本地随机端口
	meta mark set 0x2023 ct mark set 0x2023 return  # 其余 UDP/ICMP 打标走 tproxy
}
```

关键点：**这些规则是全局的，不区分来源接口**。国外 TCP 被 `redirect` 到路由器本地端口、UDP 被打上 `0x2023` 标记交给 tproxy——被拦截的包在网络栈里走的是 **INPUT 链**，而不是 FORWARD。

再看主网和访客网的区别：lan 区域 `input=ACCEPT`，被重定向的包顺利被 sing-box 接收；而 guest 区域 `input=REJECT`，只放行了 53 和 67-68，重定向到随机端口的包全部被 `reject_from_guest` 拒掉。国内流量因为命中 geoip-cn 绕行规则根本不进入拦截，所以表现为"国内通、国外不通"。DNS 不受影响（53 已放行），解析正常但连接建立不了，故障现象完全吻合。

### 修复：fw4 的 nftables include

第一反应是加一条"放行 TCP 33903 端口"的规则，但 33903 是 sing-box 每次启动时**随机分配**的，写死端口重启 sing-box 后就失效了。正确的思路是按 conntrack 特征匹配"被拦截"的流量，与端口无关：

- 被 `redirect` 的 TCP：conntrack 带 DNAT 状态 → `ct status dnat`
- 被 tproxy 接管的 UDP/ICMP：带标记 → `ct mark 0x2023`

本来想用 fw4 规则的 `option extra` 写原生 nft 表达式，结果这个版本的 fw4 不支持该选项（reload 时会警告并跳过规则）。fw4 支持 `config include` 类型的 `nftables` 片段，可以插入指定链的指定位置，正好够用。

先写片段文件 `/etc/fw4-guest-singbox.nft`：

```
ct status dnat counter accept comment "Guest-Singbox-TCP"
ct mark 0x2023 counter accept comment "Guest-Singbox-UDP"
```

再在 `/etc/config/firewall` 注册 include，插到 `input_guest` 链最前面（必须在 reject 跳转之前）：

```
config include 'guest_singbox'
	option type 'nftables'
	option chain 'input_guest'
	option position 'chain-prepend'
	option path '/etc/fw4-guest-singbox.nft'
```

`service firewall reload` 后验证链内容，手机再测，国外网站恢复，且 `Guest-Singbox-TCP` 规则的计数器开始增长，确认流量走的就是这条规则。

### 持久化

- 所有 UCI 配置（`/etc/config/`）和 nft 片段（`/etc/`）都在闪存 overlay 分区上，重启不丢。
- 但 sysupgrade 固件升级只保留默认列表里的文件，自定义的 `/etc/fw4-guest-singbox.nft` 需要手动登记到 `/etc/sysupgrade.conf`：

```
echo "/etc/fw4-guest-singbox.nft" >> /etc/sysupgrade.conf
```

### 小结

最终的隔离效果：访客可以正常上网（国外流量自动走 sing-box 分流，和主网体验一致），但访问不了内网任何设备，也打不开路由器的 LuCI/SSH，访客设备之间也被 AP 隔离。整个方案的要点有两个：一是全部使用新增配置段、不动现有配置，出问题可以随时 `uci revert`；二是理解了"透明代理拦截的流量走 INPUT 链"这一点后，guest 区域 `input=REJECT` 与代理的共存问题就变成了一个清晰的防火墙放行问题——按 conntrack 特征放行，而不是按易变的端口放行。
