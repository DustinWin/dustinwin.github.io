---
title: 搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案
description: 此教程搭载 sing-boxr 内核并使用其特性防止 DNS 泄露，即针对未知域名走国外 DNS 解析
date: 2024-08-22 16:51:20 +0800
categories: [DNS 配置, DNS 防泄漏]
tags: [sing-box, sing-boxr, ShellCrash, ruleset, rule_set, 进阶, DNS, DNS 泄露]
---

> 说明
{: .prompt-tip }
1. 此方案彻底防止了 DNS 泄露（针对未知域名走国外 DNS 解析，解析出 IP 在国内则走国内 DNS 解析和 `🀄️ 直连 IP` 规则，否则走 `fake-ip` 和 `🐟 漏网之鱼` 规则），兼容性无法保证，请慎用
2. 本教程以 [ShellCrash](https://github.com/juewuy/ShellCrash) 为例，其它客户端亦可参考
3. 可进入 <https://ipleak.net> 测试 DNS 是否泄露，“DNS Addresses” 栏目下没有中国国旗（因 `ipleak.net` 属未知域名，默认走 `🐟 漏网之鱼` 规则），即代表 DNS 没有发生泄露

## 一、 导入规则集合文件
`route.rule_set` 须添加 `fakeip-filter`、`cn`、和 `proxy`，如下：

```json
{
  "route": {
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/fakeip-filter.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/cn.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/proxy.srs"
      }
    ]
  }
}
```

## 二、 DNS 防泄漏配置
1. 进入主菜单 → 2 内核功能设置 → 2 切换 DNS 运行模式，选择“3 mix混合模式”  
<img src="/assets/img/dns/dns-mix.png" alt="ShellCrash DNS 运行模式设置" width="60%" />

2. 进入主菜单 → 2 内核功能设置 → 2 切换 DNS 运行模式 → 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

3. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），修改为如下内容：
>推荐将 `client_subnet` 设置为当前网络的公网 IP 段，如当前网络公网 IP 为 `202.103.17.123`，可设置为 `202.103.17.0/24`（后续维护更新可直接执行命令 `sed -i -E "s/(\"client_subnet\": \")[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/jsons/dns.json`）
{: .prompt-info }

```json
{
  "dns": {
    "servers": [
      {
        "tag": "hosts",
        "type": "hosts",
        "predefined": {
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ]
        }
      },
      { "tag": "dns_resolver", "type": "https", "server": "223.5.5.5"},
      { "tag": "dns_direct", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "dns_resolver" },
      // `outbounds` 里必须存在 `🚀 节点选择`
      { "tag": "dns_proxy", "type": "https", "server": "dns.google", "domain_resolver": "dns_resolver", "detour": "🚀 节点选择" },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.1/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "hosts" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "fakeip-filter", "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "disable_cache": true,
    "disable_expire": false,
    "independent_cache": true,
    "reverse_mapping": true,
    "client_subnet": "202.103.17.0/24"
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

4. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "route": {
    "rules": [
      { "action": "resolve", "server": "dns_proxy" }
    ]
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
