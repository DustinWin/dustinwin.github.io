---
title: 分享 ShellCrash 搭载 sing-boxr 内核采用 ruleset 方案的一套配置
description: 此配置搭载 sing-boxr 内核，采用 <code>rule_set</code> 规则搭配 .srs 规则集文件
date: 2024-08-22 19:45:26 +0800
categories: [分享配置, Router]
tags: [sing-box, sing-boxr, ShellCrash, ruleset, rule_set, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)（以 ARM64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
3. 本方案绕过了 CNIP 且不搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)，在 DNS 层拦截广告
4. 本人将路由器设置了每天早上 6 点重启，使得《[六](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrash-singboxr-ruleset/#%E5%85%AD-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中设置的定时任务生效
5. 本教程搭载 [sing-box 内核 reF1nd-Test 版](https://github.com/reF1nd/sing-box/tree/reF1nd-testing)

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
    { "tag": "游戏平台", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "🆚 vless 节点" ] },
    { "tag": "AI 平台", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "🆚 vless 节点" ] },
    { "tag": "游戏服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "微软服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "谷歌服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "苹果服务", "type": "selector", "outbounds": [ "全球直连", "节点选择" ] },
    { "tag": "国外域名", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
    { "tag": "电报消息", "type": "selector", "outbounds": [ "节点选择", "香港节点", "台湾节点", "日本节点", "新加坡节点", "美国节点", "免费节点", "🆚 vless 节点" ] },
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
      { "rule_set": [ "microsoft-cn" ], "outbound": "微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "游戏服务" },
      { "rule_set": [ "games" ], "outbound": "游戏平台" },
      { "rule_set": [ "ai" ], "outbound": "AI 平台" },
      { "rule_set": [ "networktest" ], "outbound": "网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "国外域名" },
      { "rule_set": [ "telegramip" ], "outbound": "电报消息" },
      { "action": "resolve", "match_only": true },
      { "rule_set": [ "cnip" ], "outbound": "全球直连" }
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

## 二、 导入 [sing-box reF1nd 版内核](https://github.com/reF1nd/sing-box)和 zashboard 面板
连接 SSH 后执行如下命令：

```shell
curl -sS -o /tmp/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-ref1nd-test-linux-arm64.upx
mkdir -p $CRASHDIR/ui/
curl -sS -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
sc
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 5 Sing-Box-reF1nd 内核

## 三、 编辑 dns.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
- 注：推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

```json
{
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
      { "tag": "dns_resolver", "type": "local" },
      { "tag": "dns_alidns", "type": "quic", "server": "dns.alidns.com", "domain_resolver": "hosts" },
      { "tag": "dns_dnspod", "type": "https", "server": "doh.pub", "domain_resolver": "hosts" },
      { "tag": "dns_google", "type": "https", "server": "dns.google", "domain_resolver": "hosts", "detour": "GLOBAL" },
      { "tag": "dns_cloudflare", "type": "https", "server": "cloudflare-dns.com", "domain_resolver": "hosts", "detour": "GLOBAL" },
      { "tag": "dns_direct", "type": "group", "servers": [ "dns_alidns", "dns_dnspod" ] },
      { "tag": "dns_proxy", "type": "group", "servers": [ "dns_google", "dns_cloudflare" ] },
      { "tag": "dns_fakeip", "type": "fakeip", "inet4_range": "28.0.0.0/8", "inet6_range": "fc00::/16" }
    ],
    "rules": [
      { "clash_mode": [ "Direct" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "action": "predefined" },
      { "rule_set": [ "trackerslist", "microsoft-cn", "apple-cn", "google-cn", "games-cn" ], "server": "dns_direct" },
      { "rule_set": [ "games", "ai", "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      { "rule_set": [ "private", "cn" ], "server": "dns_direct" },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "optimistic": true,
    "reverse_mapping": true
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

---

>`DNS` 私货
{: .prompt-tip }

注：
- ① 本 `dns` 配置中，国内域名走国内 DNS 解析，国外域名走 `fakeip`，未知域名也走 `fakeip`，在匹配 `rule_set:cnip` 规则时会先由国外 DNS 解析且配置 `client_subnet` 提高了兼容性，解析出 IP 在国内则走 `国内 IP` 规则，否则走 `漏网之鱼` 规则（有效解决了“心理 DNS 泄露问题”，详见《[搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-singboxr-ruleset/)》）
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
      { "tag": "dns_resolver", "type": "local" },
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
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "optimistic": true,
    "reverse_mapping": true,
    // 推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    "client_subnet": "211.137.58.0/24"
  }
}
```

## 四、 编辑 http_clients 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/experimental.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{ "http_clients": [ { "tag": "detour_proxy", "version": 3, "detour": "GLOBAL" } ] }
```

## 五、 编辑 experimental.json 文件

连接 SSH 后执行命令 `vi $CRASHDIR/jsons/experimental.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "experimental": {
    "cache_file": {
      "enabled": true,
      "store_fakeip": true
    },
    "clash_api": {
      "external_controller": "0.0.0.0:9999",
      "external_ui": "ui",
      "external_ui_download_url": "https://github.com/Zephyruso/zashboard/archive/gh-pages-cdn-fonts.zip",
      "secret": ""
    },
    "urltest_unified_delay": true
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 六、 添加定时任务
可参考《[ShellCrash 搭载 sing-boxr 内核的配置-ruleset 方案/添加定时任务](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxr-ruleset/#%E4%BA%8C-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》

## 七、 设置部分
1. 设置可参考《[ShellCrash 搭载 sing-boxr 内核的配置-ruleset 方案/设置部分](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxr-ruleset/#%E4%B8%89-%E8%AE%BE%E7%BD%AE%E9%83%A8%E5%88%86)》，此处只列举配置的不同之处
2. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，设置如下：  
<img src="/assets/img/dns/dns-null.png" alt="设置部分 2" width="60%" />

3. 连接 SSH 后执行如下命令：
```shell
sed -i ':a;N;$!ba;s/{[[:space:]]*"ip_accept_any": true,[[:space:]]*"server": "hosts"[[:space:]]*}/{ "action": "evaluate", "server": "hosts" },\
      { "match_response": true, "ip_accept_any": true, "action": "respond" }/' "$CRASHDIR/starts/singbox_modify.sh"
sed -i '/#生成experimental.json/i\
  #生成http_clients.json\
  cat >"$TMPDIR"/jsons/http_clients.json <<EOF\
{\
  "http_clients": [\
    {\
      "tag": "detour_proxy",\
      "detour": "GLOBAL"\
    },\
    {\
      "tag": "detour_direct",\
      "detour": "DIRECT"\
    }\
  ]\
}\
EOF
' "$CRASHDIR/starts/singbox_modify.sh"
sed -i 's/log dns ntp certificate experimental/log dns ntp certificate http_clients experimental/' "$CRASHDIR/starts/singbox_modify.sh"
sed -i 's/log dns ntp certificate experimental/log dns ntp certificate http_clients experimental/' "$CRASHDIR/menus/override.sh"
```
## 八、 访问 Dashboard 面板
打开 <http://miwifi.com:9999/ui/> 后，“主机”输入 `192.168.31.1`，“端口”输入 `9999`，点击“提交”即可访问 Dashboard 面板

> 推荐设置
{: .prompt-tip }
1. 进入 zashboard 面板 → 代理 → 代理设置 → 管理隐藏代理组，隐藏不必要显示的代理组
2. 进入 zashboard 面板 → 设置 → 图标，设置“自定义图标”，可参考 [icon 文件](https://github.com/DustinWin/ruleset_geodata/releases/tag/icons)
