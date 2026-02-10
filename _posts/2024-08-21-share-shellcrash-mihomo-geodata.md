---
title: 分享 ShellCrash 搭载 mihomo 内核采用 geodata 方案的一套配置
description: 此配置搭载 mihomo 内核，采用 <code>GEOSITE</code> 和 <code>GEOIP</code> 规则搭配 geosite.dat 和 geoip.dat（或 Country.mmdb）路由规则文件
date: 2024-08-21 18:12:46 +0800
categories: [分享配置, Router]
tags: [Clash, mihomo, ShellCrash, geodata, geosite, 分享, Router]
---

> 声明
{: .prompt-warning }
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)（以 ARM64 架构为例，且安装路径为 `/data/ShellCrash`{: .filepath}）
3. 本方案绕过了 CNIP 且不搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)，在 DNS 层拦截广告
4. 本人将路由器设置了每天早上 6 点重启，使得《[五](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrash-mihomo-geodata/#%E4%BA%94-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中设置的定时任务生效
5. 若 `谷歌服务` 出现错误如 [Google Chrome](https://www.google.com/chrome/) 检查更新失败，请删除 `nameserver` 相关配置项里的所有[阿里云公共 DNS](https://help.aliyun.com/zh/dns/what-is-alibaba-cloud-public-dns)

## 一、 生成配置文件 .yaml 文件直链
具体方法此处不再赘述，请看《[生成带有自定义策略组和规则的 mihomo 配置文件直链-geodata 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-mihomo-geodata)》，贴一下我使用的配置：
- 注：`rules` 部分的 `geosite` 和 `geoip` 内容须与 `geodata-url` 中的路由规则文件相匹配

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
  - {name: AI 平台, type: select, proxies: [节点选择, 香港节点, 台湾节点, 日本节点, 新加坡节点, 美国节点, 免费节点, 🆚 vless 节点], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/ai.png"}
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
  - {name: 全球直连, type: select, proxies: [DIRECT], hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/direct.png"}

  - {name: 香港节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 台湾节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 日本节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 新加坡节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 美国节点, type: load-balance, strategy: consistent-hashing, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 免费节点, type: url-test, tolerance: 100, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}

rules:
  - GEOSITE,private,私有网络
  - GEOSITE,trackerslist,Trackerslist
  - GEOSITE,microsoft-cn,微软服务
  - GEOSITE,apple-cn,苹果服务
  - GEOSITE,google-cn,谷歌服务
  - GEOSITE,games-cn,游戏服务
  - GEOSITE,ai,AI 平台
  - GEOSITE,networktest,网络测试
  - GEOSITE,proxy,代理域名
  - GEOSITE,cn,直连域名
  - GEOIP,private,私有网络,no-resolve
  - GEOIP,cn,直连 IP
  - GEOIP,telegram,电报消息,no-resolve
  - MATCH,漏网之鱼
```

---

>`proxy-groups` 私货
{: .prompt-tip }

注：
- 1. 本 `proxy-groups` 配置中，将不同的节点类型（如：`Shadowsocks` 和 `Trojan`）分别配置 `type: url-test` 进行延迟测试，且配置 `hidden: true` 以简化 Dashboard 面板中的显示
- 2. 再将上述延迟测试最低的策略组配置 `type: fallback` 进行自动回退

```yaml
proxy-groups:
  - {name: 香港节点, type: fallback, proxies: ["香港-ss", "香港-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/hongkong.png"}
  - {name: 香港-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 香港-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇭🇰|港|hk|hongkong|hong kong)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 台湾节点, type: fallback, proxies: ["台湾-ss", "台湾-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/taiwan.png"}
  - {name: 台湾-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 台湾-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇹🇼|台|tw|taiwan|tai wan)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 日本节点, type: fallback, proxies: ["日本-ss", "日本-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/japan.png"}
  - {name: 日本-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 日本-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇯🇵|日|jp|japan)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 新加坡节点, type: fallback, proxies: ["新加坡-ss", "新加坡-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/singapore.png"}
  - {name: 新加坡-ss, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 新加坡-trojan, type: url-test, tolerance: 50, use: [🛫 机场订阅], filter: "(?i)(🇸🇬|新|sg|singapore)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 美国节点, type: fallback, proxies: ["美国-ss", "美国-trojan"], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/unitedstates.png"}
  - {name: 美国-ss, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", exclude-type: "Trojan", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/shadowsocks.png"}
  - {name: 美国-trojan, type: url-test, tolerance: 100, use: [🛫 机场订阅], filter: "(?i)(🇺🇸|美|us|unitedstates|united states)", exclude-type: "Shadowsocks", hidden: true, icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/trojan.png"}
  - {name: 免费节点, type: url-test, tolerance: 100, use: [🆓 免费订阅], icon: "https://github.com/DustinWin/ruleset_geodata/releases/download/icons/free.png"}
```

---

## 二、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和 [zashboard 面板](https://github.com/Zephyruso/zashboard)
连接 SSH 后执行如下命令：

```shell
curl -o /tmp/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-arm64.upx
mkdir -p $CRASHDIR/ui/
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/
sc
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 3 Mihomo(Meta) 内核

## 三、 导入路由规则文件
> 路由规则文件和 user.yaml 都属高度定制，牵一发而动全身
{: .prompt-warning }
连接 SSH 后执行如下命令：

```shell
curl -o $CRASHDIR/GeoSite.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/geosite.dat
curl -o $CRASHDIR/GeoIP.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/geoip-lite.dat
curl -o $CRASHDIR/Country.mmdb -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/Country-lite.mmdb
curl -o $CRASHDIR/cn_ip.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv4.txt
curl -o $CRASHDIR/cn_ipv6.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv6.txt
```

## 四、 编辑 user.yaml 文件
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
log-level: error
allow-lan: true
unified-delay: true
tcp-concurrent: true
external-ui-url: "https://github.com/Zephyruso/zashboard/archive/gh-pages-cdn-fonts.zip"
profile: {store-selected: true, store-fake-ip: true}

geodata-mode: true

hosts:
  miwifi.com: [192.168.31.1, 127.0.0.1]
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  services.googleapis.cn: [services.googleapis.com]

dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter: ['geosite:trackerslist,private,cn']
  nameserver:
    - https://doh.pub/dns-query
    - quic://dns.alidns.com:853
  nameserver-policy: {'geosite:ads': [rcode://success]}
```

---

>`DNS` 私货
{: .prompt-tip }

注：
- 1. 本 `dns` 配置中，仅国外域名 `proxy` 走 `fake-ip`，直连域名（含国内域名 `cn`）走国内 DNS 解析，未知域名走国外 DNS 解析（有效解决了“心理 DNS 泄露问题”，详见《[搭载 mihomo 内核配置 DNS 不泄露教程-geodata 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsnoleaks-mihomo-geodata/)》），且配置 `ecs` 提高了兼容性
- 2. 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS（可进入光猫或路由器拨号页面查看，或者前往[公共 DNS 大全](https://toolb.cn/publicdns)查询）的 IP 段，如默认 DNS 为 `211.137.58.20`，可设置为 `211.137.58.0/24`

```yaml
hosts:
  miwifi.com: [192.168.31.1, 127.0.0.1]
  doh.pub: [1.12.12.12, 120.53.53.53, 2402:4e00::]
  dns.alidns.com: [223.5.5.5, 223.6.6.6, 2400:3200::1, 2400:3200:baba::1]
  dns.google: [8.8.8.8, 8.8.4.4, 2001:4860:4860::8888, 2001:4860:4860::8844]
  dns11.quad9.net: [9.9.9.11, 149.112.112.11, 2620:fe::11, 2620:fe::fe:11]

dns:
  enable: true
  ipv6: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  fake-ip-range: 28.0.0.0/8
  fake-ip-range6: fc00::/16
  fake-ip-filter-mode: whitelist
  fake-ip-filter: ['geosite:proxy']
  respect-rules: true
  nameserver:
    # 推荐将 `ecs` 设置为当前宽带运营商分配的默认 DNS 的 IP 段
    - 'https://dns.google/dns-query#ecs=211.137.58.0/24'
    - 'https://dns11.quad9.net/dns-query#ecs=211.137.58.0/24'
  proxy-server-nameserver:
    - https://doh.pub/dns-query
    - quic://dns.alidns.com:853
  direct-nameserver:
    - https://doh.pub/dns-query
    - quic://dns.alidns.com:853
  nameserver-policy: {'geosite:ads': [rcode://success]}
```

---

## 五、 添加定时任务
1. 连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o $CRASHDIR/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-arm64.upx >/dev/null 2>&1#更新mihomo内核
202#curl -o $CRASHDIR/GeoSite.dat -L https://ghfast.top/https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-geodata/geosite.dat && curl -o $CRASHDIR/GeoIP.dat -L https://ghfast.top/https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-geodata/geoip-lite.dat && curl -o $CRASHDIR/Country.mmdb -L https://ghfast.top/https://github.com/DustinWin/ruleset_geodata/releases/download/mihomo-geodata/Country-lite.mmdb >/dev/null 2>&1#更新geodata路由规则文件
203#curl -o $CRASHDIR/cn_ip.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv4.txt && curl -o $CRASHDIR/cn_ipv6.txt -L https://ghfast.top/https://github.com/DustinWin/geoip/releases/download/ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `sc`，进入 ShellCrash 配置脚本 → 5) 自动任务 → 1) 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-mihomo-geodata.png" alt="添加定时任务" width="60%" />

## 六、 设置部分
1. 设置可参考《[ShellCrash 搭载 mihomo 内核的配置-geodata 方案](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-geodata)》，此处只列举配置的不同之处
2. 进入 ShellCrash 配置脚本 → 2) 功能设置 → 2) DNS 设置 → 9) 修改 DNS 服务器，设置如下：  
<img src="/assets/img/dns/dns-null.png" alt="设置部分 2" width="60%" />

3. 进入主菜单 → 6) 配置文件管理 → a) 添加提供者 → 1) 设置名称或代号，如输入“mihomo”；后进入 2) 设置链接或路径，粘贴《[一](https://proxy-tutorials.dustinwin.us.kg/posts/share-shellcrash-mihomo-geodata/#%E4%B8%80-%E7%94%9F%E6%88%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-yaml-%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE)》中生成的 .yaml 配置文件直链，选择“a) 保存此提供者”
4. 进入 6) 配置文件管理 → c) 在线生成配置文件 → 6) 自定义浏览器 UA，选择“2) 不使用 UA”
5. 进入 6) 配置文件管理 → 1) mihomo，选择选择“e) 在线获取此配置文件”，启动服务即可

## 七、 访问 Dashboard 面板
打开 <http://miwifi.com:9999/ui/> 后，“主机”输入 `192.168.31.1`，“端口”输入 `9999`，点击“提交”即可访问 Dashboard 面板
