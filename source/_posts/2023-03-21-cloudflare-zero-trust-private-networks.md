---
title: cloudflare zero trust:private networks
date: 2023-03-21 21:31:46
categories: "2023-03"
tags: [cloudflare,内网穿透]
---

自疫情来，有居家办公开始，我对内网穿透的需求越来越大且越来越依赖。公司网络需要安装DLP（Data Loss Prevention）软件才能访问，我称之为监控，个人电脑需要访问的话就必须要安装DLP 后才能使用VPN 接入，个人电脑我是不想并且不会安装DLP的，并且我也不想天天背着公司电脑上下班，于是就在想有没有什么办法可以绕过这个DLP 这个监控软件还能安全连接到公司内网。中间折腾花了不少时间，刚好办公室里有一台闲置的PC安装了Ubuntu并且常年开机，尝试过的方案有[softether](https://www.softether.org/)、[ngrok](https://ngrok.com/)、[tailscale](https://tailscale.com/)以及[cloudflare zero trust](https://www.cloudflare.com/zero-trust-hub/)，发现tailscale 已经能很好满足我的需求，配置好`exit-node` 和 `subnets`就能把网络打通，最近又发现cloudflare 的zero trust 已经发布并且把之前的tunnel 功能也集成在里面，它也能做内网穿透，直接把内网暴露到cloudflare 的网络中，这样就可以在我个人的电脑上安装warp 接入到cloudflare 中就可以访问公司网络了，目前这个功能在少于50个用户的组织中是免费的，而且zero trust 还能做代理访问防火墙（GFW）外的内容而不需要你像tailscale一样需要有一台外网VPS 才能做到，流量也没有限制。它的架构大概如下（图片来源cloudflare 博客）：
![](./assets/cloudflare-private-networks.png)

下面记录一下如何用cloudflare zero trust 做内网穿透。

## 要求
* 有一个cloudflare 账户并且关联了一个域名，这个域名可以在cloudflare 购买或者把域名nameserver 指向cloudflare 家的`rihana.ns.cloudflare.com` 和 `sam.ns.cloudflare.com`
* 开通cloudflare zero trust,现在可以直接在dashboard 上操作了，点开后按提示操作就可以了，主要是创建一个组织并且选择一个付费计划，选择免费的即可。

## Dashboard 上的一些配置
Zero Trust 是一个面向企业的服务，因此它有很多规则类的配置，包括一些准入规则，屏蔽规则等等。这里主要配置一下谁能加入我们的组织以及怎么验证。

### Login methods
这个配置的作用是在于如果别人要加入你的组织，你需要通过什么方式去验证加入的这个用户的身份，cloudflare 内部集成了很多种，包括Github、Facebook、Google、Okta 和 One-time PIN等等，One-time PIN 好像是默认的，可以选择自己需要的验证方式，入口：**Settings->Authentication->Login methods**,进入后按照提示，跟着添加步骤走就可以了。

### Warp Client
用户是通过Warp client 来加入组织的，只要在电脑或者手机上安装Cloudflare Warp 这个应用后，在账户中进入选择加入Zero Trust networks 后根据提示输入组织名，然后再选择验证方式，上面的Login methods 配置了什么这里就会出现，跟着提示操作就可以。这里主要是配置符合什么样的条件才能加入你的组织，入口：**Settings->Warp Client->Device enrollment**，点击`Manage`进入配置规则，在`Rules`下新增一个规则，这个规则可以有多种表达方式，比如指定邮箱、指定邮箱域名、Login methods 以及国家等等，下面是一个示例：
![](./assets/cloudflare-rule-example.png)

表示`xxx@gmail.com` 或者`duckduckgo.com` 后缀的邮箱都可以加入组织，里面还有很多规则可以根据自己的需要选择。

有了上面的两个配置后就可以在电脑或者手机上安装Cloudflare Warp 应用然后加入组织了，加入后就可以做代理应用了。

## Cloudflare Tunnel
Tunnel 就是在你的内网机器上安装`cloudflared` 运行一个后台服务与cloudflare 网络创建一个outbound-only 的连接，从而把你的网络暴露到cloudflare 的网络中。

>Cloudflared establishes outbound connections (tunnels) between your resources and the Cloudflare edge. Tunnels are persistent objects that route traffic to DNS records. Within the same tunnel, you can run as many cloudflared processes (connectors) as needed. These processes will establish connections to the Cloudflare edge and send traffic to the nearest Cloudflare data center.
![](./assets/how-cloudflare-tunnel-works.jpeg)

### 安装 cloudflared
`cloudflared`在[GitHub](https://github.com/cloudflare/cloudflared/releases) 上有很多编译好的二进制文件，选择适合自己的安装即可，具体的安装步骤参考[install-and-setup/installation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)


### 创建Tunnel
创建Tunnel 有两个途径，一个是在dashboard 页面上操作，一个是安装了cloudflared 机器上用命令行安装。这里建议是通过命令行安装，因为我试过了几次用cloudflare 的dashboard 安装总是不成功。

首先第一步是先登录，`cloudflared tunnel login`，点击控制台的链接进行登录授权。成功后会在当前用户的`$HOME/.cloudflared/`目录下创建一个证书`cert.pem`

接着就是创建Tunnel `cloudflared tunnel create vpc-office`,把这里的vpc-office 换成你自己的名字，创建成功后会打印出Tunnel 的uuid，并且在`$HOME/.cloudflared` 目录下创建一个以uuid为名字的json 文件，比如`07cc7011-2d18-49eb-9d4a-2db9b4d50067.json`

在`$HOME/.cloudflared` 下创建一个配置文件`config.yml`,内容如下：
```
	tunnel: 07cc7011-2d18-49eb-9d4a-2db9b4d50067 # 这里就是tunnel 的uuid
	credentials-file: /home/root/.cloudflared/07cc7011-2d18-49eb-9d4a-2db9b4d50067.json # json文件
	warp-routing:
	   enabled: true #这里是暴露私有网络，如果是仅仅暴露一个服务的话可以不用开启
```

运行Tunnel `cloudflared tunnel run vpc-office`看看是否Tunnel 能运行成功。这样只是在前台运行成功，这里要暴露网络，因此需要运行成后台，具体可以参考[run-as-a-service](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/as-a-service/)

我这里是Ubuntu，操作如下：把`config.yml` 拷贝到`/etc/cloudflared/config.yml`，然后`sudo cloudflared service install` 这样就可以安装成功了，通过`systemctl status cloudflared`可以查看运行的状态。

另外，也可以改一下cloudflared参数`sudo sysctl -w net.core.rmem_max=2500000` 

### 暴露网络
在cloudflared 安装好后，需要配置一些路由规则来告诉cloudflare 那些网段是走这个Tunnel。

```
	cloudflared tunnel route ip add 192.168.2.0/24 vpc-office
	cloudflared tunnel route ip add 192.168.14.0/24 vpc-office
	cloudflared tunnel route ip add 10.0.20.0/24 vpc-office
```
上面的命令就是把192.168.2.0/24，192.168.14.0/24 和 10.0.20.0/24配置成走这个Tunnel，也就是组织内的用户在访问到的地址在这个三个网段时就会被路由到这个公司内网中，具体的网段要根据自己的实际情况配置。

### Split tunnle 配置
Cloudflare 默认有一些内网IP是不会经过它的网关的，而是直接在用户设备上连接的，因此要确认是否跟你公司的内网网段重复，如果是的话需要在cloudflare 中把它删掉或者添加这样你配置的路由规则才会生效。
配置入口：**Settings->Network->Split tunnels and local domain fallback->Device settings**，在**Device settings**下选择你的profile 点击进入配置，找到**Split Tunnels**。根据配置的模式，如果是**Include**，那么就要把上面配置的网段加进去，如果是**Exlude** 那么就看看cloudflare 默认不走路由规则的网段是否和你要配置的网段冲突，如果是就把它删掉即可。

### Local Domain Fallback
很多时候，公司内部有很多内部域名，需要通过内部的DNS 服务器去解析才行，这时就需要配置**Local Domain Fallback**，在上面Split tunnel 配置的页面中找到**Local Domain Fallback**，点击进入管理，在域名项输入内网的域名，在DNS Servers项输入内部的DNS 服务器添加即可。


## 参考
* [building-many-private-virtual-networks-through-cloudflare-zero-trust](https://blog.cloudflare.com/building-many-private-virtual-networks-through-cloudflare-zero-trust/)
* [securely-access-home-network-with-Cloudflare-Tunnel-and-WARP](https://savjee.be/blog/securely-access-home-network-with-Cloudflare-Tunnel-and-WARP/)
* [Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/)