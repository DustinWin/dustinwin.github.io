---
title: 分享 mihomo for Windows 采用 ruleset 方案的一套配置
description: 此配置搭载 mihomo 内核，采用 <code>RULE-SET</code> 规则搭配 .list 和 .mrs 规则集合文件
date: 2024-12-04 02:30:00 +0800
categories: [分享配置, Windows]
tags: [Clash, mihomo, Windows, ruleset, rule-set, 分享]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案采用**裸核**的方式运行，更加精简
3. 我所在地区的中国移动网络使用[阿里云公共 DNS](https://help.aliyun.com/zh/dns/what-is-alibaba-cloud-public-dns) 且 `RULE-SET,google-cn,谷歌服务` 走直连时会有异常情况出现（如 [Google Chrome](https://www.google.com/chrome/) 检查更新失败），所以使用[系统 DNS](https://wiki.metacubex.one/config/dns/type/#system) 来代替

## 一、 生成配置文件 .yaml 文件直链
具体方法请参考《[生成带有自定义策略组和规则的 mihomo 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-mihomo-ruleset)》，贴一下我使用的配置：

```yaml
proxy-providers:
  🛫 机场订阅:
    type: http
    # 修改为你的 Clash 订阅链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport.yaml
    interval: 86400
    filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600

  🆓 免费订阅:
    type: http
    # 修改为你的 Clash 订阅链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/free.yaml
    interval: 43200
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600

log-level: error
ipv6: true
allow-lan: true
mixed-port: 7890
unified-delay: true
tcp-concurrent: true
external-controller: 127.0.0.1:9999
external-ui: ui
external-ui-url: "https://github.com/Zephyruso/zashboard/archive/gh-pages.zip"
profile: {store-selected: true, store-fake-ip: true}

sniffer:
  enable: true
  parse-pure-ip: true
  sniff: {HTTP: {ports: [80, 8080-8880], override-destination: true}, TLS: {ports: [443, 8443]}, QUIC: {ports: [443, 8443]}}
  skip-domain: ['Mijia Cloud']

tun:
  enable: true
  stack: mixed
  dns-hijack: [any:53]
  auto-route: true
  auto-detect-interface: true
  device: mihomo
  strict-route: true

hosts:
  miwifi.com: [192.168.31.1, 127.0.0.1]
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]

dns:
  enable: true
  prefer-h3: true
  ipv6: true
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter: ['rule-set:private,trackerslist,cn']
  nameserver:
    - https://doh.pub/dns-query
    - system
  nameserver-policy: {'rule-set:ads': [rcode://success]}

# 若没有单个出站代理节点，须删除所有 `🆚 vless 节点` 相关内容
proxies:
  - name: 🆚 vless 节点
    type: vless
    server: example.com
    port: 443
    uuid: {uuid}
    network: ws
    tls: true
    udp: false
    sni: example.com
    client-fingerprint: chrome
    ws-opts:
      path: "/?ed=2048"
      headers:
        host: example.com

proxy-groups:
  - {name: 节点选择, type: select, proxies: [香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/proxy.png"}
  - {name: 网络测试, type: select, proxies: [全球直连, 节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/networktest.png"}
  - {name: 游戏平台, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/games.png"}
  - {name: AI 平台, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ai.png"}
  - {name: 游戏服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/games-cn.png"}
  - {name: 微软服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/microsoft-cn.png"}
  - {name: 谷歌服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/google-cn.png"}
  - {name: 苹果服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/apple-cn.png"}
  - {name: 国内域名, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cn.png"}
  - {name: 国内 IP, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cnip.png"}
  - {name: 国外域名, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/global.png"}
  - {name: 电报消息, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/telegram.png"}
  - {name: 直连软件, type: select, proxies: [全球直连], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/applications.png"}
  - {name: 私有网络, type: select, proxies: [全球直连], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/private.png"}
  # 若机场的 UDP 质量不是很好，导致某游戏无法登录或进入房间，可以添加 `disable-udp: true` 配置项解决
  - {name: 漏网之鱼, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点, 全球直连], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/match.png"}
  - {name: 广告域名, type: select, proxies: [全球拦截, 全球绕过], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ads.png"}
  - {name: 全球拦截, type: select, proxies: [REJECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/reject.png"}
  - {name: 全球绕过, type: select, proxies: [PASS], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/pass.png"}
  - {name: 全球直连, type: select, proxies: [DIRECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/direct.png"}

  - {name: 香港节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 台湾节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 日本节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 新加坡节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 美国节点, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 免费节点, type: url-test, tolerance: 100, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}

rule-providers:
  ads:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/ads.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ads.mrs"
    interval: 86400

  private:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/private.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/private.mrs"
    interval: 86400

  trackerslist:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/trackerslist.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/trackerslist.mrs"
    interval: 86400

  applications:
    type: http
    behavior: classical
    format: text
    path: ./ruleset/applications.list
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/applications.list"
    interval: 86400

  microsoft-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/microsoft-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/microsoft-cn.mrs"
    interval: 86400

  apple-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/apple-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/apple-cn.mrs"
    interval: 86400

  google-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/google-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/google-cn.mrs"
    interval: 86400

  games-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/games-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/games-cn.mrs"
    interval: 86400

  games:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/games.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/games.mrs"
    interval: 86400

  ai:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/ai.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ai.mrs"
    interval: 86400

  networktest:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/networktest.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/networktest.mrs"
    interval: 86400

  proxy:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/proxy.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/proxy.mrs"
    interval: 86400

  cn:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cn.mrs"
    interval: 86400

  privateip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/privateip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/privateip.mrs"
    interval: 86400

  cnip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/cnip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cnip.mrs"
    interval: 86400

  telegramip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/telegramip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/telegramip.mrs"
    interval: 86400

rules:
  - RULE-SET,private,私有网络
  - RULE-SET,ads,广告域名
  - RULE-SET,applications,直连软件
  - RULE-SET,microsoft-cn,微软服务
  - RULE-SET,apple-cn,苹果服务
  - RULE-SET,google-cn,谷歌服务
  - RULE-SET,games-cn,游戏服务
  - RULE-SET,games,游戏平台
  - RULE-SET,ai,AI 平台
  - RULE-SET,networktest,网络测试
  - RULE-SET,proxy,国外域名
  - RULE-SET,cn,国内域名
  - RULE-SET,privateip,私有网络,no-resolve
  - RULE-SET,cnip,国内 IP
  - RULE-SET,telegramip,电报消息,no-resolve
  - MATCH,漏网之鱼
```

---

>`proxy-groups` 私货
{: .prompt-tip }

注：
- 1. 本 `proxy-groups` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `type: url-test` 进行延迟测试，且配置 `hidden: true` 以简化 Dashboard 面板中的显示。再将延迟测试最低的策略组配置 `type: load-balance` 进行负载均衡
- 2. 将不同的优选节点分别配置 `type: fallback` 进行故障转移，且配置 `hidden: true` 以简化 Dashboard 面板中的显示。再将故障转移后的策略组配置 `type: url-test` 进行延迟测试

```yaml
proxy-groups:
  - {name: 香港节点, type: load-balance, strategy: consistent-hashing, proxies: [香港-ss, 香港-trojan], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 香港-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)((🇭🇰|港|hk|hongkong|hong kong).*ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 香港-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", exclude-filter: "(?i)(ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 台湾节点, type: load-balance, strategy: consistent-hashing, proxies: [台湾-ss, 台湾-trojan], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 台湾-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)((🇹🇼|台|tw|taiwan|tai wan).*ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 台湾-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", exclude-filter: "(?i)(ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 日本节点, type: load-balance, strategy: consistent-hashing, proxies: [日本-ss, 日本-trojan], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 日本-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)((🇯🇵|日|jp|japan).*ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 日本-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", exclude-filter: "(?i)(ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 新加坡节点, type: load-balance, strategy: consistent-hashing, proxies: [新加坡-ss, 新加坡-trojan], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 新加坡-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)((🇸🇬|新|sg|singapore).*ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 新加坡-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", exclude-filter: "(?i)(ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 美国节点, type: load-balance, strategy: consistent-hashing, proxies: [美国-ss, 美国-trojan], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 美国-ss, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)((🇺🇸|美|us|unitedstates|united states).*ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 美国-trojan, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", exclude-filter: "(?i)(ss)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 免费节点, type: url-test, tolerance: 100, proxies: [移动优选节点, CF 优选节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}
  - {name: 移动优选节点, type: fallback, use: [🆓 免费订阅], filter: "(?i)(cmcc)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cmcc.png"}
  - {name: CF 优选节点, type: fallback, use: [🆓 免费订阅], filter: "(?i)(cfip)", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cfip.png"}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，仅国外域名 `proxy` 走 `fake-ip`，国内域名 `cn` 走国内 DNS 解析，未知域名走国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 mihomo 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-mihomo-ruleset/)》），且配置 `ecs` 提高了兼容性
- 2. 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`
- 3. 记得删除 `rule-providers.trackerslist`

```yaml
hosts:
  miwifi.com: [192.168.31.1, 127.0.0.1]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  dns11.quad9.net: [9.9.9.11, 149.112.112.11, 2620:fe::11, 2620:fe::fe:11]

dns:
  enable: true
  ipv6: true
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter-mode: whitelist
  fake-ip-filter: ['rule-set:games,ai,proxy']
  respect-rules: true
  nameserver:
    # 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    - 'https://dns.google/dns-query#ecs=211.137.58.0/24'
    - 'https://dns11.quad9.net/dns-query#ecs=211.137.58.0/24'
  proxy-server-nameserver:
    - quic://dns.alidns.com:853
    - https://doh.pub/dns-query
  direct-nameserver:
    - https://doh.pub/dns-query
    - system
  nameserver-policy:
    'rule-set:ads': [rcode://success]
    'rule-set:private,microsoft-cn,apple-cn,google-cn,games-cn,cn':
      - quic://dns.alidns.com:853
      - https://doh.pub/dns-query
```

----

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

## 三、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和配置文件并启动 mihomo
### 1. 导入内核和配置文件
- ① 编辑本文文档，粘贴如下内容：  
  注：
  - ➊ 将《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-windows-mihomo-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-yaml-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .yaml 文件直链替换下面命令中的 `{.yaml 配置文件直链}`
  - ➋ 或者删除此条命令，直接进入 `%PROGRAMFILES%\mihomo\profiles`{: .filepath} 文件夹，新建 config.yaml 文件并粘贴配置内容

  ```shell
  #!/bin/bash

  title="安装、更新 mihomo 内核和配置文件"

  while true; do
    clear
    echo "==============================================="
    echo "        安装、更新 mihomo 内核和配置文件       "
    echo "==============================================="
    echo
    echo "1. 安装（更新）mihomo 内核和面板"
    echo "2. 导入（更新）配置文件"
    echo "3. 启动 mihomo 服务"
    echo "4. 停止 mihomo 服务"
    echo "0. 退出"
    echo "==============================================="
    read -p "请选择操作（0-4）：" choice
    case $choice in
      1)
        echo "安装（更新）mihomo 内核和面板..."
        cd "$PROGRAMFILES"
        if [ -f "./mihomo/mihomo.exe" ]; then
          echo "检测到已安装 mihomo 内核，是否更新？（Y/n）"
          while true; do
            read -r choice
            case $choice in
              Y|y)
                echo "下载 mihomo 内核..."
                curl -o "$USERPROFILE/Downloads/mihomo.exe" -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-windows-amd64-v3.exe
                echo "下载 mihomo 内核成功"

                echo "结束 mihomo 相关进程..."
                taskkill //f //t //im "mihomo*"
                echo "结束 mihomo 相关进程成功"

                echo "更新 mihomo 内核..."
                mv -f "$USERPROFILE/Downloads/mihomo.exe" ./mihomo
                if [ -f "./mihomo/profiles/config.yaml" ]; then
                  echo "更新 mihomo 内核成功，是否启动服务？（Y/n）"
                  while true; do
                    read -r choice
                    case $choice in
                      Y|y)
                        echo "启动 mihomo 服务..."
                        cd ./mihomo
                        start //min mihomo.exe run
                        read -n1 -r -p "启动 mihomo 服务成功，按任意键返回菜单..."
                        break
                        ;;
                      N|n)
                        read -n1 -r -p "取消启动服务，按任意键返回菜单..."
                        break
                        ;;
                      *)
                        echo "无效选择，请重新输入！"
                        ;;
                    esac
                  done
                  break
                else
                  echo "更新 mihomo 内核成功，请返回菜单导入配置文件！"
                  read -n1 -r -p "按任意键返回菜单..."
                  break
                fi
                ;;
              N|n)
                read -n1 -r -p "取消更新内核，按任意键返回菜单..."
                break
                ;;
              *)
                echo "无效选择，请重新输入！"
                ;;
            esac
          done
        else
          echo "检测到未安装 mihomo 内核，正在安装..."
          mkdir -p ./mihomo/ui
          curl -o ./mihomo/mihomo.exe -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-windows-amd64-v3.exe
          curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C ./mihomo/ui
          echo "安装 mihomo 内核和面板成功"

          echo "赋予 mihomo 权限..."
          cmd //c "takeown /f mihomo /a /r /d y"
          icacls mihomo /inheritance:r
          icacls mihomo /remove[:g] "TrustedInstaller"
          icacls mihomo /remove[:g] "CREATOR OWNER"
          icacls mihomo /remove[:g] "ALL APPLICATION PACKAGES"
          icacls mihomo /remove[:g] "所有受限制的应用程序包"
          icacls mihomo /grant[:r] "SYSTEM:(OI)(CI)F"
          icacls mihomo /grant[:r] "Administrators:(OI)(CI)F"
          icacls mihomo /grant[:r] "Users:(OI)(CI)F"
          echo "赋予 mihomo 权限成功，请返回菜单导入配置文件！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      2)
        ask_run(){
          while true; do
            read -r choice
            case $choice in
              Y|y)
                echo "启动 mihomo 服务..."
                cd ./mihomo
                start //min mihomo.exe run
                read -n1 -r -p "启动 mihomo 服务成功，按任意键返回菜单..."
                break
                ;;
              N|n)
                read -n1 -r -p "取消启动服务，按任意键返回菜单..."
                break
                ;;
              *)
                echo "无效选择，请重新输入！"
                ;;
            esac
          done
        }

        echo "导入（更新）配置文件..."
        cd "$PROGRAMFILES"
        if [ -f "./mihomo/mihomo.exe" ]; then
          if [ -f "./mihomo/profiles/config.yaml" ]; then
            echo "检测到配置文件，是否更新？（Y/n）"
            while true; do
              read -r choice
              case $choice in
                Y|y)
                  echo "下载配置文件..."
                  curl -o "$USERPROFILE/Downloads/config.yaml" -L https://ghfast.top/{.yaml 配置文件直链}
                  echo "下载配置文件成功"

                  echo "结束 mihomo 相关进程..."
                  taskkill //f //t //im "mihomo*"
                  echo "结束 mihomo 相关进程成功"

                  echo "更新配置文件..."
                  mv -f "$USERPROFILE/Downloads/config.yaml" ./mihomo/profiles
                  echo "更新配置文件成功，是否启动服务？（Y/n）"
                  ask_run
                  break
                  ;;
                N|n)
                  read -n1 -r -p "取消更新配置文件，按任意键返回菜单..."
                  break
                  ;;
                *)
                  echo "无效选择，请重新输入！"
                  ;;
              esac
            done
          else
            echo "未检测到配置文件，导入配置文件..."
            mkdir -p ./mihomo/profiles
            curl -o "$USERPROFILE/Downloads/config.yaml" -L https://ghfast.top/{.yaml 配置文件直链}
            mv -f "$USERPROFILE/Downloads/config.yaml" ./mihomo/profiles
            echo "导入配置文件成功，是否启动服务？（Y/n）"
            ask_run
          fi
        else
          echo "未检测到 mihomo 内核，请返回菜单安装内核！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      3)
        echo "启动 mihomo 服务..."
        cd "$PROGRAMFILES"
        if [[ -f "./mihomo/mihomo.exe" && -f "./mihomo/profiles/config.yaml" ]]; then
          cd "./mihomo"
          start //min mihomo.exe -d profiles
          read -n1 -r -p "启动 mihomo 服务成功，按任意键返回菜单..."
        else
          echo "未检测到 mihomo 内核和配置文件，请返回菜单安装内核并导入配置文件！"
          read -n1 -r -p "按任意键返回菜单..."
        fi
        ;;
      4)
        echo "停止 mihomo 服务..."
        taskkill //f //t //im "mihomo*"
        read -n1 -r -p "停止 mihomo 服务成功，按任意键返回菜单..."
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

### 2. 启动 mihomo
- ① 编辑本文文档，粘贴如下内容：

  ```shell
  cd "%PROGRAMFILES%\mihomo"
  start /min mihomo.exe -d profiles
  ```

- ② 另存为 run.bat 文件并复制到 `%PROGRAMFILES%\mihomo`{: .filepath} 文件夹中
- ③ 右击 run.bat 文件并选择“以管理员身份运行”即可  
  小窍门：
  - ➊ 右击 run.bat 文件并选择“发送到桌面快捷方式”
  - ➋ 右击快捷方式并点击“属性” → “高级”，勾选“以管理员身份运行”并“确定”
  - ➌ 若想开机启动 mihomo，可搜索“Windows 添加任务计划”教程自行添加

## 四、 访问 Dashboard 面板
.yaml 文件已配置 [zashboard 面板](https://github.com/Zephyruso/zashboard)  
打开 <http://miwifi.com:9999/ui/> 后，“端口”输入 `9999`，点击“提交”，即可访问 Dashboard 面板
