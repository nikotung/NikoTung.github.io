---
title: 使用mosdns 做DNS 分流
date: 2023-01-14 22:54:39
categories: "2023-01"
tags: [DNS]
---

之前一直有用[nextdns](https://nextdns.io/) 作为DoH，用于过滤广告、tracking和DNS 加密，在家里也是配合openclash 使用。使用mosdns 的主要原因是，它可以直接配置DoH 服务器的IP 地址，因为nextdns 被墙了，没办法直接使用。

### 安装

之前的ipk 版本都是手动安装，而这次看到一个[luci-app-mosdns](https://github.com/sbwml/luci-app-mosdns) 可以直接一键安装，于是就把老版本的直接卸载掉重新安装。这次安装的版本是v4.5.3。

```
sh -c "$(curl -ksS https://raw.githubusercontent.com/sbwml/luci-app-mosdns/master/install.sh)"
```

配置其实就很简单，把你远程DNS 配置上去就可以，本地DNS 也可以配置，也可以使用自定义配置的形式。另外就在openclash 上启用“自定义上游 DNS 服务器”选项打开，然后把服务地址配置成`127.0.0.1:5335`。

![](/assets/openclash-dns.png)

生成的配置大概像下面一样。

```
log:
  level: info
  file: "/tmp/mosdns.log"

include: []

data_providers:
  - tag: geoip
    file: "/usr/share/v2ray/geoip.dat"
    auto_reload: true

  - tag: geosite
    file: "/usr/share/v2ray/geosite.dat"
    auto_reload: true

  - tag: whitelist
    file: "/etc/mosdns/rule/whitelist.txt"
    auto_reload: true

  - tag: blocklist
    file: "/etc/mosdns/rule/blocklist.txt"
    auto_reload: true

  - tag: hosts
    file: "/etc/mosdns/rule/hosts.txt"
    auto_reload: true

  - tag: redirect
    file: "/etc/mosdns/rule/redirect.txt"
    auto_reload: true

plugins:
  - tag: lazy_cache
    type: cache
    args:
      size: 200000
      lazy_cache_ttl: 259200

  - tag: modify_ttl
    type: ttl
    args:
      minimal_ttl: 0
      maximum_ttl: 0

  - tag: "forward_local"
    type: fast_forward
    args:
      upstream:
        - addr: 119.29.29.29
        - addr: 114.114.114.114

  - tag: "forward_remote"
    type: fast_forward
    args:
      upstream:
        - addr: tls://8.8.8.8
        - addr: tls://1.1.1.1

  - tag: query_is_whitelist_domain
    type: query_matcher
    args:
      domain:
        - "provider:whitelist"

  - tag: query_is_blocklist_domain
    type: query_matcher
    args:
      domain:
        - "provider:blocklist"

  - tag: query_is_hosts_domain
    type: hosts
    args:
      hosts:
        - "provider:hosts"

  - tag: query_is_redirect_domain
    type: redirect
    args:
      rule:
        - "provider:redirect"

  - tag: query_is_local_domain
    type: query_matcher
    args:
      domain:
        - "provider:geosite:cn"

  - tag: query_is_non_local_domain
    type: query_matcher
    args:
      domain:
        - "provider:geosite:geolocation-!cn"

  - tag: response_has_local_ip
    type: response_matcher
    args:
      ip:
        - "provider:geoip:cn"

  - tag: query_is_ad_domain
    type: query_matcher
    args:
      domain:
        - "provider:geosite:category-ads-all"

  - tag: match_qtype65
    type: query_matcher
    args:
      qtype: [65]

  - tag: "main_sequence"
    type: "sequence"
    args:
      exec:
        - _misc_optm
        - query_is_hosts_domain
        - query_is_redirect_domain

        - if: query_is_whitelist_domain
          exec:
            - forward_local
            - modify_ttl
            - _return

        - if: "query_is_blocklist_domain || query_is_ad_domain || match_qtype65"
          exec:
            - _new_nxdomain_response
            - _return

        - lazy_cache

        - if: query_is_local_domain
          exec:
            - forward_local
            - modify_ttl
            - _return

        - if: query_is_non_local_domain
          exec:
            - _prefer_ipv4
            - forward_remote
            - modify_ttl
            - _return
        - primary:
            - forward_local
            - if: "(! response_has_local_ip) && [_response_valid_answer]"
              exec:
                - _drop_response
          secondary:
            - _prefer_ipv4
            - forward_remote
            - modify_ttl
          fast_fallback: 200

servers:
  - exec: main_sequence
    listeners:
      - protocol: udp
        addr: ":5335"
      - protocol: tcp
        addr: ":5335"
```

### nextdns

因为DoH 被墙了，只能用IP，`dig dns.nextdns.io`，把解析到的ip 配置到mosdns 的hosts 插件中。


### Reference

* [🚀使用mosdns替代openclash默认的dns分流](https://kuokuo.io/2022/03/24/openclash-with-mosdns/)
* [v2ray dat rules](https://github.com/Loyalsoldier/v2ray-rules-dat/releases)
* [mosdns](https://irine-sistiana.gitbook.io/mosdns-wiki/)