---
title: 生成带有自定义策略组和规则的 mihomo 配置文件直链-ruleset 方案
description: 此教程搭载 mihomo 内核，采用 <code>RULE-SET</code> 规则搭配 .list 和 .mrs 规则集合文件
date: 2024-08-21 07:41:24 +0800
categories: [直链配置, mihomo 直链]
tags: [Clash, mihomo, 直链, 订阅, ruleset, rule-set, 基础]
---

> 说明
{: .prompt-tip }
1. 本教程可以生成扩展名为 .yaml 配置文件直链，可以**一键导入使用了 [mihomo](https://github.com/MetaCubeX/mihomo) 内核的客户端**  
如：[ShellCrash](https://github.com/juewuy/ShellCrash)、[OpenClash](https://github.com/vernesong/OpenClash)、[Clash Verge](https://github.com/MetaCubeX/clash-verge) 和 [Clash.Meta for Android](https://github.com/MetaCubeX/ClashMetaForAndroid) 等，详见[支持 mihomo 的工具](https://wiki.metacubex.one/startup/client)
2. 生成的订阅链接地址不会改变，支持更新订阅，**支持国内访问，支持同步机场节点**
3. 生成的订阅链接**自带规则集**，规则集来源 [DustinWin/ruleset_geodata/ruleset](https://github.com/DustinWin/ruleset_geodata#%E4%BA%8C-ruleset-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
4. 请先**确定自己机场的订阅链接是否为 Clash 订阅链接**，若不是，需前往[肥羊在线订阅转换工具](https://suburl.v1.mk)进行转换，“生成类型”选择“Clash”，其它参数保持默认即可，转换后的订阅链接需要在末尾添加`&flag=clash`，然后添加到 .yaml 文件代理集合 `proxy-providers` 的 `url` 中
5. 推荐使用 [Visual Studio Code](https://code.visualstudio.com/Download) 等专业编辑器来修改配置文件
6. ShellCrash 支持本地导入配置文件，可以直接将下方的 .yaml 直链文件内容复制到 `$CRASHDIR/yamls/config.yaml`{: .filepath} 文件中，可代替通过 ShellCrash 配置脚本 → 6) 配置文件管理 → a) 添加提供者

## 一、 准备编辑 .yaml 直链文件
### 1. 注册 [Gist](https://gist.github.com)
进入 <https://gist.github.com> 网站并注册

### 2. 打开编辑页面
登录并打开 Gist 可以直接编辑文件，或者点击页面右上角头像左边的“+”图标新建文件

### 3. 输入描述和完整文件名
“Gist description...”输入描述，随意填写；“Filename including extension...”输入完整文件名**包括扩展名**，如 mihomolink.yaml  
<img src="/assets/img/link/file-extension-yaml.png" alt="输入描述和完整文件名" width="60%" />

## 二、 添加模板
### 1. 白名单模式（没有命中规则的网络流量统统使用代理，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户，推荐）

```yaml
# 代理集合（获取机场订阅链接内的所有节点）
proxy-providers:
  🛫 机场订阅 1:
    type: http
    # 机场订阅链接，使用 Clash 链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport1.yaml
    interval: 86400
    # 初步筛选需要的节点，可有效减轻路由器压力，支持正则表达式，不筛选可删除此配置项
    filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)"
    # 初步排除不需要的节点，支持正则表达式，若不排除可删除此配置项
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      # 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 机场订阅 1-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 机场订阅 1-"
      # 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 机场订阅 1”；推荐有多个机场时使用
      additional-suffix: "-🛫 机场订阅 1"

  🛫 机场订阅 2:
    type: http
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport2.yaml
    interval: 43200
    filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)"
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      # 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 机场订阅 2-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 机场订阅 2-"
      # 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 机场订阅 2”；推荐有多个机场时使用
      additional-suffix: "-🛫 机场订阅 2"

# 单个出站代理节点（以 vless 为例）
proxies:
  - name: 🆓 免费节点
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

# 策略组
proxy-groups:
  # 手动选择国家或地区节点；根据“国家或地区策略组”名称对 `proxies` 值进行增删改，须一一对应
  - {name: 🚀 节点选择, type: select, proxies: [♻️ 自动选择, 👉 手动选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  # 选择`🎯 全球直连`为测试本地网络（运营商网络速度和 IPv6 支持情况），可选择其它节点用于测试机场节点速度和 IPv6 支持情况
  - {name: 📈 网络测试, type: select, proxies: [🎯 全球直连, 🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  - {name: 🕹️ 游戏平台, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🤖 AI 平台, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🎮 游戏服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🪟 微软服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🇬 谷歌服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🍎 苹果服务, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🌍 国外媒体, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🇨🇳 国内域名, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🀄️ 国内 IP, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🌎 国外域名, type: select, proxies: [🚀 节点选择, 🎯 全球直连]}
  - {name: 📲 电报消息, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  # 若使用 ShellCrash，由于无法判断本机进程（默认 `find-process-mode: off`），需删除此条 `⬇️ 直连软件`；若在面板 Dashboard 中需隐藏该策略组，可添加 `hidden: true` 配置项
  - {name: ⬇️ 直连软件, type: select, proxies: [🎯 全球直连], hidden: true}
  - {name: 📋 Trackerslist, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  - {name: 🔒 私有网络, type: select, proxies: [🎯 全球直连], hidden: true}
  # 若机场的 UDP 质量不是很好，导致某游戏无法登录或进入房间，可添加 `disable-udp: true` 配置项解决
  - {name: 🐟 漏网之鱼, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点, 🎯 全球直连]}
  - {name: 🛑 广告域名, type: select, proxies: [🔴 全球拦截, 🟢 全球绕过]}
  - {name: 🔴 全球拦截, type: select, proxies: [REJECT], hidden: true}
  - {name: 🟢 全球绕过, type: select, proxies: [PASS], hidden: true}
  - {name: 🎯 全球直连, type: select, proxies: [DIRECT], hidden: true}

  # ----------------国家或地区策略组---------------------
  # 自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 50ms 才会切换到延迟低的那个节点；筛选出“香港”节点，支持正则表达式
  - {name: 🇭🇰 香港节点, type: url-test, tolerance: 50, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)"}
  # 节点自动回退，默认选择第一个节点，节点超时后则会按代理顺序选择下一个可用节点，以此类推。也被叫做“故障转移”
  - {name: 🇹🇼 台湾节点, type: fallback, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)"}
  # 节点负载均衡，即将请求均匀分配到多个节点上，优点是更稳定，速度可能有提升；将相同顶级域名的请求分配给策略组内的同一个代理节点；推荐在节点复用比较多的情况下使用
  - {name: 🇯🇵 日本节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇯🇵|日|jp|japan)"}
  # 可使用 `include-all: true` 代替 `use: [🛫 机场订阅 1, 🛫 机场订阅 2, ...]`，意思为引入所有出站代理以及代理集合
  - {name: 🇸🇬 新加坡节点, type: url-test, tolerance: 50, include-all: true, filter: "(?i)(🇸🇬|新|sg|singapore)"}
  - {name: 🇺🇸 美国节点, type: url-test, tolerance: 100, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)"}
  - {name: ♻️ 自动选择, type: url-test, tolerance: 100, include-all: true}
  - {name: 👉 手动选择, type: select, include-all: true}

# 规则集（yaml 文件每天自动更新）
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

  # 若使用 ShellCrash，由于无法判断本机进程（默认 `find-process-mode: off`），需删除此条 `applications`
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

  media:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/media.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/media.mrs"
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

  mediaip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/mediaip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/mediaip.mrs"
    interval: 86400

# 规则
rules:
  # 自定义规则优先放前面
  - RULE-SET,private,🔒 私有网络
  - RULE-SET,ads,🛑 广告域名
  - RULE-SET,trackerslist,📋 Trackerslist
  # 为了使 P2P 流量（BT 下载）走直连，可添加一条 `DST-PORT` 规则（ShellCrash 会默认启用“只代理常用端口”，可删除此条 `DST-PORT`）
  - DST-PORT,6881-6889,🎯 全球直连
  # 若使用 ShellCrash，由于无法判断本机进程（默认 `find-process-mode: off`），需删除此条 `RULE-SET`
  - RULE-SET,applications,⬇️ 直连软件
  - RULE-SET,microsoft-cn,🪟 微软服务
  - RULE-SET,apple-cn,🍎 苹果服务
  - RULE-SET,google-cn,🇬 谷歌服务
  - RULE-SET,games-cn,🎮 游戏服务
  - RULE-SET,games,🕹️ 游戏平台
  - RULE-SET,media,🌍 国外媒体
  - RULE-SET,ai,🤖 AI 平台
  - RULE-SET,networktest,📈 网络测试
  - RULE-SET,proxy,🌎 国外域名
  - RULE-SET,cn,🇨🇳 国内域名
  - RULE-SET,privateip,🔒 私有网络,no-resolve
  - RULE-SET,cnip,🀄️ 国内 IP
  - RULE-SET,telegramip,📲 电报消息,no-resolve
  - RULE-SET,mediaip,🌍 国外媒体
  - MATCH,🐟 漏网之鱼
```

将模板内容复制到自己 Gist 新建的 .yaml 文件中

### 2. 黑名单模式（只有命中规则的网络流量才使用代理，适用于服务器线路网络质量不稳定或不够快，或服务器流量紧缺的用户。通常也是软路由用户、家庭网关用户的常用模式）

```yaml
# 代理集合（获取机场订阅链接内的所有节点）
proxy-providers:
  🛫 机场订阅 1:
    type: http
    # 机场订阅链接，使用 Clash 链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport1.yaml
    interval: 86400
    # 初步筛选需要的节点，可有效减轻路由器压力，支持正则表达式，不筛选可删除此配置项
    filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)"
    # 初步排除不需要的节点，支持正则表达式，若不排除可删除此配置项
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      # 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 机场订阅 1-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 机场订阅 1-"
      # 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 机场订阅 1”；推荐有多个机场时使用
      additional-suffix: "-🛫 机场订阅 1"

  🛫 机场订阅 2:
    type: http
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport2.yaml
    interval: 43200
    filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)"
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      # 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 机场订阅 2-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 机场订阅 2-"
      # 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 机场订阅 2”；推荐有多个机场时使用
      additional-suffix: "-🛫 机场订阅 2"

# 单个出站代理节点（以 vless 为例）
proxies:
  - name: 🆓 免费节点
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

# 策略组
proxy-groups:
  # 手动选择国家或地区节点；根据“国家或地区策略组”名称对 `proxies` 值进行增删改，须一一对应
  - {name: 🚀 节点选择, type: select, proxies: [♻️ 自动选择, 👉 手动选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  # 选择`🎯 全球直连`为测试本地网络（运营商网络速度和 IPv6 支持情况），可选择其它节点用于测试机场节点速度和 IPv6 支持情况
  - {name: 📈 网络测试, type: select, proxies: [🎯 全球直连, 🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  - {name: 🕹️ 游戏平台, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🤖 AI 平台, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🌍 国外媒体, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点]}
  - {name: 🌎 国外域名, type: select, proxies: [🚀 节点选择, 🎯 全球直连]}
  - {name: 📲 电报消息, type: select, proxies: [🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  - {name: 📋 Trackerslist, type: select, proxies: [🎯 全球直连, 🚀 节点选择]}
  # 若在面板 Dashboard 中需隐藏该策略组，可添加 `hidden: true` 配置项
  - {name: 🔒 私有网络, type: select, proxies: [🎯 全球直连], hidden: true}
  - {name: 🐟 漏网之鱼, type: select, proxies: [🎯 全球直连, 🚀 节点选择, 🇭🇰 香港节点, 🇹🇼 台湾节点, 🇯🇵 日本节点, 🇸🇬 新加坡节点, 🇺🇸 美国节点, 🆓 免费节点]}
  - {name: 🛑 广告域名, type: select, proxies: [🔴 全球拦截, 🟢 全球绕过]}
  - {name: 🔴 全球拦截, type: select, proxies: [REJECT], hidden: true}
  - {name: 🟢 全球绕过, type: select, proxies: [PASS], hidden: true}
  - {name: 🎯 全球直连, type: select, proxies: [DIRECT], hidden: true}

  # ----------------国家或地区策略组---------------------
  # 自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 50ms 才会切换到延迟低的那个节点；筛选出“香港”节点，支持正则表达式
  - {name: 🇭🇰 香港节点, type: url-test, tolerance: 50, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)"}
  # 节点自动回退，默认选择第一个节点，节点超时后则会按代理顺序选择下一个可用节点，以此类推。也被叫做“故障转移”
  - {name: 🇹🇼 台湾节点, type: fallback, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)"}
  # 节点负载均衡，即将请求均匀分配到多个节点上，优点是更稳定，速度可能有提升；将相同顶级域名的请求分配给策略组内的同一个代理节点；推荐在节点复用比较多的情况下使用
  - {name: 🇯🇵 日本节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇯🇵|日|jp|japan)"}
  # 可使用 `include-all: true` 代替 `use: [🛫 机场订阅 1, 🛫 机场订阅 2, ...]`，意思为引入所有出站代理以及代理集合
  - {name: 🇸🇬 新加坡节点, type: url-test, tolerance: 50, include-all: true, filter: "(?i)(🇸🇬|新|sg|singapore)"}
  - {name: 🇺🇸 美国节点, type: url-test, tolerance: 100, use: [🛫 机场订阅 1, 🛫 机场订阅 2], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)"}
  - {name: ♻️ 自动选择, type: url-test, tolerance: 100, include-all: true}
  - {name: 👉 手动选择, type: select, include-all: true}

# 规则集（yaml 文件每天自动更新）
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

  games:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/games.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/games.mrs"
    interval: 86400

  media:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/media.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/media.mrs"
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

  tld-proxy:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/tld-proxy.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/tld-proxy.mrs"
    interval: 86400

  gfw:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/gfw.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/gfw.mrs"
    interval: 86400

  telegramip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/telegramip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/telegramip.mrs"
    interval: 86400

  mediaip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/mediaip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/mediaip.mrs"
    interval: 86400

# 规则
rules:
  - RULE-SET,private,🔒 私有网络
  - RULE-SET,ads,🛑 广告域名
  - RULE-SET,trackerslist,📋 Trackerslist
  - RULE-SET,games,🕹️ 游戏平台
  - RULE-SET,media,🌍 国外媒体
  - RULE-SET,ai,🤖 AI 平台
  - RULE-SET,networktest,📈 网络测试
  - RULE-SET,tld-proxy,🌎 国外域名
  - RULE-SET,gfw,🌎 国外域名
  - RULE-SET,telegramip,📲 电报消息,no-resolve
  - RULE-SET,mediaip,🌍 国外媒体
  - MATCH,🐟 漏网之鱼
```

将模板内容复制到自己 Gist 新建的 .yaml 文件中

## 三、 修改模板
1. 将代理集合 `proxy-providers` 中的 `url` 链接改成自己机场的订阅链接（必须为 Clash 订阅链接，详见《说明 4》）  
2. 确定自己机场中有哪些国家或地区的节点，然后对 `proxy-groups` 中的 “**国家或地区策略组**”以及 `proxies` 里的国家或地区进行增删改
   - 注：两者中的国家或地区必须一一对应，新增就全部新增，删除就全部删除，修改就全部修改（重要）

3. 在“国家或地区策略组”中的 `filter` 支持[正则表达式](https://www.lddgo.net/string/golangregex)，可以精确地筛选出指定的国家或地区节点  
例如：我想筛选出“香港 IPLC”节点，`filter` 可以这样写：
`filter: "香港.*IPLC|IPLC.*香港"`
   - 小窍门：使用 [ChatGPT](https://chatgpt.com) 等 AI 工具查询符合自己要求的正则表达式

4. 在 `🚀 节点选择` 策略组下的 `proxies` 里，可以将最稳定的节点放在最前面，配置完成后会自动选择最稳定的节点  
5. 在“国家或地区策略组”里，`type` 为 `url-test` 就是自动选择延迟最低的节点，将 `url-test` 改成 `select` 就是手动选择节点  
举个例子：我的机场包含有 2 个节点，分别是新加坡节点和日本节点，我想让 [Netflix](https://www.netflix.com/) 自动选择延迟最低的新加坡节点，[哔哩哔哩](https://www.bilibili.com)可以手动选择日本任一节点，这个需求怎么写？  
注：
   - ① 以下只是节选，请酌情套用
   - ② 本教程搭配的规则集合文件包含有 `netflix`、`netflixip` 和 `bilibili`

```yaml
# 策略组
proxy-groups:
  # 默认选择新加坡节点
  - {name: 🎥 奈飞视频, type: select, proxies: [🇸🇬 新加坡节点]}
  # 默认选择日本节点，也可切换到直连
  - {name: 📺 哔哩哔哩, type: select, proxies: [🇯🇵 日本节点, 🎯 全球直连]}
  # 自动选择延迟最低的新加坡节点；容差大于 50ms 才会切换到延迟低的那个节点
  - {name: 🇸🇬 新加坡节点, type: url-test, tolerance: 50, include-all: true, filter: "(?i)(🇸🇬|新|sg|singapore)"}
  # 手动选择日本任一节点
  - {name: 🇯🇵 日本节点, type: select, include-all: true, filter: "(?i)(🇯🇵|日|jp|japan)"}
  - {name: 🎯 全球直连, type: select, proxies: [DIRECT], hidden: true}

# 规则集（yaml 文件每天自动更新）
rule-providers:
  netflix:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/netflix.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/netflix.mrs"
    interval: 86400

  netflixip:
    type: http
    behavior: ipcidr
    format: mrs
    path: ./ruleset/netflixip.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/netflixip.mrs"
    interval: 86400

  bilibili:
    type: http
    behavior: domain
    format: mrs
    path: ./ruleset/bilibili.mrs
    url: "https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-ruleset/bilibili.mrs"
    interval: 86400

# 规则
rules:
  # 自定义规则优先放前面
  - RULE-SET,netflix,🎥 奈飞视频
  - RULE-SET,netflixip,🎥 奈飞视频
  - RULE-SET,bilibili,📺 哔哩哔哩
```
> 若有其它需求，可进入 [blackmatrix7/ios_rule_script/rule/Clash](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Clash) 搜索关键字，通过能够搜索到的关键字来编写规则（推荐使用“xxx_Classical.yaml”文件，`rule-provider` 内须配置 `behavior: classical`）
{: .prompt-tip }

## 四、 生成 .yaml 文件链接
1. 编辑完成后，点击右下角的“Create secret gist”按钮，然后点击右上角的“Raw”按钮  
<img src="/assets/img/link/click-raw-yaml.png" alt="生成 .yaml 文件链接 1" width="60%" />

2. 取出地址栏中的网址，删除后面的一串随机码，**完成后该 .yaml 文件直链才是最终生成的订阅链接**，该订阅链接地址不会改变，在不更改文件名的情况下即使编辑该 .yaml 文件并提交了 n 次也不会改变。举个例子，这是原地址：  
`https://gist.githubusercontent.com/DustinWin/3d1a5039fc6f88a1da44f8e0b1c8e181/raw/6e9d5fbbaf3b1f721eb9245e7944b2ca39512705/mihomolink.yaml`  
删除后面的一串随机码（当前编辑该文件生成的随机码“6e9d5fbbaf3b1f721eb9245e7944b2ca39512705”）  
<img src="/assets/img/link/2705-yaml.png" alt="生成 .yaml 文件链接 2" width="60%" />  
删除后变成：  
`https://gist.githubusercontent.com/DustinWin/3d1a5039fc6f88a1da44f8e0b1c8e181/raw/mihomolink.yaml`

- 注：若无法直连访问，可在链接上添加 `https://ghfast.top/` 前缀，即：将链接改为 `https://ghfast.top/https://gist.githubusercontent.com/DustinWin/3d1a5039fc6f88a1da44f8e0b1c8e181/raw/mihomolink.yaml`

## 五、 导入订阅链接（以 ShellCrash 导入订阅链接为例）
1. 进入 ShellCrash 配置脚本 → a) 添加提供者 → 1) 设置名称或代号，如输入 `mihomo`；后进入 2) 设置链接或路径，粘贴最终生成的订阅链接，选择“a) 保存此提供者”
2. 进入 6) 配置文件管理 → 6) 配置文件管理 → c) 在线生成配置文件 → 6) 自定义浏览器 UA，选择“2) 不使用 UA”
3. 进入 6) 配置文件管理 → 1) mihomo，选择“e) 在线获取此配置文件”即可
4. 具体设置请参考《[ShellCrash 搭载 mihomo 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-ruleset)》
