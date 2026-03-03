---
title: 搭载 mihomo 内核进行 DNS 分流教程-ruleset 方案
description: 此教程搭载 mihomo 内核并使用其特性进行 DNS 分流，即指定国内域名 <code>rule-set:cn</code> 走国内 DNS 解析，其它域名包括国外域名都走 <code>fake-ip</code>
date: 2024-08-21 07:52:58 +0800
categories: [DNS 配置, DNS 分流]
tags: [Clash, mihomo, 进阶, DNS, DNS 分流]
---

> 说明
{: .prompt-tip }
1. 使用 [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
2. 本教程以 ShellCrash 为例，其它客户端亦可参考
3. 本教程搭载 [mihomo 内核 Meta 版](https://github.com/MetaCubeX/mihomo/tree/Meta)（导入内核方法可参考《[ShellCrash 和 AdGuard Home 快速安装教程/导入 mihomo 内核 或 sing-box 内核](https://proxy-tutorials.dustinwin.us.kg/posts/pin-toolsinstall/#%E4%BA%8C-%E5%AF%BC%E5%85%A5-mihomo-%E5%86%85%E6%A0%B8-%E6%88%96-sing-box-%E5%86%85%E6%A0%B8)》）
4. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，国外域名走 `fake-ip`**。未知域名走 `real-ip`（在匹配 `rules.RULE-SET:cnip` 规则时会由国内 DNS 解析，解析出 IP 在国内则走 `🀄️ 国内 IP` 规则，否则走 `🐟 漏网之鱼` 规则）
5. 部分用户觉得未知域名处理方式会导致 DNS 泄露，可参考《[搭载 mihomo 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-mihomo-ruleset)》

## 一、 导入规则集合文件
`rule-providers` 须添加 `fakeip-filter`、`cn` 和 `proxy`，如下：

```yaml
rule-providers:
  fakeip-filter:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/fakeip-filter.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/fakeip-filter.mrs"
    interval: 86400

  cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cn.mrs"
    interval: 86400

  proxy:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/proxy.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/proxy.mrs"
    interval: 86400
```

## 二、 DNS 分流配置
1. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，将“DIRECT-DNS”、“PROXY-DNS”和“DEFAULT-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash DNS 进阶设置" width="60%" />

2. 连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
hosts:
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]

dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter-mode: rule
  fake-ip-filter:
    - RULE-SET,fakeip-filter,real-ip
    - RULE-SET,proxy,fake-ip
    - RULE-SET,cn,real-ip  # 此条仅演示，可删除
    - MATCH,real-ip
  nameserver:
    - https://dns.pub/dns-query
    - quic://dns.alidns.com:853
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
