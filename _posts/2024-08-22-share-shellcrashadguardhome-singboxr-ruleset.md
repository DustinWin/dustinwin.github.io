---
title: 分享 ShellCrash 搭载 sing-boxr 内核搭配 AdGuard Home 采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxr 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 19:51:07 +0800
categories: [分享配置, Router]
tags: [sing-box, sing-boxr, ShellCrash, AdGuard Home, ruleset, rule_set, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 此方案采用 [ShellCrash](https://github.com/juewuy/ShellCrash) 作为上游，[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 作为下游的模式
2. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
3. 此方案中 ShellCrash 采用了**绕过 CN_IP** 的模式（仍与 AdGuard Home 配合完美）
4. 此方案适用于 ShellCrash（以 arm64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
5. 此方案适用于 AdGuard Home（以 arm64 架构为例，且安装路径为 `/data/AdGuardHome`{: .filepath}）
6. 此方案不建议启用 ShellCrash 配置脚本 → 2) 功能设置 → 3) 透明路由流量过滤 → 2) 过滤局域网设备，因不经过内核的设备在访问 `漏网之鱼` 域名时会遇到无法访问的情况
7. 本人将路由器设置了每天早上 6 点重启，使得《[五](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxr-ruleset/#%E4%BA%94-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中设置的定时任务生效

## 一、 生成配置文件 .json 文件直链
具体方法此处不再赘述，请看《[生成带有自定义出站和规则的 sing-boxr 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxr-ruleset)》，贴一下我使用的配置：

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
  "outbounds": [
    { "tag": "节点选择", "type": "selector", "outbounds": [ "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "网络测试", "type": "selector", "outbounds": [ "全球直连", "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "AI 平台", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "Trackerslist", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "游戏服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "微软服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "谷歌服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "苹果服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "直连域名", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "直连 IP", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "代理域名", "type": "selector", "outbounds": [ "节点选择", "全球直连" ] },
    { "tag": "电报消息", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "私有网络", "type": "selector", "outbounds": [ "全球直连" ] },
    { "tag": "漏网之鱼", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点", "全球直连" ] },
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
    "rules": [
      { "rule_set": [ "private" ], "outbound": "私有网络" },
      { "rule_set": [ "trackerslist" ], "outbound": "Trackerslist" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "游戏服务" },
      { "rule_set": [ "ai" ], "outbound": "AI 平台" },
      { "rule_set": [ "networktest" ], "outbound": "网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "代理域名" },
      { "rule_set": [ "cn" ], "outbound": "直连域名" },
      { "rule_set": [ "privateip" ], "outbound": "私有网络" },
      { "rule_set": [ "telegramip" ], "outbound": "电报消息" },
      { "action": "resolve", "match_only": true },
      { "rule_set": [ "cnip" ], "outbound": "直连 IP" }
    ],
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/fakeip-filter-lite.srs"
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
        "tag": "privateip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/privateip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/privateip.srs"
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
  }
}
```

---

>`outbounds` 私货
{: .prompt-tip }

注：
- 1. 本 `outbounds` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `"type": "urltest"` 进行延迟测试（可进入 [zashboard 面板](https://github.com/Zephyruso/zashboard) → 代理 → 设置 → 管理隐藏代理组，设置隐藏以简化 Dashboard 面板中的显示）
- 2. 再将上述延迟测试最低的出站配置 `fallback` 进行自动回退

```json
{
  "outbounds": [
    { "tag": "香港节点", "type": "urltest", "outbounds": [ "香港-ss", "香港-trojan" ], "fallback": { "enabled": true, "max_delay": "200ms" } },
    { "tag": "香港-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇭🇰.*ss)" },
    { "tag": "香港-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇭🇰", "exclude": "(?i)(ss)" },
    { "tag": "台湾节点", "type": "urltest", "outbounds": [ "台湾-ss", "台湾-trojan" ], "fallback": { "enabled": true, "max_delay": "200ms" } },
    { "tag": "台湾-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇹🇼.*ss)" },
    { "tag": "台湾-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇹🇼", "exclude": "(?i)(ss)" },
    { "tag": "日本节点", "type": "urltest", "outbounds": [ "日本-ss", "日本-trojan" ], "fallback": { "enabled": true, "max_delay": "200ms" } },
    { "tag": "日本-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇯🇵.*ss)" },
    { "tag": "日本-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇯🇵", "exclude": "(?i)(ss)" },
    { "tag": "新加坡节点", "type": "urltest", "outbounds": [ "新加坡-ss", "新加坡-trojan" ], "fallback": { "enabled": true, "max_delay": "200ms" } },
    { "tag": "新加坡-ss", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇸🇬.*ss)" },
    { "tag": "新加坡-trojan", "type": "urltest", "providers": [ "🛫 机场订阅" ], "include": "🇸🇬", "exclude": "(?i)(ss)" },
    { "tag": "美国节点", "type": "urltest", "outbounds": [ "美国-ss", "美国-trojan" ], "fallback": { "enabled": true, "max_delay": "400ms" } },
    { "tag": "美国-ss", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "(?i)(🇺🇸.*ss)" },
    { "tag": "美国-trojan", "type": "urltest", "tolerance": 100, "providers": [ "🛫 机场订阅" ], "include": "🇺🇸", "exclude": "(?i)(ss)" },
    { "tag": "免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ]
}
```

---

## 二、 导入 [sing-box reF1nd 版内核](https://github.com/reF1nd/sing-box)、zashboard 面板和 CN_IP 文件
连接 SSH 后执行如下命令：

```shell
curl -o /tmp/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-ref1nd-dev-linux-armv8.upx
mkdir -p $CRASHDIR/ui/
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
curl -o $CRASHDIR/cn_ip.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv4.txt
curl -o $CRASHDIR/cn_ipv6.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv6.txt
sc
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 5 Sing-Box-reF1nd 内核

## 三、 编辑 dns.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：  
注：
- ① 由于 ShellCrash 采用的 DNS 模式为 `mix`，**ShellCrash 传给 AdGuard Home 的国外域名对应 IP 为假 IP**，会导致 AdGuard Home 检查更新和下载更新 DNS 黑名单时失败
- ② `fakeip-filter` 中添加了 AdGuard Home 常用域名，包括：`adguardteam.github.io`（AdGuard Home 自带 DNS 黑名单下载域名）、`adrules.top`（常用广告拦截下载域名）、`anti-ad.net`（常用广告拦截下载域名）和 `static.adtidy.org`（AdGuard Home 检查更新域名）
- ③ 不推荐使用自带更新去更新，推荐《[五](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxr-ruleset/#%E4%BA%94-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中通过定时任务去自动更新（AdGuard Home 程序已被压缩，节省空间）  
<img src="/assets/img/share/update-adguardhome.png" alt="编辑 dns.json 文件" width="60%" />

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
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ]
        }
      },
      { "tag": "dns_resolver", "type": "local" },
      { "tag": "dns_direct", "type": "https", "server": "doh.pub", "domain_resolver": "dns_hosts" },
      { "tag": "dns_proxy", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "dns_hosts" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "domain": [ "services.googleapis.cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "fakeip-filter", "trackerslist", "private", "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true
  }
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
          "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ]
        }
      },
      { "tag": "dns_resolver", "type": "local" },
      { "tag": "dns_direct", "type": "https", "server": "doh.pub", "domain_resolver": "dns_hosts" },
      { "tag": "dns_proxy", "type": "https", "server": "dns.google", "domain_resolver": "dns_hosts", "detour": "GLOBAL" },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "ip_accept_any": true, "server": "dns_hosts" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "fakeip-filter", "microsoft-cn", "apple-cn", "google-cn", "games-cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
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

## 四、 编辑 experimental.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/experimental.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "experimental": {
    "cache_file": {
      "enabled": true,
      "store_fakeip": true
    },
    "clash_api": {
      "external_controller": "192.168.31.1:9999",
      "external_ui": "ui",
      "external_ui_download_url": "https://github.com/Zephyruso/zashboard/archive/gh-pages.zip",
      "secret": ""
    }
  }
}
```

## 五、 添加定时任务
1. 连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o $CRASHDIR/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-ref1nd-dev-linux-armv8.upx >/dev/null 2>&1#更新sing-boxr内核
202#curl -o $CRASHDIR/cn_ip.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv4.txt && curl -o $CRASHDIR/cn_ipv6.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
203#curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_beta_linux_armv8 >/dev/null 2>&1#更新AdGuardHome
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `sc`，进入 ShellCrash 配置脚本 → 5) 自动任务 → 1) 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-singboxr-adguardhome.png" alt="添加定时任务" width="60%" />

## 六、 ShellCrash 设置
1. 设置可参考《[ShellCrash 搭载 sing-boxr 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxr-ruleset)》，此处只列举配置的不同之处
2. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 7 DNS 劫持端口，设置为“5353”（须完成《[七](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxr-ruleset/#%E4%B8%83-%E5%AE%89%E8%A3%85-adguard-home)》后才可设置）
3. 进入 2) DNS 设置 → 9) 修改 DNS 服务器，设置如下：  
<img src="/assets/img/dns/dns-null.png" alt="设置部分 2" width="60%" />

4. 进入主菜单 → 6) 配置文件管理 → a) 添加提供者 → 1) 设置名称或代号，如输入“sing-boxr”；后进入 2) 设置链接或路径，粘贴《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrashadguardhome-singboxr-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-json-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的 .yaml 配置文件直链，选择“a) 保存此提供者”
5. 进入 6) 配置文件管理 → c) 在线生成配置文件 → 6) 自定义浏览器 UA，选择“2) 不使用 UA”
6. 进入 6) 配置文件管理 → 1) sing-boxr，选择选择“e) 在线获取此配置文件”，启动服务即可

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
设置可参考《[全网最详细的解锁 SSH ShellCrash 搭载 sing-boxr 内核搭配 AdGuard Home 安装和配置教程/AdGuard Home 配置](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#2-adguard-home-%E9%85%8D%E7%BD%AE)》

## 九、 访问 Dashboard 面板
打开 <http://miwifi.com:9999/ui/> 后，“主机”输入 `192.168.31.1`，“端口”输入 `9999`，点击“提交”即可访问 Dashboard 面板

> 推荐设置
{: .prompt-tip }
1. 进入 zashboard 面板 → 代理 → 代理设置 → 管理隐藏代理组，隐藏不必要显示的代理组
2. 进入 zashboard 面板 → 设置 → 图标，设置“自定义图标”，可参考 [icon 文件](https://github.com/DustinWin/ruleset_geodata/releases/tag/icons)
