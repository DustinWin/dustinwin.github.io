---
title: 分享 ShellCrash 搭载 mihomo 内核采用 ruleset 方案的一套配置
description: 此配置搭载 mihomo 内核，采用 `RULE-SET` 规则搭配 .list 和 .mrs 规则集合文件
date: 2024-08-21 18:18:29 +0800
categories: [分享配置, Router]
tags: [Clash, mihomo, ShellCrash, ruleset, rule-set, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)（以 arm64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
3. 本方案绕过了 CNIP 且不搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)，但拦截广告效果依然强劲

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
  - {name: 人工智能, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ai.png"}
  - {name: Trackerslist, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trackerslist.png"}
  - {name: 游戏服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/games-cn.png"}
  - {name: 微软服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/microsoft-cn.png"}
  - {name: 谷歌服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/google-cn.png"}
  - {name: 苹果服务, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/apple-cn.png"}
  - {name: 直连域名, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cn.png"}
  - {name: 直连 IP, type: select, proxies: [全球直连, 节点选择], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/cnip.png"}
  - {name: 代理域名, type: select, proxies: [节点选择, 全球直连], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/global.png"}
  - {name: 电报消息, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/telegram.png"}
  - {name: 私有网络, type: select, proxies: [全球直连], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/private.png"}
  # 若机场的 UDP 质量不是很好，导致某游戏无法登录或进入房间，可以添加 `disable-udp: true` 配置项解决
  - {name: 漏网之鱼, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点, 全球直连], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/match.png"}
  - {name: 广告域名, type: select, proxies: [全球拦截, 全球绕过], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ads.png"}
  - {name: 全球拦截, type: select, proxies: [REJECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/reject.png"}
  - {name: 全球绕过, type: select, proxies: [PASS], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/pass.png"}
  - {name: 全球直连, type: select, proxies: [DIRECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/direct.png"}

  - {name: 香港节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 台湾节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 日本节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 新加坡节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 美国节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 免费节点, type: url-test, tolerance: 50, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}

rule-providers:
  ads:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/ads.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ads.mrs"
    interval: 86400

  private:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/private.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/private.mrs"
    interval: 86400

  trackerslist:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/trackerslist.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/trackerslist.mrs"
    interval: 86400

  microsoft-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/microsoft-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/microsoft-cn.mrs"
    interval: 86400

  apple-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/apple-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/apple-cn.mrs"
    interval: 86400

  google-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/google-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/google-cn.mrs"
    interval: 86400

  games-cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/games-cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/games-cn.mrs"
    interval: 86400

  ai:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/ai.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ai.mrs"
    interval: 86400

  networktest:
    type: http
    behavior: classical
    format: text
    path: ./rules/networktest.list
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/networktest.list"
    interval: 86400

  proxy:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/proxy.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/proxy.mrs"
    interval: 86400

  cn:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/cn.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cn.mrs"
    interval: 86400

  privateip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./rules/privateip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/privateip.mrs"
    interval: 86400

  cnip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./rules/cnip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/cnip.mrs"
    interval: 86400

  telegramip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./rules/telegramip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/telegramip.mrs"
    interval: 86400

rules:
  - RULE-SET,private,私有网络
  - RULE-SET,ads,广告域名
  - RULE-SET,trackerslist,Trackerslist
  - RULE-SET,microsoft-cn,微软服务
  - RULE-SET,apple-cn,苹果服务
  - RULE-SET,google-cn,谷歌服务
  - RULE-SET,games-cn,游戏服务
  - RULE-SET,ai,人工智能
  - RULE-SET,networktest,网络测试
  - RULE-SET,proxy,代理域名
  - RULE-SET,cn,直连域名
  - RULE-SET,privateip,私有网络,no-resolve
  - RULE-SET,cnip,直连 IP
  - RULE-SET,telegramip,电报消息,no-resolve
  - MATCH,漏网之鱼
```

---

>`proxy-groups` 私货
{: .prompt-tip }

注：
- 1. 本 `proxy-groups` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `type: url-test` 进行延迟测试
- 2. 再将上述延迟测试最低的策略组配置 `type: fallback` 进行自动回退

```yaml
proxy-groups:
  - {name: 香港节点, type: fallback, proxies: ["香港节点-ss", "香港节点-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 香港节点-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 香港节点-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 台湾节点, type: fallback, proxies: ["台湾节点-ss", "台湾节点-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 台湾节点-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 台湾节点-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 日本节点, type: fallback, proxies: ["日本节点-ss", "日本节点-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 日本节点-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 日本节点-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 新加坡节点, type: fallback, proxies: ["新加坡节点-ss", "新加坡节点-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 新加坡节点-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 新加坡节点-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 美国节点, type: fallback, proxies: ["美国节点-ss", "美国节点-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 美国节点-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 美国节点-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 免费节点, type: url-test, tolerance: 50, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}
```

---

## 二、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和 [zashboard 面板](https://github.com/Zephyruso/zashboard)
连接 SSH 后执行如下命令：

```shell
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-armv8.tar.gz | tar -zx -C /tmp/
mkdir -p $CRASHDIR/ui/
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
crash
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 3 Clash-Meta 内核

## 三、 编辑 user.yaml 文件
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
log-level: error
allow-lan: true
unified-delay: true
tcp-concurrent: true
external-ui-url: "https://github.com/Zephyruso/zashboard/archive/gh-pages-cdn-fonts.zip"
global-client-fingerprint: chrome
profile: {store-selected: true, store-fake-ip: true}

sniffer:
  enable: true
  sniff: {HTTP: {ports: [80, 8080-8880], override-destination: true}, TLS: {ports: [443, 8443]}, QUIC: {ports: [443, 8443]}}
  skip-domain: ['Mijia Cloud']

hosts:
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]
  miwifi.com: [192.168.31.1, 127.0.0.1]
  services.googleapis.cn: [services.googleapis.com]

dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 28.0.0.1/8
  enhanced-mode: fake-ip
  fake-ip-filter: ['rule-set:trackerslist,private,cn']
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  nameserver-policy: {'rule-set:ads': [rcode://success]}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，未知域名由国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 mihomo 内核配置 DNS 不泄露教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-mihomo-ruleset/)》），且配置 `ecs` 提高了兼容性
- 2. 推荐将 `ecs` 设置为当前网络的公网 IP 段，如当前网络公网 IP 为 `202.103.17.123`，可设置为 `202.103.17.0/24`（后续维护更新可直接执行命令 `sed -i -E "s/(ecs=)[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/yamls/user.yaml`）

```yaml
hosts:
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  doh.pub: [1.12.12.12, 1.12.12.21, 120.53.53.53]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  dns11.quad9.net: [9.9.9.11, 149.112.112.11, 2620:fe::11, 2620:fe::fe:11]
  miwifi.com: [192.168.31.1, 127.0.0.1]
  services.googleapis.cn: [services.googleapis.com]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 28.0.0.1/8
  enhanced-mode: fake-ip
  fake-ip-filter: ['rule-set:trackerslist,private,cn']
  respect-rules: true
  nameserver:
    # 推荐将 `ecs` 设置为当前网络的公网 IP 段
    - 'https://dns.google/dns-query#ecs=202.103.17.0/24'
    - 'https://dns11.quad9.net/dns-query#ecs=202.103.17.0/24'
  proxy-server-nameserver:
    - quic://dns.alidns.com:853
    - https://doh.pub/dns-query
  direct-nameserver:
    - quic://dns.alidns.com:853
    - https://doh.pub/dns-query
  nameserver-policy: {'rule-set:ads': [rcode://success]}
```

**新增定时任务**  
连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
203#sed -i -E "s/(ecs=)[0-9.]+\/[0-9]+/\1$(curl -s 4.ipw.cn | cut -d. -f1-3).0\/24/" $CRASHDIR/yamls/user.yaml && curl -H 'Authorization: Bearer ""' http://192.168.31.1:9090/configs -d '{"path": "", "payload": ""}' -X PUT >/dev/null 2>&1#更新ecs地址
```

---

## 四、 添加定时任务
1. 连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-alpha-linux-armv8.tar.gz && $CRASHDIR/start.sh restart >/dev/null 2>&1#更新mihomo内核
202#curl -o $CRASHDIR/cn_ip.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv4.txt && curl -o $CRASHDIR/cn_ipv6.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `crash`，进入 ShellCrash → 5 配置自动任务 → 1 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-mihomo-ruleset.png" alt="添加定时任务" width="60%" />

## 五、 设置部分
1. 设置可参考《[ShellCrash 搭载 mihomo 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-ruleset)》，此处只列举配置的不同之处
2. 进入主菜单 → 2 内核功能设置 → 2 切换 DNS 运行模式 → 4 DNS 进阶设置，设置如下：  
<img src="/assets/img/dns/dns-null.png" alt="设置部分 2" width="60%" />

3. 进入主菜单 → 7 内核进阶设置，不要启用 4 启用域名嗅探（因 user.yaml 文件中已添加 `dns.sniffer` 配置项）
4. 进入主菜单 → 7 内核进阶设置 → 5 自定义端口及秘钥，设置为 `9090`
5. 进入主菜单 → 6 导入配置文件 → 2 在线获取完整配置文件，粘贴《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrash-mihomo-ruleset/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-yaml-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的配置文件 .yaml 文件直链，启动服务即可

## 六、 访问 Dashboard 面板
打开 <http://miwifi.com:9090/ui/> 后，“主机”输入 `192.168.31.1`，点击“提交”即可访问 Dashboard 面板  
<img src="/assets/img/share/192-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />
