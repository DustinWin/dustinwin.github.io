---
title: 搭载 sing-boxp 内核进行 DNS 分流教程-geodata 方案
description: 此方案适用于 sing-box，搭载 sing-boxp 内核并使用其特性进行 DNS 分流
date: 2024-08-22 18:15:49 +0800
categories: [DNS 配置, DNS 分流]
tags: [sing-box, sing-boxp, geodata, geosite, 进阶, DNS, DNS 分流]
---

## 说明
1. [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
2. 本教程以 ShellCrash 为例，其它客户端亦可参考
3. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，国外域名走 `fake-ip`，未知域名走国内 DNS 解析，解析出 IP 在国内则走国内 DNS 解析和 `🇨🇳 直连 IP` 规则，否则走 `fake-ip` 和 `🐟 漏网之鱼` 规则**
4. 部分用户觉得未知域名处理方式导致 DNS 泄露，可以参考《[搭载 sing-boxp 内核配置 DNS 不泄露教程-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/dnsnoleaks-singboxp-geodata)》

## 一、 导入路由规则文件
geosite.db 文件须包含 `fakeip-filter`、`proxy` 和 `cn`，geoip.db 文件须包含 `cn`，推荐导入我定制的[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

## 二、 DNS 分流配置
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式，选择“3 mix混合模式”  
<img src="/assets/img/dns/dns-mix.png" alt="ShellCrash DNS 运行模式设置" width="60%" />

2. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为“null”  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

3. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "dns": {
    "hosts": {
      "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ]
    },
    "servers": [
      { "tag": "dns_direct", "address": [ "https://doh.pub/dns-query", "https://dns.alidns.com/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://cloudflare-dns.com/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "geosite": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "geosite": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "fallback_rules": [ { "geoip": [ "cn" ], "server": "dns_direct" }, { "match_all": true, "server": "dns_fakeip" } ], "server": "dns_direct" }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "lazy_cache": true,
    "reverse_mapping": true,
    "mapping_override": true,
    "fakeip": {
      "enabled": true,
      "inet4_range": "198.18.0.0/15",
      "inet6_range": "fc00::/18",
      "exclude_rule": { "geosite": [ "fakeip-filter" ] }
    }
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
