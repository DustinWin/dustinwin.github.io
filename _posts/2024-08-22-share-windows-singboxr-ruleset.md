---
title: 分享 sing-boxr for Windows 采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxr 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 20:01:28 +0800
categories: [分享配置, Windows]
tags: [sing-box, sing-boxr, Windows, ruleset, rule_set, 分享]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案采用**裸核**的方式运行，更加精简
3. 本教程搭载 [sing-box 内核 reF1nd-Test 版](https://github.com/reF1nd/sing-box/tree/reF1nd-testing)

## 一、 生成配置文件 .json 文件直链
具体方法请参考《[生成带有自定义出站和规则的 sing-boxr 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxr-ruleset)》，贴一下我使用的配置：
- 注：推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

```json
{
  "providers": [
    {
      "tag": "🛫 机场订阅",
      "type": "remote",
      // 修改为你的 Clash 订阅链接
      "url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport.yaml",
      // 若出现获取不了机场节点的情况，可删除此配置项
      "user_agent": "clash.meta",
      "include": "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)",
      "health_check": {
        "enabled": true,
        "url": "https://www.gstatic.com/generate_204"
      }
    },
    {
      "tag": "🆓 免费订阅",
      "type": "remote",
      // 修改为你的 sing-box 订阅链接
      "url": "https://example.com/xxx/xxx",
      "path": "./providers/free.json",
      "update_interval": "12h",
      "health_check": {
        "enabled": true,
        "url": "https://www.gstatic.com/generate_204"
      }
    }
  ],
  "log": { "level": "error", "timestamp": true },
  "dns": {
    "servers": [
      {
        "tag": "hosts",
        "type": "hosts",
        "predefined": {
          "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
          "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ]
        }
      },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "hosts" },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "hosts", "detour": "GLOBAL" },
      { "tag": "dns_cloudflare", "type": "https", "server": "cloudflare-dns.com", "domain_resolver": "hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_alidns", "dns_dnspod" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_cloudflare" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "action": "evaluate", "server": "hosts" },
      { "match_response": true, "ip_accept_any": true, "action": "respond" },
      { "clash_mode": [ "Direct" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "action": "predefined" },
      { "rule_set": [ "trackerslist", "microsoft-cn", "apple-cn", "google-cn", "games-cn" ], "server": "dns_direct" },
      { "rule_set": [ "games", "ai", "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "private", "cn" ], "server": "dns_direct" },
      { "action": "evaluate", "server": "dns_direct" },
      { "match_response": true, "rule_set": [ "cnip" ], "action": "respond" },
      { "match_response": true, "ip_accept_any": true, "invert": true, "server": "dns_proxy" },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "optimistic": true,
    "reverse_mapping": true,
    // 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    "client_subnet": "211.137.58.0/24"
  },
  "http_clients": [ { "tag": "detour_proxy", "version": 3, "detour": "GLOBAL" } ],
  "inbounds": [
    { "tag": "tun-in", "type": "tun", "interface_name": "sing-box", "address": [ "172.18.0.1/30", "fdfe:dcba:9876::1/126" ], "auto_route": true, "strict_route": true, "stack": "mixed" }
  ],
  "outbounds": [
    { "tag": "节点选择", "type": "selector", "outbounds": [ "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "网络测试", "type": "selector", "outbounds": [ "全球直连", "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "游戏平台", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "🆚 vless 节点" ] },
    { "tag": "AI 平台", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "🆚 vless 节点" ] },
    { "tag": "游戏服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "微软服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "谷歌服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "苹果服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "国内域名", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "国内 IP", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "国外域名", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "电报消息", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "直连软件", "type": "selector", "outbounds": [ "全球直连" ] },
    { "tag": "私有网络", "type": "selector", "outbounds": [ "全球直连" ] },
    { "tag": "漏网之鱼", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点", "全球直连" ] },
    { "tag": "广告域名", "type": "selector", "outbounds": [ "全球拦截", "全球直连" ] },
    { "tag": "全球拦截", "type": "block" },
    { "tag": "全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "DIRECT", "type": "direct" },
    { "tag": "GLOBAL", "type": "selector", "outbounds": [ "节点选择", "DIRECT" ] },
    // 若没有单个出站节点，须删除所有 `🆚 vless 节点` 相关内容
    {
      "tag": "🆚 vless 节点",
      "type": "vless",
      "server": "example.com",
      "server_port": 443,
      "uuid": "{uuid}",
      "network": "tcp",
      "tls": { "enabled": true, "server_name": "example.com", "insecure": false },
      "transport": { "type": "ws", "path": "/?ed=2048", "headers": { "Host": "example.com" } }
    },

    { "tag": "香港节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇭🇰|港|hk|hongkong|hong kong)" },
    { "tag": "台湾节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇹🇼|台|tw|taiwan|tai wan)" },
    { "tag": "日本节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇯🇵|日|jp|japan)" },
    { "tag": "新加坡节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇸🇬|新|sg|singapore)" },
    { "tag": "美国节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇺🇸|美|us|unitedstates|united states)" },
    { "tag": "免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ],
  "route": {
    "default_domain_resolver": "dns_direct",
    "rules": [
      { "rule_set": [ "telegramip" ], "invert": true, "action": "sniff" },
      { "type": "logical", "mode": "or", "rules": [ { "protocol": [ "dns" ] }, { "port": 53 } ], "action": "hijack-dns" },
      { "clash_mode": [ "Direct" ], "outbound": "DIRECT" },
      { "clash_mode": [ "Global" ], "outbound": "GLOBAL" },
      { "rule_set": [ "private" ], "outbound": "私有网络" },
      { "rule_set": [ "ads" ], "outbound": "广告域名" },
      { "rule_set": [ "applications" ], "outbound": "直连软件" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "游戏服务" },
      { "rule_set": [ "games" ], "outbound": "游戏平台" },
      { "rule_set": [ "ai" ], "outbound": "AI 平台" },
      { "rule_set": [ "networktest" ], "outbound": "网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "国外域名" },
      { "rule_set": [ "cn" ], "outbound": "国内域名" },
      { "ip_is_private": true, "outbound": "私有网络" },
      { "rule_set": [ "telegramip" ], "outbound": "电报消息" },
      { "action": "resolve", "match_only": true },
      { "rule_set": [ "cnip" ], "outbound": "国内 IP" }
    ],
    "rule_set": [
      {
        "tag": "trackerslist",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/trackerslist.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/trackerslist.srs"
      },
      {
        "tag": "ads",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ads.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/ads.srs"
      },
      {
        "tag": "private",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/private.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/private.srs"
      },
      {
        "tag": "applications",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/applications.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/applications.srs"
      },
      {
        "tag": "microsoft-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/microsoft-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/microsoft-cn.srs"
      },
      {
        "tag": "apple-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/apple-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/apple-cn.srs"
      },
      {
        "tag": "google-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/google-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/google-cn.srs"
      },
      {
        "tag": "games-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/games-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/games-cn.srs"
      },
      {
        "tag": "games",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/games.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/games.srs"
      },
      {
        "tag": "ai",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ai.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/ai.srs"
      },
      {
        "tag": "networktest",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/networktest.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/networktest.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/proxy.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cn.srs"
      },
      {
        "tag": "telegramip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/telegramip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/telegramip.srs"
      },
      {
        "tag": "cnip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cnip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cnip.srs"
      }
    ],
    "final": "漏网之鱼",
    "auto_detect_interface": true
  },
  "experimental": {
    "cache_file": {
      "enabled": true,
      "store_fakeip": true
    },
    "clash_api": {
      "external_controller": "127.0.0.1:9999",
      "external_ui": "ui",
      "external_ui_download_url": "https://github.com/Zephyruso/zashboard/archive/gh-pages.zip",
      "secret": ""
    },
    "urltest_unified_delay": true
  }
}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- ① 本 `dns` 配置中，国外域名走 `fakeip`，国内域名走国内 DNS 解析，未知域名在匹配 `rule_set:cnip` 规则时会先由国外 DNS 解析且配置 `client_subnet` 提高了兼容性，解析出 IP 在国内则走国内 DNS 解析且走 `国内 IP` 规则，否则走 `fakeip` 且走 `漏网之鱼` 规则（有效解决了“心理 DNS 泄露问题”，详见《[搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxr-ruleset/)》）
- ② 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_hosts",
        "type": "hosts",
        "predefined": {
          "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
          "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ]
        }
      },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "hosts" },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_quad9", "type": "https", "server": "dns11.quad9.net", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_alidns", "dns_dnspod" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_quad9" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "dns_hosts" },
      { "clash_mode": [ "Direct" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "action": "predefined" },
      { "rule_set": [ "trackerslist", "microsoft-cn", "apple-cn", "google-cn", "games-cn" ], "server": "dns_direct" },
      { "rule_set": [ "games", "ai", "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "private", "cn" ], "server": "dns_direct" },
      // 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
      { "action": "evaluate", "server": "dns_proxy", "client_subnet": "211.137.58.0/24" },
      { "match_response": true, "rule_set": [ "cnip" ], "server": "dns_direct" },
      { "match_response": true, "ip_accept_any": true, "invert": true, "action": "respond" },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "optimistic": true,
    "reverse_mapping": true
  }
}
```

---

>`outbounds` 私货
{: .prompt-tip }

注：
- ① 本 `outbounds` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `"type": "urltest"` 进行延迟测试（可进入 [zashboard 面板](https://github.com/Zephyruso/zashboard) → 代理 → 设置 → 管理隐藏代理组，设置隐藏以简化 Dashboard 面板中的显示）。再将延迟测试最低的策略组配置 `"type": "loadbalance"` 进行负载均衡供用户选择使用
- ② 将不同的优选节点分别配置 `"fallback": { "enabled": true }` 进行故障转移（可进入 zashboard 面板 → 代理 → 设置 → 管理隐藏代理组，设置隐藏以简化 Dashboard 面板中的显示）。再将故障转移后的策略组配置 `"type": "urltest"` 进行延迟测试供用户选择使用

```json
{
  "outbounds": [
    { "tag": "香港节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "香港-ss", "香港-trojan" ] },
    { "tag": "香港-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)((🇭🇰|港|hk|hongkong|hong kong).*ss)" },
    { "tag": "香港-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇭🇰|港|hk|hongkong|hong kong)", "exclude": "(?i)(ss)" },
    { "tag": "台湾节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "台湾-ss", "台湾-trojan" ] },
    { "tag": "台湾-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)((🇹🇼|台|tw|taiwan|tai wan).*ss)" },
    { "tag": "台湾-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇹🇼|台|tw|taiwan|tai wan)", "exclude": "(?i)(ss)" },
    { "tag": "日本节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "日本-ss", "日本-trojan" ] },
    { "tag": "日本-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)((🇯🇵|日|jp|japan).*ss)" },
    { "tag": "日本-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇯🇵|日|jp|japan)", "exclude": "(?i)(ss)" },
    { "tag": "新加坡节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "新加坡-ss", "新加坡-trojan" ] },
    { "tag": "新加坡-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)((🇸🇬|新|sg|singapore).*ss)" },
    { "tag": "新加坡-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇸🇬|新|sg|singapore)", "exclude": "(?i)(ss)" },
    { "tag": "美国节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "美国-ss", "美国-trojan" ] },
    { "tag": "美国-ss", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "(?i)((🇺🇸|美|us|unitedstates|united states).*ss)" },
    { "tag": "美国-trojan", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇺🇸|美|us|unitedstates|united states)", "exclude": "(?i)(ss)" },
    { "tag": "免费节点", "type": "urltest", "tolerance": 100, "outbounds": [ "移动优选节点", "CF 优选节点" ] },
    { "tag": "移动优选节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ], "include": "(?i)(cmcc)", "fallback": { "enabled": true, "max_delay": "400ms" } },
    { "tag": "CF 优选节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ], "include": "(?i)(cfip)", "fallback": { "enabled": true, "max_delay": "400ms" } }
  ]
}
```

## 二、 添加以管理员身份运行 Bash 文件的支持
1. 下载安装 [Git for Windows](https://github.com/git-for-windows/git/releases)，安装目录默认为 `C:\Program Files\Git`{: .filepath}
2. 编辑文本文档，粘贴如下内容：

```text
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\.sh]
@="sh_auto_file"

[HKEY_CLASSES_ROOT\sh_auto_file\shell\runas\command]
@="\"C:\\Program Files\\Git\\git-bash.exe\" \"%1\""
```
另存为 .reg 文件，双击导入

## 三、 导入 [sing-box reF1nd 版内核](https://github.com/reF1nd/sing-box)和配置文件并启动 sing-boxr
### 1. 导入内核和配置文件
- ① 编辑本文文档，粘贴如下内容：  
  注：
  - ➊ 将《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-windows-singboxr-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .json 文件直链替换下面命令中的 `{.json 配置文件直链}`
  - ➋ 或删除此条命令，直接进入 `%PROGRAMFILES%\sing-box`{: .filepath} 文件夹，新建 config.json 文件并粘贴配置内容

  ```shell
  #!/bin/bash

  title="安装、更新 sing-boxr 内核和配置文件"

  while true; do
    clear
    echo "==============================================="
    echo "      安装、更新 sing-boxr 内核和配置文件      "
    echo "==============================================="
    echo
    echo "1. 安装（更新）sing-boxr 内核和面板"
    echo "2. 导入（更新）配置文件"
    echo "3. 启动 sing-boxr 服务"
    echo "4. 停止 sing-boxr 服务"
    echo "0. 退出"
    echo "==============================================="
    read -p "请选择操作（0-4）：" choice
    case $choice in
      1)
        echo "安装（更新）sing-boxr 内核和面板..."
        cd "$PROGRAMFILES"
        if [ -f "./sing-box/sing-box.exe" ]; then
          echo "检测到已安装 sing-boxr 内核，是否更新？（Y/n）"
          while true; do
            read -n1 -r choice
            case $choice in
              [Yy])
                echo
                echo "下载 sing-boxr 内核..."
                curl -sS -o "$USERPROFILE/Downloads/sing-box.exe" -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-ref1nd-test-windows-amd64-v3.exe
                echo "下载 sing-boxr 内核成功"

                echo "结束 sing-boxr 相关进程..."
                taskkill //f //t //im "sing-box*"
                echo "结束 sing-boxr 相关进程成功"

                echo "更新 sing-boxr 内核..."
                mv -f "$USERPROFILE/Downloads/sing-box.exe" ./sing-box
                if [ -f "./sing-box/config.json" ]; then
                  echo "更新 sing-boxr 内核成功，是否启动服务？（Y/n）"
                  while true; do
                    read -n1 -r choice
                    case $choice in
                      [Yy])
                        echo
                        echo "启动 sing-boxr 服务..."
                        cd ./sing-box
                        start //min sing-box.exe run
                        read -n1 -r -p "启动 sing-boxr 服务成功，按任意键返回菜单..."
                        break
                        ;;
                      [Nn])
                        break
                        ;;
                      *)
                        echo
                        echo "无效选择，请重新输入！"
                        ;;
                    esac
                  done
                  break
                else
                  echo "更新 sing-boxr 内核成功，请返回菜单导入配置文件！"
                  read -n1 -r -p "按任意键返回菜单..."
                  break
                fi
                ;;
              [Nn])
                break
                ;;
              *)
                echo
                echo "无效选择，请重新输入！"
                ;;
            esac
          done
        else
          echo "检测到未安装 sing-boxr 内核，正在安装..."
          mkdir -p ./sing-box/ui
          curl -sS -o ./sing-box/sing-box.exe -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-ref1nd-test-windows-amd64-v3.exe
          curl -sS -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C ./sing-box/ui
          echo "安装 sing-boxr 内核和面板成功"

          echo "赋予 sing-box 权限..."
          cmd //c "takeown /f sing-box /a /r /d y"
          icacls sing-box /inheritance:r
          icacls sing-box /remove[:g] "TrustedInstaller"
          icacls sing-box /remove[:g] "CREATOR OWNER"
          icacls sing-box /remove[:g] "ALL APPLICATION PACKAGES"
          icacls sing-box /remove[:g] "所有受限制的应用程序包"
          icacls sing-box /grant[:r] "SYSTEM:(OI)(CI)F"
          icacls sing-box /grant[:r] "Administrators:(OI)(CI)F"
          icacls sing-box /grant[:r] "Users:(OI)(CI)F"
          echo "赋予 sing-box 权限成功，请返回菜单导入配置文件！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      2)
        ask_run(){
          while true; do
            read -n1 -r choice
            case $choice in
              [Yy])
                echo
                echo "启动 sing-boxr 服务..."
                cd ./sing-box
                start //min sing-box.exe run
                read -n1 -r -p "启动 sing-boxr 服务成功，按任意键返回菜单..."
                break
                ;;
              [Nn])
                break
                ;;
              *)
                echo
                echo "无效选择，请重新输入！"
                ;;
            esac
          done
        }

        echo "导入（更新）配置文件..."
        cd "$PROGRAMFILES"
        if [ -f "./sing-box/sing-box.exe" ]; then
          if [ -f "./sing-box/config.json" ]; then
            echo "检测到配置文件，是否更新？（Y/n）"
            while true; do
              read -n1 -r choice
              case $choice in
                [Yy])
                  echo
                  echo "下载配置文件..."
                  curl -sS -o "$USERPROFILE/Downloads/config.json" -L https://ghfast.top/{.json 配置文件直链}
                  echo "下载配置文件成功"

                  echo "结束 sing-boxr 相关进程..."
                  taskkill //f //t //im "sing-box*"
                  echo "结束 sing-boxr 相关进程成功"

                  echo "更新配置文件..."
                  mv -f "$USERPROFILE/Downloads/config.json" ./sing-box
                  echo "更新配置文件成功，是否启动服务？（Y/n）"
                  ask_run
                  break
                  ;;
                [Nn])
                  break
                  ;;
                *)
                  echo
                  echo "无效选择，请重新输入！"
                  ;;
              esac
            done
          else
            echo "未检测到配置文件，导入配置文件..."
            curl -sS -o "$USERPROFILE/Downloads/config.json" -L https://ghfast.top/{.json 配置文件直链}
            mv -f "$USERPROFILE/Downloads/config.json" ./sing-box
            echo "导入配置文件成功，是否启动服务？（Y/n）"
            ask_run
          fi
        else
          echo "未检测到 sing-boxr 内核，请返回菜单安装内核！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      3)
        echo "启动 sing-boxr 服务..."
        cd "$PROGRAMFILES"
        if [[ -f "./sing-box/sing-box.exe" && -f "./sing-box/config.json" ]]; then
          cd "./sing-box"
          start //min sing-box.exe run
          read -n1 -r -p "启动 sing-boxr 服务成功，按任意键返回菜单..."
        else
          echo "未检测到 sing-boxr 内核和配置文件，请返回菜单安装内核并导入配置文件！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      4)
        echo "停止 sing-boxr 服务..."
        taskkill //f //t //im "sing-box*"
        read -n1 -r -p "停止 sing-boxr 服务成功，按任意键返回菜单..."
        ;;
      0)
        echo "退出程序"
        exit 0
        ;;
      *)
        echo "无效选择，请重新输入！"
        read -n1 -r -p "按任意键返回菜单..."
        ;;
    esac
  done
  
  ```

- ② 另存为 .sh 文件，右击并选择“以管理员身份运行”

### 2. 启动 sing-boxr
- ① 编辑本文文档，粘贴如下内容：

  ```shell
  cd "%PROGRAMFILES%\sing-box"
  start /min sing-box.exe run
  ```

- ② 另存为 run.bat 文件并复制到 `%PROGRAMFILES%\sing-box`{: .filepath} 文件夹中
- ③ 右击 run.bat 文件并选择“以管理员身份运行”即可  
  小窍门：
  - ➊ 右击 run.bat 文件并选择“发送到桌面快捷方式”
  - ➋ 右击快捷方式并点击“属性” → “高级”，勾选“以管理员身份运行”并“确定”
  - ➌ 若想开机启动 sing-boxr，可搜索“Windows 添加任务计划”教程自行添加

## 四、 访问 Dashboard 面板
.json 文件已配置 zashboard 面板  
打开 <http://miwifi.com:9999/ui/> 后，“端口”输入 `9999`，点击“提交”，即可访问 Dashboard 面板

> 推荐设置
{: .prompt-tip }
1. 进入 zashboard 面板 → 代理 → 代理设置 → 管理隐藏代理组，隐藏不必要显示的代理组
2. 进入 zashboard 面板 → 设置 → 图标，设置“自定义图标”，可参考 [icon 文件](https://github.com/DustinWin/ruleset_geodata/releases/tag/icons)
