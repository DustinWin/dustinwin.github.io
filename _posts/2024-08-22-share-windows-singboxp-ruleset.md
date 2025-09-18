---
title: 分享 sing-boxp for Windows 采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxp 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 20:01:28 +0800
categories: [分享配置, Windows]
tags: [sing-box, sing-boxp, Windows, ruleset, rule_set, 分享]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案采用**裸核**的方式运行，更加精简

## 一、 生成配置文件 .json 文件直链
具体方法请参考《[生成带有自定义出站和规则的 sing-boxp 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxp-ruleset)》，贴一下我使用的配置：

```json
{
  "outbound_providers": [
    {
      "tag": "🛫 机场订阅",
      "type": "remote",
      // 修改为你的 Clash 订阅链接
      "download_url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport.yaml",
      "download_interval": "24h",
      "download_ua": "clash.meta",
      "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)" ],
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m"
    },
    {
      "tag": "🆓 免费订阅",
      "type": "remote",
      // 修改为你的 sing-box 订阅链接
      "download_url": "https://example.com/xxx/xxx",
      "path": "./providers/free.json",
      "download_interval": "24h",
      "download_ua": "sing-box",
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m"
    }
  ],
  "log": { "level": "error", "timestamp": true },
  "dns": {
    "hosts": {
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "doh.pub": [ "1.12.12.12", "1.12.12.21", "120.53.53.53" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ],
      "miwifi.com": [ "192.168.31.1", "127.0.0.1" ]
    },
    "servers": [
      { "tag": "dns_block", "address": "rcode://success" },
      { "tag": "dns_direct", "address": [ "https://dns.alidns.com/dns-query", "https://doh.pub/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://cloudflare-dns.com/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "server": "dns_block", "disable_cache": true, "rewrite_ttl": 0 },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 },
      { "fallback_rules": [ { "rule_set": [ "cnip" ], "server": "dns_direct" }, { "match_all": true, "server": "dns_fakeip", "rewrite_ttl": 1 } ], "server": "dns_direct", "allow_fallthrough": true }
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
      "exclude_rule": { "rule_set": [ "fakeip-filter", "trackerslist", "private", "cn" ] }
    }
  },
  "inbounds": [
    { "tag": "tun-in", "type": "tun", "interface_name": "sing-box", "address": [ "172.19.0.1/30", "fdfe:dcba:9876::1/126" ], "auto_route": true, "strict_route": true, "stack": "mixed", "sniff": true }
  ],
  "outbounds": [
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "📈 网络测试", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "🤖 AI 平台", "type": "selector", "outbounds": [ "🚀 节点选择", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "📋 Trackerslist", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🎮 游戏服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🪟 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇬 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🛡️ 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🀄️ 直连 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🧱 代理域名", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🚀 节点选择", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "🐟 漏网之鱼", "type": "selector", "outbounds": [ "🚀 节点选择", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点", "🎯 全球直连" ] },
    { "tag": "🛑 广告域名", "type": "selector", "outbounds": [ "🔴 全球拦截", "🎯 全球直连" ] },
    { "tag": "🔴 全球拦截", "type": "selector", "outbounds": [ "REJECT" ] },
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "REJECT", "type": "block" },
    { "tag": "DIRECT", "type": "direct" },
    { "tag": "GLOBAL", "type": "selector", "outbounds": [ "DIRECT", "REJECT", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "dns-out", "type": "dns" },
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

    { "tag": "🇭🇰 香港节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong)" ] },
    { "tag": "🇹🇼 台湾节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇹🇼|台|tw|taiwan|tai wan)" ] },
    { "tag": "🇯🇵 日本节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇯🇵|日|jp|japan)" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇸🇬|新|sg|singapore)" ] },
    { "tag": "🇺🇸 美国节点", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇺🇸|美|us|unitedstates|united states)" ] },
    { "tag": "🆓 免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ],
  "route": {
    "rules": [
      { "protocol": [ "dns" ], "outbound": "dns-out" },
      { "clash_mode": [ "Direct" ], "outbound": "DIRECT" },
      { "clash_mode": [ "Global" ], "outbound": "GLOBAL" },
      { "rule_set": [ "private" ], "outbound": "🎯 全球直连" },
      { "rule_set": [ "ads" ], "outbound": "🛑 广告域名" },
      { "rule_set": [ "trackerslist" ], "outbound": "📋 Trackerslist" },
      { "rule_set": [ "applications" ], "outbound": "🎯 全球直连" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "🪟 微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "🍎 苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "🇬 谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "🎮 游戏服务" },
      { "rule_set": [ "ai" ], "outbound": "🤖 AI 平台" },
      { "rule_set": [ "networktest" ], "outbound": "📈 网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "🧱 代理域名" },
      { "rule_set": [ "cn" ], "outbound": "🛡️ 直连域名" },
      { "rule_set": [ "privateip" ], "outbound": "🎯 全球直连", "skip_resolve": true },
      { "rule_set": [ "cnip" ], "outbound": "🀄️ 直连 IP" },
      { "rule_set": [ "telegramip" ], "outbound": "📲 电报消息", "skip_resolve": true }
    ],
    "rule_set": [
      {
        "tag": "ads",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ads.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/ads.srs"
      },
      {
        "tag": "private",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/private.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/private.srs"
      },
      {
        "tag": "trackerslist",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/trackerslist.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/trackerslist.srs"
      },
      {
        "tag": "applications",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/applications.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/applications.srs"
      },
      {
        "tag": "microsoft-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/microsoft-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/microsoft-cn.srs"
      },
      {
        "tag": "apple-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/apple-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/apple-cn.srs"
      },
      {
        "tag": "google-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/google-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/google-cn.srs"
      },
      {
        "tag": "games-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/games-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/games-cn.srs"
      },
      {
        "tag": "ai",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ai.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/ai.srs"
      },
      {
        "tag": "networktest",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/networktest.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/networktest.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/proxy.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/cn.srs"
      },
      {
        "tag": "privateip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/privateip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/privateip.srs"
      },
      {
        "tag": "cnip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cnip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/cnip.srs"
      },
      {
        "tag": "telegramip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/telegramip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/telegramip.srs"
      }
    ],
    "final": "🐟 漏网之鱼",
    "auto_detect_interface": true,
    "concurrent_dial": true
  },
  "experimental": {
    "cache_file": { "enabled": true },
    "clash_api": {
      "external_controller": "127.0.0.1:9090",
      "external_ui": "ui",
      "external_ui_download_url": "https://github.com/Zephyruso/zashboard/archive/gh-pages.zip",
      "secret": "",
      "default_mode": "Rule"
    }
  }
}
```

