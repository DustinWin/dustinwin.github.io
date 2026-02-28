---
title: 搭载 sing-boxr 内核配置 DNS 不泄露教程-ruleset 方案
description: 此教程搭载 sing-boxr 内核并使用其特性防止 DNS 泄露，即针对未知域名走国外 DNS 解析
date: 2024-08-22 16:51:20 +0800
categories: [DNS 配置, DNS 防泄漏]
tags: [sing-box, sing-boxr, ShellCrash, ruleset, rule_set, 进阶, DNS, DNS 泄露]
---

> 说明
{: .prompt-tip }
1. 此方案彻底防止了 DNS 泄露（未知域名在匹配 `route.rules.rule_set:cnip` 规则时会走国外 DNS 解析且配置 `client_subnet`，解析出 IP 在国内则走 `🀄️ 直连 IP` 规则，否则走 `🐟 漏网之鱼` 规则），兼容性高，可放心使用
2. 本教程以 [ShellCrash](https://github.com/juewuy/ShellCrash) 为例，其它客户端亦可参考
3. 本教程搭载 [sing-box 内核 reF1nd-main 版](https://github.com/reF1nd/sing-box/tree/reF1nd-main)（导入内核方法可参考《[ShellCrash 和 AdGuard Home 快速安装教程/导入 mihomo 内核 或 sing-box 内核](https://proxy-tutorials.dustinwin.us.kg/posts/pin-toolsinstall/#%E4%BA%8C-%E5%AF%BC%E5%85%A5-mihomo-%E5%86%85%E6%A0%B8-%E6%88%96-sing-box-%E5%86%85%E6%A0%B8)》）
4. 可进入 <https://ipleak.net> 测试 DNS 是否泄露，“DNS Addresses” 栏目下没有中国国旗（因 `ipleak.net` 属未知域名，默认走 `🐟 漏网之鱼` 规则），即代表 DNS 没有发生泄露

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

## 二、 ShellCrash 防泄漏配置
进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，将“DIRECT-DNS”、“PROXY-DNS”和“DEFAULT-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

## 三、 DNS 防泄漏配置
### 1. DNS 模式为 `mix`（推荐）
- ① 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），修改为如下内容：
  
  >推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`
  {: .prompt-info }

  ```json
  {
    "dns": {
      "servers": [
        {
          "tag": "dns_hosts",
          "type": "hosts",
          "predefined": {
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
        { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
        { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
        { "rule_set": [ "fakeip-filter" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
        { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
        { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 }
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

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
- ② 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
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

### 2. DNS 模式为 `fakeip`（不推荐）
- ① 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），修改为如下内容：
  
  >推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`
  {: .prompt-info }

  ```json
  {
    "dns": {
      "servers": [
        {
          "tag": "dns_hosts",
          "type": "hosts",
          "predefined": {
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
        { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
        { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
        { "rule_set": [ "fakeip-filter" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 },
        { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
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

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
- ② 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
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

### 3. DNS 模式为 `redirhost`
- ① 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），修改为如下内容：
  
  >推荐将 `client_subnet` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`
  {: .prompt-info }

  ```json
  {
    "dns": {
      "servers": [
        {
          "tag": "dns_hosts",
          "type": "hosts",
          "predefined": {
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
        { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
        { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
        { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy", "rewrite_ttl": 1 },
        { "rule_set": [ "fakeip-filter", "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct", "rewrite_ttl": 1 }
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

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
- ② 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
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
