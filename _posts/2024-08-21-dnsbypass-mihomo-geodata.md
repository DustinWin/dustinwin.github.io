---
title: 搭载 mihomo 内核进行 DNS 分流教程-geodata 方案
description: 此方案适用于 mihomo，搭载 mihomo 内核并使用其特性进行 DNS 分流
date: 2024-08-21 08:48:20 +0800
categories: [DNS 配置, DNS 分流]
tags: [Clash, mihomo, 进阶, DNS, DNS 分流]
---

## 说明
1. 使用 [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
2. 本教程以 ShellCrash 为例，其它客户端亦可参考
3. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，其它域名包括国外域名都走 `fake-ip`，未知域名走国内 DNS 解析，解析出 IP 在国内则走 `🇨🇳 直连 IP` 规则，否则走 `🐟 漏网之鱼` 规则**
4. 部分用户觉得未知域名处理方式导致 DNS 泄露，可以参考《[搭载 mihomo 内核配置 DNS 不泄露教程-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/dnsnoleaks-mihomo-geodata)》

## 一、 导入路由规则文件
geosite.dat 文件须包含 `fakeip-filter` 和 `cn`，推荐导入我定制的[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

## 二、 DNS 分流配置
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式，选择“3 mix混合模式”  
<img src="/assets/img/dns/dns-mix.png" alt="ShellCrash DNS 运行模式设置" width="60%" />

2. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为“null”  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash DNS 进阶设置" width="60%" />

3. 连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
hosts:
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['geosite:fakeip-filter,cn']
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  direct-nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
