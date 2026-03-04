---
title: 分享 sing-boxr for Android 采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxr 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 19:58:51 +0800
categories: [分享配置, Android]
tags: [sing-box, sing-boxr, Android, ruleset, rule_set, 分享]
---

> 声明
{: .prompt-warning }
请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬

## 一、 生成配置文件 .json 文件直链
具体方法请参考《[生成带有自定义出站和规则的 sing-boxr 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxr-ruleset)》，贴一下我使用的配置：

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
        "tag": "dns_hosts",
        "type": "hosts",
        "predefined": {
          "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
          "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
          "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ]
        }
      },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "dns_hosts" },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "dns_hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_quad9", "type": "https", "server": "dns11.quad9.net", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_dnspod", "dns_alidns" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_quad9" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "dns_hosts" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "action": "predefined" },
      { "domain": [ "services.googleapis.cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "private", "trackerslist", "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true
  },
  "inbounds": [
    // 启动服务时如果出现 `tun-in` 报错，可将 `"stack": "mixed"` 修改为 `"stack": "system"`
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
    { "tag": "国外域名", "type": "selector", "outbounds": [ "节点选择", "全球直连" ] },
    { "tag": "电报消息", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "直连软件", "type": "selector", "outbounds": [ "全球直连" ] },
    { "tag": "Trackerslist", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
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

    { "tag": "香港节点", "type": "loadbalance", "strategy": "consistent-hashing", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇭🇰|港|hk|hongkong|hong kong)" },
    { "tag": "台湾节点", "type": "loadbalance", "strategy": "consistent-hashing", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇹🇼|台|tw|taiwan|tai wan)" },
    { "tag": "日本节点", "type": "loadbalance", "strategy": "consistent-hashing", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇯🇵|日|jp|japan)" },
    { "tag": "新加坡节点", "type": "loadbalance", "strategy": "consistent-hashing", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇸🇬|新|sg|singapore)" },
    { "tag": "美国节点", "type": "loadbalance", "strategy": "consistent-hashing", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇺🇸|美|us|unitedstates|united states)" },
    { "tag": "免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ],
  "route": {
    "default_domain_resolver": { "server": "dns_direct" },
    "rules": [
      { "action": "sniff" },
      { "protocol": [ "dns" ], "action": "hijack-dns" },
      { "clash_mode": [ "Direct" ], "outbound": "DIRECT" },
      { "clash_mode": [ "Global" ], "outbound": "GLOBAL" },
      { "rule_set": [ "private" ], "outbound": "私有网络" },
      { "rule_set": [ "ads" ], "outbound": "广告域名" },
      { "rule_set": [ "trackerslist" ], "outbound": "Trackerslist" },
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
        "tag": "trackerslist",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/trackerslist.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/trackerslist.srs"
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
    "auto_detect_interface": true,
    "override_android_vpn": true
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
    }
  }
}
```

---

>`outbounds` 私货
{: .prompt-tip }

注：
- 1. 本 `outbounds` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `"type": "urltest"` 进行延迟测试（可进入 [zashboard 面板](https://github.com/Zephyruso/zashboard) → 代理 → 设置 → 管理隐藏代理组，设置隐藏以简化 Dashboard 面板中的显示）。再将延迟测试最低的策略组配置 `"type": "loadbalance"` 进行负载均衡
- 2. 将不同的优选节点分别配置 `"fallback": { "enabled": true }` 进行故障转移（可进入 zashboard 面板 → 代理 → 设置 → 管理隐藏代理组，设置隐藏以简化 Dashboard 面板中的显示）。再将故障转移后的策略组配置 `"type": "urltest"` 进行延迟测试

```json
{
  "outbounds": [
    { "tag": "香港节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "香港-ss", "香港-trojan" ] },
    { "tag": "香港-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇭🇰.*ss)" },
    { "tag": "香港-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇭🇰", "exclude": "(?i)(ss)" },
    { "tag": "台湾节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "台湾-ss", "台湾-trojan" ] },
    { "tag": "台湾-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇹🇼.*ss)" },
    { "tag": "台湾-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇹🇼", "exclude": "(?i)(ss)" },
    { "tag": "日本节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "日本-ss", "日本-trojan" ] },
    { "tag": "日本-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇯🇵.*ss)" },
    { "tag": "日本-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇯🇵", "exclude": "(?i)(ss)" },
    { "tag": "新加坡节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "新加坡-ss", "新加坡-trojan" ] },
    { "tag": "新加坡-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇸🇬.*ss)" },
    { "tag": "新加坡-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇸🇬", "exclude": "(?i)(ss)" },
    { "tag": "美国节点", "type": "loadbalance", "strategy": "consistent-hashing", "outbounds": [ "美国-ss", "美国-trojan" ] },
    { "tag": "美国-ss", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇺🇸.*ss)" },
    { "tag": "美国-trojan", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "🇺🇸", "exclude": "(?i)(ss)" },
    { "tag": "免费节点", "type": "urltest", "tolerance": 100, "outbounds": [ "移动优选节点", "CF 优选节点" ] },
    { "tag": "移动优选节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ], "include": "(?i)(cmcc)", "fallback": { "enabled": true, "max_delay": "400ms" } },
    { "tag": "CF 优选节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ], "include": "(?i)(cfip)", "fallback": { "enabled": true, "max_delay": "400ms" } }
  ]
}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，国外域名 `proxy` 走 `fake-ip`，私有网络 `private` 和国内域名 `cn` 走国内 DNS 解析，未知域名走国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxr-ruleset/)》），且配置 `client_subnet` 提高了兼容性
- 2. 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_hosts",
        "type": "hosts",
        "predefined": {
          "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
          "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
          "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
          "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ]
        }
      },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "dns_hosts" },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "dns_hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_quad9", "type": "https", "server": "dns11.quad9.net", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_dnspod", "dns_alidns" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_quad9" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "dns_hosts" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "action": "predefined" },
      { "rule_set": [ "microsoft-cn", "apple-cn", "google-cn", "games-cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "rule_set": [ "games", "ai", "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "private", "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true,
    // 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    "client_subnet": "211.137.58.0/24"
  }
}
```

---

## 二、 导入配置文件并启动 sing-boxr
1. 进入 sing-boxr for Android → 仪表 → 新配置 → 手动创建，“类型”选择“远程”，在“URL”处粘贴《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-android-singboxr-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .json 直链，“自动更新间隔”填写 `1440`，最后点击“创建”
2. 进入 sing-boxr for Android → 仪表，点击“▶️”图标即可启动 sing-boxr 服务
- 注：首次启用可能会报错，重试几次即可

## 三、 访问 Dashboard 面板
.json 文件已配置 zashboard 面板  
打开 <http://miwifi.com:9999/ui/> 后，“端口”输入`9999`，点击“提交”，即可访问 Dashboard 面板

> 推荐设置
{: .prompt-tip }
1. 进入 zashboard 面板 → 代理 → 代理设置 → 管理隐藏代理组，隐藏不必要显示的代理组
2. 进入 zashboard 面板 → 设置 → 图标，设置“自定义图标”，可参考 [icon 文件](https://github.com/DustinWin/ruleset_geodata/releases/tag/icons)
