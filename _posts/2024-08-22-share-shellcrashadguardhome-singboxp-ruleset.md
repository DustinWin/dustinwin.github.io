---
title: 分享 ShellCrash 搭载 sing-boxp 内核搭配 AdGuard Home 采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxp 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 19:51:07 +0800
categories: [分享配置, Router]
tags: [sing-box, sing-boxp, ShellCrash, AdGuard Home, ruleset, rule_set, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 此方案采用 [ShellCrash](https://github.com/juewuy/ShellCrash) 作为上游，[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 作为下游的模式
2. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
3. 此方案中 ShellCrash 采用了**绕过 CN_IP** 的模式（仍与 AdGuard Home 配合完美）
4. 此方案适用于 ShellCrash（以 arm64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
5. 此方案适用于 AdGuard Home（以 arm64 架构为例，且安装路径为 `/data/AdGuardHome`{: .filepath}）

## 一、 生成配置文件 .json 文件直链
具体方法此处不再赘述，请看《[生成带有自定义出站和规则的 sing-boxp 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxp-ruleset)》，贴一下我使用的配置：

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
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "DIRECT", "type": "direct" },
    { "tag": "GLOBAL", "type": "selector", "outbounds": [ "DIRECT", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
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
      { "rule_set": [ "trackerslist" ], "outbound": "📋 Trackerslist" },
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
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/fakeip-filter-lite.srs"
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

## 二、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)、[zashboard 面板](https://github.com/Zephyruso/zashboard)和 CN_IP 文件
连接 SSH 后执行如下命令：

```shell
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz | tar -zx -C /tmp/
mkdir -p $CRASHDIR/ui/
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
curl -o $CRASHDIR/cn_ip.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv4.txt
curl -o $CRASHDIR/cn_ipv6.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv6.txt
crash
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 5 Sing-Box-Puer 内核

## 三、 编辑 dns.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：  
注：
- ① 由于 ShellCrash 采用的 DNS 模式为 `mix`，**ShellCrash 传给 AdGuard Home 的国外域名对应 IP 为假 IP**，会导致 AdGuard Home 检查更新和下载更新 DNS 黑名单时失败
- ② `dns.fakeip.exclude_rule` 中添加了 AdGuard Home 常用域名，包括：`adguardteam.github.io`（AdGuard Home 自带 DNS 黑名单下载域名）、`adrules.top`（常用广告拦截下载域名）、`anti-ad.net`（常用广告拦截下载域名）和 `static.adtidy.org`（AdGuard Home 检查更新域名），可使这些域名不走 `fakeip`
- ③ 不推荐使用自带更新去更新，推荐《[五](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxp-ruleset/#%E4%BA%94-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中通过定时任务去自动更新（AdGuard Home 程序已被压缩，节省空间）  
<img src="/assets/img/share/update-adguardhome.png" alt="编辑 dns.json 文件" width="60%" />

```json
{
  "dns": {
    "hosts": {
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "doh.pub": [ "1.12.12.12", "1.12.12.21", "120.53.53.53" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ],
      "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
      "services.googleapis.cn": [ "services.googleapis.com" ]
    },
    "servers": [
      { "tag": "dns_direct", "address": [ "https://dns.alidns.com/dns-query", "https://doh.pub/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://cloudflare-dns.com/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
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
  }
}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，未知域名由国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 sing-boxp 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxp-ruleset/)》），且配置 `client_subnet` 提高了兼容性
- 2. 推荐将 `client_subnet` 设置为当前网络的公网 IP 段，如当前网络公网 IP 为 `202.103.17.123`，可设置为 `202.103.17.0/24`（后续维护更新可直接执行命令 `sed -i -E "s/(\"client_subnet\": \")[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/jsons/dns.json`）

```json
{
  "dns": {
    "hosts": {
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "doh.pub": [ "1.12.12.12", "1.12.12.21", "120.53.53.53" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "dns11.quad9.net": [ "9.9.9.11", "149.112.112.11", "2620:fe::11", "2620:fe::fe:11" ],
      "miwifi.com": [ "192.168.31.1", "127.0.0.1" ],
      "services.googleapis.cn": [ "services.googleapis.com" ]
    },
    "servers": [
      { "tag": "dns_direct", "address": [ "quic://dns.alidns.com:853", "https://doh.pub/dns-query" ], "detour": "DIRECT" },
      // 推荐将 `client_subnet` 设置为当前网络的公网 IP 段
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://dns11.quad9.net/dns-query" ], "client_subnet": "202.103.17.0/24" },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
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
      "exclude_rule": { "rule_set": [ "fakeip-filter", "trackerslist", "private", "cn" ] }
    }
  }
}
```

**新增定时任务**  
连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
205#sed -i -E "s/(\"client_subnet\": \")[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/jsons/dns.json >/dev/null 2>&1#更新client_subnet地址
```

---

## 四、 添加定时任务
1. 连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz && $CRASHDIR/start.sh restart >/dev/null 2>&1#更新sing-box_PuerNya版内核
202#curl -o $CRASHDIR/cn_ip.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv4.txt && curl -o $CRASHDIR/cn_ipv6.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
203#rm -rf $CRASHDIR/ui/* && curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/ >/dev/null 2>&1#更新zashboard面板
204#curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_beta_linux_armv8 && /data/AdGuardHome/AdGuardHome -s restart >/dev/null 2>&1#更新AdGuardHome
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `crash`，进入 ShellCrash → 5 配置自动任务 → 1 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-singbox-adguardhome.png" alt="添加定时任务" width="60%" />

## 五、 新建文件夹
连接 SSH 后执行命令 `mkdir -p $CRASHDIR/providers/ $CRASHDIR/ruleset/`  
注：
- ① 因出站提供者 `outbound_providers` 配置的 `path` 路径中含有文件夹 `providers`{: .filepath}，须手动新建此文件夹才能使 .json 订阅文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）
- ② 因规则集 `route.rule_set` 配置的 `path` 路径中含有文件夹 `ruleset`{: .filepath}，须手动新建此文件夹才能使 .srs 规则集文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）

## 六、 ShellCrash 设置
1. 设置可参考《[ShellCrash 搭载 sing-boxp 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxp-ruleset)》，此处只列举配置的不同之处
2. 进入主菜单 → 2 内核功能设置 → 2 切换 DNS 运行模式 → 4 DNS 进阶设置，设置如下：  
<img src="/assets/img/share/close-dns-null.png" alt="设置部分 2" width="60%" />

3. 进入主菜单 → 7 内核进阶设置 → 5 自定义端口及秘钥，设置为 `9090`
4. 进入主菜单 → 6 导入配置文件 → 2 在线获取完整配置文件，粘贴《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxp-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .json 文件直链，启动服务即可

## 七、 安装 AdGuard Home
连接 SSH 后执行如下命令：

```shell
mkdir -p /data/AdGuardHome
curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_beta_linux_armv8
chmod +x /data/AdGuardHome/AdGuardHome
/data/AdGuardHome/AdGuardHome -s install
/data/AdGuardHome/AdGuardHome -s start
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
cat <<EOF >> /data/auto_ssh/auto_ssh.sh

sleep 10s
/data/AdGuardHome/AdGuardHome -s install
/data/AdGuardHome/AdGuardHome -s start
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
EOF
```

## 八、 AdGuard Home 设置
1. 设置可参考《[全网最详细的解锁 SSH ShellCrash 搭载 sing-boxp 内核搭配 AdGuard Home 安装和配置教程/AdGuard Home 配置](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxp/#2-adguard-home-%E9%85%8D%E7%BD%AE)》（可跳过“添加 DNS 重写”的步骤），此处只列举配置的不同之处
2. 进入设置 → DNS 设置，“后备 DNS 服务器”设置为：

```text
quic://dns.alidns.com:853
https://doh.pub/dns-query
```

## 九、 访问 Dashboard 面板
打开 <http://miwifi.com:9090/ui/> 后，“主机”输入 `192.168.31.1`，点击“提交”即可访问 Dashboard 面板  
<img src="/assets/img/share/192-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />
