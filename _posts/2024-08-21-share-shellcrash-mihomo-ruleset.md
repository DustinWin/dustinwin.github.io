---
title: 分享 ShellCrash 搭载 mihomo 内核采用 ruleset 方案的一套配置
description: 此配置搭载 mihomo 内核，采用 <code>RULE-SET</code> 规则搭配 .list 和 .mrs 规则集合文件
date: 2024-08-21 18:18:29 +0800
categories: [分享配置, Router]
tags: [Clash, mihomo, ShellCrash, ruleset, rule-set, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)（以 ARM64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
3. 本方案绕过了 CNIP 且不搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)，在 DNS 层拦截广告
4. 本人将路由器设置了每天早上 6 点重启，使得《[四](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrash-mihomo-ruleset/#%E5%9B%9B-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中设置的定时任务生效

## 一、 生成配置文件 .yaml 文件直链
具体方法此处不再赘述，请看《[生成带有自定义策略组和规则的 mihomo 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-mihomo-ruleset)》，贴一下我使用的配置：

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
    interval: 86400
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600

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
  - {name: 国外域名, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/global.png"}
  - {name: 电报消息, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/telegram.png"}
  # 若机场的 UDP 质量不是很好，导致某游戏无法登录或进入房间，可以添加 `disable-udp: true` 配置项解决
  - {name: 漏网之鱼, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点, 全球直连], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/match.png"}
  - {name: 全球直连, type: select, proxies: [DIRECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/direct.png"}

  - {name: 香港节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 台湾节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 日本节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 新加坡节点, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 美国节点, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 免费节点, type: url-test, tolerance: 100, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}

rule-providers:
  trackerslist:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/trackerslist.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/trackerslist.mrs"
    interval: 86400

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
  - RULE-SET,microsoft-cn,微软服务
  - RULE-SET,apple-cn,苹果服务
  - RULE-SET,google-cn,谷歌服务
  - RULE-SET,games-cn,游戏服务
  - RULE-SET,games,游戏平台
  - RULE-SET,ai,AI 平台
  - RULE-SET,networktest,网络测试
  - RULE-SET,proxy,国外域名
  - RULE-SET,cnip,全球直连
  - RULE-SET,telegramip,电报消息,no-resolve
  - MATCH,漏网之鱼
```

---

>`proxy-groups` 私货
{: .prompt-tip }

注：
- ① 本 `proxy-groups` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `type: url-test` 进行延迟测试，且配置 `hidden: true` 以简化 Dashboard 面板中的显示。再将延迟测试最低的策略组配置 `type: load-balance` 进行负载均衡供用户选择使用
- ② 将不同的优选节点分别配置 `type: fallback` 进行故障转移，且配置 `hidden: true` 以简化 Dashboard 面板中的显示。再将故障转移后的策略组配置 `type: url-test` 进行延迟测试供用户选择使用

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

## 二、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和 [zashboard 面板](https://github.com/Zephyruso/zashboard)
连接 SSH 后执行如下命令：

```shell
curl -sS -o /tmp/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-arm64.upx
mkdir -p $CRASHDIR/ui/
curl -sS -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
sc
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 3 Mihomo(Meta) 内核

## 三、 编辑 user.yaml 文件
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
log-level: error
allow-lan: true
unified-delay: true
tcp-concurrent: true
external-ui-url: "https://github.com/Zephyruso/zashboard/archive/gh-pages-cdn-fonts.zip"
profile: {store-selected: true, store-fake-ip: true}

hosts:
  miwifi.com: [192.168.31.1, 127.0.0.1]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]

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
    - RULE-SET,trackerslist,real-ip
    - RULE-SET,microsoft-cn,real-ip
    - RULE-SET,apple-cn,real-ip
    - RULE-SET,google-cn,real-ip
    - RULE-SET,games-cn,real-ip
    - RULE-SET,games,fake-ip
    - RULE-SET,ai,fake-ip
    - RULE-SET,proxy,fake-ip
    - RULE-SET,private,real-ip
    - RULE-SET,cn,real-ip
    - MATCH,fake-ip
  nameserver:
    - quic://dns.alidns.com:853
    - https://dns.pub/dns-query
  nameserver-policy: {'rule-set:ads': [rcode://success]}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

---

>`DNS` 私货
{: .prompt-tip }

注：
- ① 本 `dns` 配置中，国内域名走国内 DNS 解析，国外域名走 `fake-ip`，未知域名也走 `fake-ip`，在匹配 `RULE-SET:cn` 规则时会由国外 DNS 解析且配置 `ecs` 提高了兼容性，解析出 IP 在国内则走 `国内 IP` 规则，否则走 `漏网之鱼` 规则（有效解决了“心理 DNS 泄露问题”，详见《[搭载 mihomo 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-mihomo-ruleset/)》）
- ② 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

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
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter-mode: rule
  fake-ip-filter:
    - RULE-SET,trackerslist,real-ip
    - RULE-SET,microsoft-cn,real-ip
    - RULE-SET,apple-cn,real-ip
    - RULE-SET,google-cn,real-ip
    - RULE-SET,games-cn,real-ip
    - RULE-SET,games,fake-ip
    - RULE-SET,ai,fake-ip
    - RULE-SET,proxy,fake-ip
    - RULE-SET,private,real-ip
    - RULE-SET,cn,real-ip
    - MATCH,fake-ip
  respect-rules: true
  nameserver:
    # 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    - 'https://dns.google/dns-query#ecs=211.137.58.0/24'
    - 'https://dns11.quad9.net/dns-query#ecs=211.137.58.0/24'
  proxy-server-nameserver:
    - quic://dns.alidns.com:853
    - https://doh.pub/dns-query
  direct-nameserver:
    - quic://dns.alidns.com:853
    - https://doh.pub/dns-query
  nameserver-policy:
    'rule-set:ads': [rcode://success]
    'rule-set:trackerslist,microsoft-cn,apple-cn,google-cn,games-cn,private,cn': [quic://dns.alidns.com:853, https://doh.pub/dns-query]
```

## 四、 添加定时任务
可参考《[ShellCrash 搭载 mihomo 内核的配置-ruleset 方案/添加定时任务](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-ruleset/#%E4%BA%8C-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》

## 五、 设置部分
1. 设置可参考《[ShellCrash 搭载 mihomo 内核的配置-ruleset 方案/设置部分](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-ruleset/#%E4%B8%89-%E8%AE%BE%E7%BD%AE%E9%83%A8%E5%88%86)》，此处只列举配置的不同之处
2. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，设置如下：  
<img src="/assets/img/dns/dns-null.png" alt="设置部分 2" width="60%" />

## 六、 访问 Dashboard 面板
打开 <http://miwifi.com:9999/ui/> 后，“主机”输入 `192.168.31.1`，“端口”输入 `9999`，点击“提交”即可访问 Dashboard 面板
