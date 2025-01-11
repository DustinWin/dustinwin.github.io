---
title: ShellCrash 搭载 mihomo 内核本地配置自定义策略组和规则-geodata 方案
description: 此方案适用于 mihomo，搭载 mihomo 内核，采用 `GEOSITE` 和 `GEOIP` 规则搭配 geosite.dat 和 geoip.dat（或 Country.mmdb）路由规则文件
date: 2024-08-21 08:27:14 +0800
categories: [工具配置, ShellCrash 配置]
tags: [Clash, mihomo, ShellCrash, geodata, geosite, 进阶, 本地, Router]
---

> 说明
{: .prompt-tip }
1. 本教程只适用于 [ShellCrash](https://github.com/juewuy/ShellCrash)
2. 本教程**仅适合白名单模式**（没有命中规则的网络流量统统使用代理，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户）
3. 本教程最终效果媲美《[生成带有自定义策略组和规则的 mihomo 配置文件直链-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/link-mihomo-geodata)》（策略组更直观，操作更方便）
4. 若仅配置自定义策略组和规则，可直接跳过第《二》步
5. 所有步骤完成后，请连接 SSH 执行命令 `$CRASHDIR/start.sh restart` 后生效
6. 推荐使用 [Visual Studio Code](https://code.visualstudio.com/Download) 等专业编辑器来修改配置文件

## 一、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
可参考《[ShellCrash 搭载 mihomo 内核的配置-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/toolsettings-shellcrash-mihomo-geodata)》里的步骤《一、二》进行操作

## 二、 导入配置文件
1. 进入 ShellCrash → 6 导入配置文件 → 1 在线生成 meta 配置文件 → 4 选取在线配置规则模版，选择 4 [ACL4SSR](https://acl4ssr-sub.github.io) 极简版（适合自建节点）  
<img src="/assets/img/tools/subscribe-easy.png" alt="导入配置文件" width="60%" />

2. 进入 ShellCrash → 6 导入配置文件 → 1 在线生成 meta 配置文件，输入订阅链接后回车，再输入 `1` 并回车即可

## 三、 自定义策略组和规则
### 1. 自定义 others.yaml（用于编写自定义的锚点、入站、代理集合 `proxy-providers`、子规则 `sub-rules` 和 script 脚本等功能）
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/others.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
## 代理集合（获取机场订阅链接内的所有节点）
proxy-providers:
  🛫 我的机场 1:
    type: http
    ## 机场订阅链接，使用 Clash 链接
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport1.yaml
    interval: 86400
    ## 初步筛选需要的节点，可有效减轻路由器压力，支持正则表达式，不筛选可删除此配置项
    filter: "(?i)港|hk|hongkong|hong kong|台|tw|taiwan|日本|jp|japan|新|sg|singapore|美|us|unitedstates|united states"
    ## 初步排除不需要的节点，支持正则表达式，若不排除可删除此配置项
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      ## 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 我的机场 1-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 我的机场 1-"
      ## 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 我的机场 1”；推荐有多个机场时使用
      additional-suffix: "-🛫 我的机场 1"

  🛫 我的机场 2:
    type: http
    url: "https://example.com/xxx/xxx&flag=clash"
    path: ./proxies/airport2.yaml
    interval: 86400
    filter: "(?i)港|hk|hongkong|hong kong|台|tw|taiwan|日本|jp|japan|新|sg|singapore|美|us|unitedstates|united states"
    exclude-filter: "高倍|直连|×10"
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 600
    override:
      ## 为节点名称添加固定前缀，如节点名称原为“香港节点”会变成“🛫 我的机场 2-香港节点”；推荐有多个机场时使用
      additional-prefix: "🛫 我的机场 2-"
      ## 为节点名称添加固定后缀，如节点名称原为“香港节点”会变成“香港节点-🛫 我的机场 2”；推荐有多个机场时使用
      additional-suffix: "-🛫 我的机场 2"
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 2. 自定义 proxies.yaml（用于添加自定义出站代理 `proxies`）
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/proxies.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：  
注：
- ① 此处以“vless”节点类型为例，其它节点类型写法可参考[通用字段](https://wiki.metacubex.one/config/proxies)
- ② 必须在 proxy-groups.yaml 里添加自定义的节点才可以正常选择和使用

```yaml
- name: 🆓 免费节点
  ## 节点类型
  type: vless
  ## 代理节点服务器（域名/IP）
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
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 3. 自定义 proxy-groups.yaml（用于添加自定义策略组 `proxy-groups`）
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/proxy-groups.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
## 策略组

## 手动选择国家或地区节点；根据“国家或地区策略组”名称对 `proxies` 值进行增删改，须一一对应
- name: 🈯 节点指定
  type: select
  proxies:
    - 🇭🇰 香港节点
    - 🇹🇼 台湾节点
    - 🇯🇵 日本节点
    - 🇸🇬 新加坡节点
    - 🇺🇸 美国节点
    ## 添加 proxies.yaml 中的自定义节点
    - 🆓 免费节点

## 选择 `🎯 全球直连` 为测试本地网络（运营商网络速度和 IPv6 支持情况），可选择其它节点用于测试机场节点速度和 IPv6 支持情况
- name: 📈 网络测试
  type: select
  proxies:
    - 🎯 全球直连
    - 🇭🇰 香港节点
    - 🇹🇼 台湾节点
    - 🇯🇵 日本节点
    - 🇰🇷 韩国节点
    - 🇸🇬 新加坡节点
    - 🇺🇸 美国节点
    ## 添加 proxies.yaml 中的自定义节点
    - 🆓 免费节点

- name: 🤖 人工智能
  type: select
  proxies:
    - 🇭🇰 香港节点
    - 🇹🇼 台湾节点
    - 🇯🇵 日本节点
    - 🇸🇬 新加坡节点
    - 🇺🇸 美国节点

- name: 🎮 游戏服务
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🪟 微软服务
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🇬 谷歌服务
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🍎 苹果服务
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🇨🇳 直连域名
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🇨🇳 直连 IP
  type: select
  proxies:
    - 🎯 全球直连
    - 🈯 节点指定

- name: 🧱 代理域名
  type: select
  proxies:
    - 🈯 节点指定
    - 🎯 全球直连

- name: 📲 电报信息
  type: select
  proxies:
    - 🈯 节点指定

- name: 🔒 私有网络
  type: select
  proxies:
    - 🎯 全球直连

- name: 🛑 广告拦截
  type: select
  proxies:
    - REJECT

## ----------------国家或地区策略组---------------------

## 自动选择节点，即按照 url 测试结果使用延迟最低的节点
- name: 🇭🇰 香港节点
  type: url-test
  ## 测试后容差大于 50ms 才会切换到延迟低的那个节点
  tolerance: 50
  include-all-providers: true
  ## 筛选出“香港”节点，支持正则表达式
  filter: "(?i)港|hk|hongkong|hong kong"

- name: 🇹🇼 台湾节点
  type: url-test
  tolerance: 50
  include-all-providers: true
  filter: "(?i)台|tw|taiwan"

- name: 🇯🇵 日本节点
  type: url-test
  tolerance: 50
  include-all-providers: true
  filter: "(?i)日本|jp|japan"

- name: 🇸🇬 新加坡节点
  type: url-test
  tolerance: 50
  include-all-providers: true
  filter: "(?i)新|sg|singapore"

- name: 🇺🇸 美国节点
  type: url-test
  tolerance: 50
  include-all-providers: true
  filter: "(?i)美|us|unitedstates|united states"
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 4. 自定义 rules.yaml（用于添加自定义规则 `rules`）
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/rules.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
## 规则

## 自定义规则优先放前面
- GEOSITE,private,🔒 私有网络
- GEOSITE,ads,🛑 广告拦截
- GEOSITE,microsoft-cn,🪟 微软服务
- GEOSITE,apple-cn,🍎 苹果服务
- GEOSITE,google-cn,🇬 谷歌服务
- GEOSITE,games-cn,🎮 游戏服务
- GEOSITE,ai,🤖 人工智能
- GEOSITE,networktest,📈 网络测速
- GEOSITE,proxy,🧱 代理域名
- GEOSITE,cn,🇨🇳 直连域名
- GEOIP,telegram,📲 电报信息,no-resolve
- GEOIP,private,🔒 私有网络,no-resolve
- GEOIP,cn,🇨🇳 直连 IP
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车  
**贴一张面板效果图（举个例子：我手动选择 `🇹🇼 台湾节点` 策略组，而该策略组是将机场内所有台湾节点按照 url 测试结果自动选择延迟最低的台湾节点）：**  
<img src="/assets/img/tools/show-dashboard.png" alt="面板效果图" width="60%" />

## 四、 修改策略组或规则
**举例：我的机场包含有 2 个节点，分别是新加坡节点和日本节点，我想让 [Netflix](https://www.netflix.com/) 自动选择延迟最低的新加坡节点，[哔哩哔哩](https://www.bilibili.com)可以手动选择日本任一节点**
> 一定要保证缩进对齐！一定要保证缩进对齐！一定要保证缩进对齐！
{: .prompt-warning }

### 1. 修改 proxy-groups.yaml 文件
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/proxy-groups.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
## 策略组

## 默认选择新加坡节点
- name: 🎥 奈飞视频
  type: select
  proxies:
    - 🇸🇬 新加坡节点

## 默认选择日本节点，也可切换到直连
- name: 📺 哔哩哔哩
  type: select
  proxies:
    - 🇯🇵 日本节点
    - 🎯 全球直连

## 自动选择延迟最低的新加坡节点；容差大于 50ms 才会切换到延迟低的那个节点
- name: 🇸🇬 新加坡节点
  type: url-test
  tolerance: 50
  include-all-providers: true
  filter: "(?i)(新|sg|singapore)"

## 手动选择日本任一节点
- name: 🇯🇵 日本节点
  type: select
  include-all-providers: true
  filter: "(?i)日本|jp|japan"
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 2. 修改 rules.yaml 文件
连接 SSH 后执行命令 `vi $CRASHDIR/yamls/rules.yaml`，按一下 Ins 键（Insert 键），**优先在最上方**粘贴如下内容：

```yaml
## 规则

## 自定义规则优先放前面
- GEOSITE,netflix,🎥 奈飞视频
- GEOIP,netflix,🎥 奈飞视频,no-resolve
- GEOSITE,bilibili,📺 哔哩哔哩
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
> 若有其它需求，可导入 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 路由规则文件，并分别进入 [MetaCubeX/meta-rules-dat/meta/geo](https://github.com/MetaCubeX/meta-rules-dat/tree/meta/geo) 的 *geosite* 和 *geoip* 目录搜索关键字，通过能够搜索到的关键字来编写策略组和规则
{: .prompt-tip }

## 五、 添加小规则
仅添加特定网址走直连或走代理，连接 SSH 后执行命令 `vi $CRASHDIR/yamls/rules.yaml`，按一下 Ins 键（Insert 键），在**最上方**粘贴如下内容：  
注：
- ① 以下内容只是举例，请根据自身需要进行增删改
- ② 其它规则请参考《[mihomo Wiki](https://wiki.metacubex.one/config/rules)》

```yaml
## 规则

## 以 googleapis.cn 为后缀（包括 googleapis.cn）的所有域名走代理
- DOMAIN-SUFFIX,googleapis.cn,🈯 节点指定
## 与哔哩哔哩相关的所有域名走直连
- GEOSITE,bilibili,DIRECT
## 含有 ipv6 关键字的所有域名走直连
- DOMAIN-KEYWORD,ipv6,DIRECT
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
