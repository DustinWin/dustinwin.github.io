---
title: 分享 ShellCrash 搭载 mihomo 内核采用 ruleset 方案的一套配置
description: 此方案适用于 mihomo，搭载 mihomo 内核，采用 `RULE-SET` 规则搭配 .yaml、.text 和 .mrs 规则集合文件
date: 2024-08-21 18:18:29 +0800
categories: [分享配置, Router]
tags: [Clash, mihomo, ShellCrash, ruleset, rule-set, 分享, Router]
---

## 声明
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)（以 arm64 架构为例，且安装路径为 */data/ShellCrash*）
3. 本方案绕过了 CNIP 且不搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)，但拦截广告效果依然强劲

## 一、 生成配置文件 .yaml 文件直链
具体方法此处不再赘述，请看《[生成带有自定义策略组和规则的 mihomo 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/link-mihomo-ruleset)》，贴一下我使用的配置：

```yaml
proxy-providers:
  🛫 我的机场:
    type: http
    ## 修改为你的 Clash 订阅链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport.yaml
    interval: 86400
    filter: "🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
  🆓 免费订阅:
    type: http
    ## 修改为你的 Clash 订阅链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/free.yaml
    interval: 86400
    filter: "🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600

## 若没有单个出站代理节点，须删除所有 `🆚 vless 节点` 相关内容
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
  - {name: 🚀 节点选择, type: select, proxies: [🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇰🇷 韩国节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点, 🆚 vless 节点]}
  ## 若机场的 UDP 质量不是很好，导致某游戏无法登录或进入房间，可以添加 `disable-udp: true` 配置项解决
  - {name: 🐟 漏网之鱼, type: select, proxies: [🚀 节点选择, 🎯 全球直连]}
  - {name: 📈 网络测试, type: select, proxies: [🎯 全球直连, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇰🇷 韩国节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点, 🆚 vless 节点]}
  - {name: 🤖 人工智能, type: select, proxies: [🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇰🇷 韩国节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🎮 游戏服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🪟 微软服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🇬 谷歌服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🍎 苹果服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🇨🇳 直连域名, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🇨🇳 直连 IP, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🧱 代理域名, type: select, proxies: [🚀 节点选择, 🎯 全球直连]}
  - {name: 📲 电报信息, type: select, proxies: [🚀 节点选择]}
  - {name: 🔒 私有网络, type: select, proxies: [🎯 全球直连]}
  - {name: 🛑 广告拦截, type: select, proxies: [REJECT]}
  - {name: 🎯 全球直连, type: select, proxies: [DIRECT]}

  - {name: 🇭🇰 香港节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇭🇰"}
  - {name: 🇹🇼 台湾节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇹🇼"}
  - {name: 🇯🇵 日本节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇯🇵"}
  - {name: 🇰🇷 韩国节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇰🇷"}
  - {name: 🇸🇬 新加坡节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇸🇬"}
  - {name: 🇺🇸 美国节点, type: url-test, tolerance: 50, use: [🛫 我的机场], filter: "🇺🇸"}
  - {name: 🆓 免费节点, type: load-balance, strategy: consistent-hashing, use: [🆓 免费订阅]}

rule-providers:
  fakeip-filter:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/fakeip-filter.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/fakeip-filter-lite.mrs"
    interval: 86400

  private:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/private.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/private.mrs"
    interval: 86400

  ads:
    type: http
    behavior: domain
    format: mrs
    path: ./rules/ads.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/ads.mrs"
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

  telegramip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./rules/telegramip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/telegramip.mrs"
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

rules:
  - RULE-SET,private,🔒 私有网络
  - RULE-SET,ads,🛑 广告拦截
  - RULE-SET,microsoft-cn,🪟 微软服务
  - RULE-SET,apple-cn,🍎 苹果服务
  - RULE-SET,google-cn,🇬 谷歌服务
  - RULE-SET,games-cn,🎮 游戏服务
  - RULE-SET,ai,🤖 人工智能
  - RULE-SET,networktest,📈 网络测试
  - RULE-SET,proxy,🧱 代理域名
  - RULE-SET,cn,🇨🇳 直连域名
  - RULE-SET,telegramip,📲 电报信息,no-resolve
  - RULE-SET,privateip,🔒 私有网络,no-resolve
  - RULE-SET,cnip,🇨🇳 直连 IP
  - MATCH,🐟 漏网之鱼
```

## 二、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和 [zashboard 面板](https://github.com/Zephyruso/zashboard)
连接 SSH 后运行如下命令：

```shell
curl -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools@mihomo/mihomo-meta-linux-armv8.tar.gz | tar -zx -C /tmp/
mkdir -p $CRASHDIR/ui/
curl -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools@Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
crash
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 3 Clash-Meta 内核

## 三、 编辑 user.yaml 文件
连接 SSH 后运行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
mode: rule
log-level: error
allow-lan: true
unified-delay: false
tcp-concurrent: true
external-ui-url: "https://github.com/Zephyruso/zashboard/releases/latest/download/dist-cdn-fonts.zip"
global-client-fingerprint: chrome
profile: {store-selected: true}

sniffer:
  enable: true
  parse-pure-ip: true
  sniff: {HTTP: {ports: [80, 8080-8880], override-destination: true}, TLS: {ports: [443, 8443]}, QUIC: {ports: [443, 8443]}}
  skip-domain: ['Mijia Cloud']

hosts:
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  miwifi.com: 192.168.31.1
  services.googleapis.cn: services.googleapis.com

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['rule-set:fakeip-filter,private,cn']
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  direct-nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 四、 添加定时任务
1. 连接 SSH 后运行 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o /data/ShellCrash/CrashCore.tar.gz -L https://ghgo.xyz/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-alpha-linux-armv8.tar.gz && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新mihomo内核
202#curl -o /data/ShellCrash/cn_ip.txt -L https://ghgo.xyz/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv4.txt && curl -o /data/ShellCrash/cn_ipv6.txt -L https://ghgo.xyz/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `crash`，进入 ShellCrash -> 5 配置自动任务 -> 1 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-mihomo-ruleset.png" alt="添加定时任务" width="60%" />

## 五、 设置部分
1. 设置可参考《[ShellCrash 搭载 mihomo 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/toolsettings-shellcrash-mihomo-ruleset)》，此处只列举配置的不同之处
2. 进入主菜单 -> 2 内核功能设置，设置如下：  
<img src="/assets/img/share/tproxy-mix-cnip.png" alt="设置部分 1" width="60%" />

3. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，设置如下：  
<img src="/assets/img/share/dns-null.png" alt="设置部分 2" width="60%" />

4. 进入主菜单 -> 7 内核进阶设置，不要启用 4 启用域名嗅探（因 user.yaml 文件中已添加 `dns.sniffer` 配置项）
5. 进入主菜单 -> 7 内核进阶设置 -> 5 自定义端口及秘钥，设置为 `9090`
6. 进入主菜单 -> 6 导入配置文件 -> 2 在线获取完整配置文件，粘贴第《一》步中生成的配置文件 .yaml 文件直链，启动服务即可

## 六、 访问 Dashboard 面板
打开 http://miwifi.com:9090/ui/ 后，“主机”输入 `192.168.31.1`，点击“提交”即可访问 Dashboard 面板  
<img src="/assets/img/share/192-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />