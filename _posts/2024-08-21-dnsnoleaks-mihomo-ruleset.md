---
title: 搭载 mihomo 内核配置 DNS 不泄露教程-ruleset 方案
description: 此教程搭载 mihomo 内核并使用其特性防止 DNS 泄露，即针对未知域名走国外 DNS 解析
date: 2024-08-21 07:52:58 +0800
categories: [DNS 配置, DNS 防泄漏]
tags: [Clash, mihomo, 进阶, DNS, DNS 泄露]
---

> 说明
{: .prompt-tip }
1. 此方案彻底防止了 DNS 泄露（针对未知域名走国外 DNS 解析，解析出 IP 在国内则走 `🀄️ 直连 IP` 规则，否则走 `🐟 漏网之鱼` 规则），兼容性无法保证，请慎用
2. 本教程以 [ShellCrash](https://github.com/juewuy/ShellCrash) 为例，其它客户端亦可参考
3. 可进入 <https://ipleak.net> 测试 DNS 是否泄露，“DNS Addresses” 栏目下没有中国国旗（因 `ipleak.net` 属未知域名，默认走 `🐟 漏网之鱼` 规则），即代表 DNS 没有发生泄露

## 一、 导入规则集合文件
`rule-providers` 须添加 `fakeip-filter`，如下：

```yaml
rule-providers:
  fakeip-filter:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/fakeip-filter.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/fakeip-filter.mrs"
    interval: 86400

  cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cn.mrs"
    interval: 86400
```

## 二、 ShellCrash 防泄漏配置
进入主菜单 → 2 内核功能设置 → 2 切换 DNS 运行模式 → 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

## 三、 DNS 防泄漏配置
### 1. DNS 模式为 `mix` 并配置 `ecs`（推荐）
连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：
>推荐将 `ecs` 设置为当前网络的公网 IP 段，如当前网络公网 IP 为 `202.103.17.123`，可设置为 `202.103.17.0/24`（后续维护更新可直接执行命令 `sed -i -E "s/(ecs=)[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/yamls/user.yaml`）
{: .prompt-info }

```yaml
hosts:
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  dns11.quad9.net: [9.9.9.11, 149.112.112.11, 2620:fe::11, 2620:fe::fe:11]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 28.0.0.1/8
  enhanced-mode: fake-ip
  fake-ip-filter: ['rule-set:fakeip-filter,cn']
  respect-rules: true
  nameserver:
    ## 推荐将 `ecs` 设置为当前网络的公网 IP 段
    - 'https://dns.google/dns-query#ecs=202.103.17.0/24'
    - 'https://dns11.quad9.net/dns-query#ecs=202.103.17.0/24'
  proxy-server-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  direct-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 2. DNS 模式为 `mix`
连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
hosts:
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  cloudflare-dns.com: [1.1.1.1, 1.0.0.1, 2606:4700:4700::1111, 2606:4700:4700::1001]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 28.0.0.1/8
  enhanced-mode: fake-ip
  fake-ip-filter: ['rule-set:fakeip-filter,cn']
  respect-rules: true
  nameserver:
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  direct-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 3. DNS 模式为 `fake-ip`
- ① 额外编辑配置文件
  在《[生成带有自定义策略组和规则的 mihomo 配置文件直链-ruleset 方案/添加模板](https://proxy-tutorials.dustinwin.top/posts/link-mihomo-ruleset/#%E4%BA%8C-%E6%B7%BB%E5%8A%A0%E6%A8%A1%E6%9D%BF)》编辑 .yaml 配置文件时，将 `rules` 里所有 IP 相关的规则末尾加上 `no-resolve`，即修改为：

  ```yaml
    - RULE-SET,telegramip,📲 电报消息,no-resolve
    - RULE-SET,privateip,🔒 私有网络,no-resolve
    - RULE-SET,cnip,🀄️ 直连 IP,no-resolve
  ```

- ② 连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

  ```yaml
  hosts:
    dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
    doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]

  dns:
    enable: true
    prefer-h3: true
    ipv6: true
    listen: 0.0.0.0:1053
    fake-ip-range: 28.0.0.1/8
    enhanced-mode: fake-ip
    fake-ip-filter: ['rule-set:fakeip-filter']
    nameserver:
      - https://dns.alidns.com/dns-query
      - https://doh.pub/dns-query
  ```

- ③ 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 4. DNS 模式为 `redir-host`
连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
hosts:
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  cloudflare-dns.com: [1.1.1.1, 1.0.0.1, 2606:4700:4700::1111, 2606:4700:4700::1001]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 28.0.0.1/8
  enhanced-mode: fake-ip
  fake-ip-filter: ['+.*']
  respect-rules: true
  nameserver:
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  direct-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