---

>`outbounds` 私货
{: .prompt-tip }

注：
- 1. 本 `outbounds` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `"type": "urltest"` 进行延迟测试（推荐在 [zashborad 面板](https://github.com/Zephyruso/zashboard)中将其隐藏）
- 2. 再将上述延迟测试最低的出站配置 `"type": "selector"` 进行手动选择

```json
{
  "outbounds": [
    { "tag": "🇭🇰 香港节点", "type": "selector", "outbounds": [ "🇭🇰 香港-ss", "🇭🇰 香港-trojan" ] },
    { "tag": "🇭🇰 香港-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong)" ], "types": ["shadowsocks"] },
    { "tag": "🇭🇰 香港-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong)" ], "types": ["trojan"] },
    { "tag": "🇹🇼 台湾节点", "type": "selector", "outbounds": [ "🇹🇼 台湾-ss", "🇹🇼 台湾-trojan" ] },
    { "tag": "🇹🇼 台湾-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇹🇼|台|tw|taiwan|tai wan)" ], "types": ["shadowsocks"] },
    { "tag": "🇹🇼 台湾-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇹🇼|台|tw|taiwan|tai wan)" ], "types": ["trojan"] },
    { "tag": "🇯🇵 日本节点", "type": "selector", "outbounds": [ "🇯🇵 日本-ss", "🇯🇵 日本-trojan" ] },
    { "tag": "🇯🇵 日本-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇯🇵|日|jp|japan)" ], "types": ["shadowsocks"] },
    { "tag": "🇯🇵 日本-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇯🇵|日|jp|japan)" ], "types": ["trojan"] },
    { "tag": "🇸🇬 新加坡节点", "type": "selector", "outbounds": [ "🇸🇬 新加坡-ss", "🇸🇬 新加坡-trojan" ] },
    { "tag": "🇸🇬 新加坡-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇸🇬|新|sg|singapore)" ], "types": ["shadowsocks"] },
    { "tag": "🇸🇬 新加坡-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇸🇬|新|sg|singapore)" ], "types": ["trojan"] },
    { "tag": "🇺🇸 美国节点", "type": "selector", "outbounds": [ "🇺🇸 美国-ss", "🇺🇸 美国-trojan" ] },
    { "tag": "🇺🇸 美国-ss", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇺🇸|美|us|unitedstates|united states)" ], "types": ["shadowsocks"] },
    { "tag": "🇺🇸 美国-trojan", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "includes": [ "(?i)(🇺🇸|美|us|unitedstates|united states)" ], "types": ["trojan"] },
    { "tag": "🆓 免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ]
}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，未知域名由国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 sing-boxp 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxp-ruleset/)》），且配置 `client_subnet` 提高了兼容性
- 2. 推荐将 `client_subnet` 设置为当前网络的公网 IP 段，如当前网络公网 IP 为 `202.103.17.123`，可设置为 `202.103.17.0/24`

```json
{
  "dns": {
    "hosts": {
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "doh.pub": [ "1.12.12.12", "1.12.12.21", "120.53.53.53" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ],
      "miwifi.com": [ "192.168.31.1", "127.0.0.1" ]
    },
    "servers": [
      { "tag": "dns_block", "address": "rcode://success" },
      { "tag": "dns_direct", "address": [ "quic://dns.alidns.com:853", "https://doh.pub/dns-query" ], "detour": "DIRECT" },
      // 推荐将 `client_subnet` 设置为当前网络的公网 IP 段
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://dns11.quad9.net/dns-query" ], "client_subnet": "202.103.17.0/24" },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "server": "dns_block", "disable_cache": true, "rewrite_ttl": 0 },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 },
      { "fallback_rules": [ { "rule_set": [ "cnip" ], "server": "dns_direct" }, { "match_all": true, "server": "dns_fakeip", "rewrite_ttl": 1 } ], "server": "dns_proxy", "allow_fallthrough": true }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "lazy_cache": true,
    "reverse_mapping": true,
    "mapping_override": true,
    "fakeip": {
      "enabled": true,
      "inet4_range": "198.18.0.0/15",
      "inet6_range": "fc00::/18",
      "exclude_rule": { "rule_set": [ "trackerslist", "private", "cn" ] }
    }
  }
}
```

---

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

3. 另存为 .reg 文件，双击导入

## 三、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)和配置文件并启动 sing-box
### 1. 导入内核和配置文件
- ① 编辑本文文档，粘贴如下内容：  
  注：
  - ➊ 将《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-windows-singboxp-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .json 文件直链替换下面命令中的 `{.json 配置文件直链}`
  - ➋ 或删除此条命令，直接进入 `%PROGRAMFILES%\sing-box`{: .filepath} 文件夹，新建 config.json 文件并粘贴配置内容

  ```shell
  #!/bin/bash

  echo "导入 sing-box 内核和配置文件..."
  cd "$PROGRAMFILES"
  mkdir -p sing-box/providers sing-box/ruleset sing-box/ui
  curl -o sing-box/sing-box.exe -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-windows-amd64.exe
  curl -o sing-box/config.json -L https://ghfast.top/{.json 配置文件直链}
  sed -i -E "s/(\"client_subnet\": \")[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" sing-box/config.json
  echo "导入 sing-box 内核和配置文件成功"

  echo "安装 Zashboard 面板..."
  curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C sing-box/ui
  echo "安装 Zashboard 面板成功"

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
  echo "赋予 sing-box 权限成功"

  read -p "按任意键退出" -n1 -s
  ```

- ② 另存为 .sh 文件，右击并选择“以管理员身份运行”

### 2. 启动 sing-box
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
  - ➌ 若想开机启动 sing-box，可搜索“Windows 添加任务计划”教程自行添加

## 四、 更新 sing-box PuerNya 版内核和配置文件
编辑本文文档，粘贴如下内容：  
注：
- ① 将《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-windows-singboxp-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .json 文件直链替换下面命令中的 `{.json 配置文件直链}`
- ② 或者删除此条命令，直接进入 `%PROGRAMFILES%\sing-box`{: .filepath} 文件夹，修改 config.json 文件内的配置内容

```shell
#!/bin/bash

echo "下载 sing-box 相关文件..."
cd "$PROGRAMFILES/sing-box"
curl -o "$USERPROFILE/Downloads/sing-box.exe" -L https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-windows-amd64.exe
curl -o "$USERPROFILE/Downloads/config.json" -L {.json 配置文件直链}
echo "下载 sing-box 相关文件成功"

echo "结束 sing-box 相关进程..."
taskkill //f //t //im "sing-box*"
echo "结束 sing-box 相关进程成功"

echo "更新 sing-box 内核和配置文件..."
mv -f "$USERPROFILE/Downloads/sing-box.exe" .
mv -f "$USERPROFILE/Downloads/config.json" .
sed -i -E "s/(\"client_subnet\": \")[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" config.json
echo "更新 sing-box 内核和配置文件成功"

echo "等待 10 秒启动 sing-box 服务..."
sleep 10
start //min sing-box.exe -d profiles
echo "启动 sing-box 服务成功"

read -p "按任意键退出" -n1 -s
```

另存为 .sh 文件，右击并选择“以管理员身份运行”

## 五、 访问 Dashboard 面板
.json 文件已配置 [zashboard 面板](https://github.com/Zephyruso/zashboard)  
打开 <http://miwifi.com:9090/ui/> 后可直接点击“提交”，即可访问 Dashboard 面板  
<img src="/assets/img/share/127-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />
