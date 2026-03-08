---
title: 搭载 sing-boxr 内核进行 DNS 分流教程-ruleset方案
description: 此教程搭载 sing-boxr 内核并使用其特性进行 DNS 分流，即指定国内域名 <code>rule_set:cn</code> 走国内 DNS 解析，国外域名走 <code>fake-ip</code>
date: 2024-08-22 18:24:06 +0800
categories: [DNS 配置, DNS 分流]
tags: [sing-box, sing-boxr, ShellCrash, ruleset, rule_set, 进阶, DNS, DNS 分流]
---

> 说明
{: .prompt-tip }
1. [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
2. 本教程以 ShellCrash 为例，其它客户端亦可参考
3. 本教程搭载 [sing-box 内核 reF1nd-Stable 版](https://github.com/reF1nd/sing-box/tree/reF1nd-stable)（导入内核方法可参考《[ShellCrash 和 AdGuard Home 快速安装教程/导入 mihomo 内核 或 sing-box 内核](https://proxy-tutorials.dustinwin.us.kg/posts/pin-toolsinstall/#%E4%BA%8C-%E5%AF%BC%E5%85%A5-mihomo-%E5%86%85%E6%A0%B8-%E6%88%96-sing-box-%E5%86%85%E6%A0%B8)》）
4. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，国外域名走 `fakeip`**。未知域名走 real-ip（在匹配 `route.rules.rule_set:cnip` 规则时会由国内 DNS 解析，解析出 IP 在国内则走 `🀄️ 国内 IP` 规则，否则走 `🐟 漏网之鱼` 规则）
5. 部分用户觉得未知域名处理方式会导致 DNS 泄露，可参考《[搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxr-ruleset)》

## 一、 导入规则集合文件
`route.rule_set` 须添加 `fakeip-filter`、`cn` 和 `proxy`，如下：

```json
{
  "route": {
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/fakeip-filter.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cn.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/proxy.srs"
      }
    ]
  }
}
```

## 二、 DNS 分流配置
1. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，将“DIRECT-DNS”、“PROXY-DNS”和“DEFAULT-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

2. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_hosts",
        "type": "hosts",
        "predefined": {
          "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
          "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ]
        }
      },
      { "tag": "dns_resolver", "type": "local" },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "dns_hosts" },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "dns_hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_quad9", "type": "https", "server": "dns11.quad9.net", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_dnspod", "dns_alidns" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_quad9" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "fakeip-filter" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 }  // 此条仅演示，可删除
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

4. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
- 注：为提高匹配效率，推荐参考《[生成带有自定义出站和规则的 sing-boxr 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxr-ruleset/)》编写 `route.rules`，将 `"action": "resolve"` 放置在域名规则集之后，IP 规则集之前

```json
{
  "route": {
    "rules": [
      { "action": "resolve", "match_only": true }
    ]
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
