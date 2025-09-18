---
title: ShellCrash 搭载 sing-boxp 内核本地配置自定义出站和规则-ruleset 方案
description: 此教程搭载 sing-boxp 内核，可通过修改本地配置文件的方式来自定义出站和规则 <code>rule_set</code>
date: 2024-08-22 18:10:32 +0800
categories: [工具配置, ShellCrash 配置]
tags: [sing-box, sing-boxp, ShellCrash, ruleset, rule_set, 进阶, 本地, Router]
---

> 说明
{: .prompt-tip }
1. 本教程只适用于 [ShellCrash](https://github.com/juewuy/ShellCrash) 
2. 本教程**仅适合白名单模式**（没有命中规则的网络流量统统使用代理，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户）
3. 本教程最终效果媲美《[生成带有自定义出站和规则的 sing-boxp 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxp-ruleset)》（出站分组更直观，操作更方便）
4. 若仅配置自定义出站和规则，可直接跳过《[二](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxp-local-ruleset/#%E4%BA%8C-%E5%AF%BC%E5%85%A5%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)》
5. 出站提供者 outbound_providers.json、出站 outbounds.json 和规则 route.json 为合并模式（在基础配置上新增）
6. 所有步骤完成后，请连接 SSH 后执行命令 `$CRASHDIR/start.sh restart` 生效
7. 推荐使用 [Visual Studio Code](https://code.visualstudio.com/Download) 等专业编辑器来修改配置文件

## 一、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box)
可参考《[ShellCrash 搭载 sing-boxp 内核的配置-ruleset 方案/导入 sing-box PuerNya 版内核](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxp-ruleset/#%E4%B8%80-%E5%AF%BC%E5%85%A5-sing-box-PuerNya-%E7%89%88%E5%86%85%E6%A0%B8)》里的步骤进行操作

## 二、 导入配置文件
1. 进入 ShellCrash → 6 导入配置文件 → 1 在线生成 singbox 配置文件 → 4 选取在线配置规则模版，选择 4 [ACL4SSR](https://acl4ssr-sub.github.io) 极简版（适合自建节点）  
<img src="/assets/img/tools/subscribe-easy.png" alt="导入配置文件" width="60%" />

2. 进入 ShellCrash → 6 导入配置文件 → 1 在线生成 singbox 配置文件，输入订阅链接后回车，再输入 `1` 并回车即可

## 三、 自定义出站和规则
### 1. 自定义出站提供者 outbound_providers.json（用于添加自定义出站提供者 `outbound_providers`）
- ① 连接 SSH 后执行命令 `mkdir -p $CRASHDIR/providers/`
  - 注：因出站提供者 `outbound_providers` 配置的 `path` 路径中含有文件夹 `providers`{: .filepath}，须手动新建此文件夹才能使 .json 订阅文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）

- ② 继续执行命令 `vi $CRASHDIR/jsons/outbound_providers.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：

  ```json
  {
    // 出站提供者（获取机场订阅链接内的所有节点）
    "outbound_providers": [
      {
        "tag": "🛫 机场订阅 1",
        "type": "remote",
        // 机场订阅链接，使用 Clash 链接
        "download_url": "https://example.com/xxx/xxx&flag=clash",
        "path": "./providers/airport1.yaml",
        "download_interval": "24h",
        "download_ua": "clash.meta",
        // 初步筛选需要的节点，可有效减轻路由器压力，支持正则表达式，若不筛选可删除此配置项
        "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)" ],
        // 初步排除不需要的节点，支持正则表达式，若不排除可删除此配置项
        "excludes": "高倍|直连|×10",
        "healthcheck_url": "https://www.gstatic.com/generate_204",
        "healthcheck_interval": "10m",
        "outbound_override": {
          // 设置出站标签的前缀，如出站标签原为“香港节点”会变成“🛫 机场订阅 1-香港节点”；推荐有多个机场时使用
          "tag_prefix": "🛫 机场订阅 1-",
          // 设置出站标签的后缀，如出站标签原为“香港节点”会变成“香港节点-🛫 机场订阅 1”；推荐有多个机场时使用
          "tag_suffix": "-🛫 机场订阅 1"
        }
      },
      {
        "tag": "🛫 机场订阅 2",
        "type": "remote",
        // 机场订阅链接，使用 sing-box 链接
        "download_url": "https://example.com/xxx/xxx",
        "path": "./providers/airport2.json",
        "download_interval": "24h",
        "download_ua": "sing-box",
        "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong|🇹🇼|台|tw|taiwan|tai wan|🇯🇵|日|jp|japan|🇸🇬|新|sg|singapore|🇺🇸|美|us|unitedstates|united states)" ],
        "excludes": "高倍|直连|×10",
        "healthcheck_url": "https://www.gstatic.com/generate_204",
        "healthcheck_interval": "10m",
        "outbound_override": {
          // 设置出站标签的前缀，如出站标签原为“香港节点”会变成“🛫 机场订阅 2-香港节点”；推荐有多个机场时使用
          "tag_prefix": "🛫 机场订阅 2-",
          // 设置出站标签的后缀，如出站标签原为“香港节点”会变成“香港节点-🛫 机场订阅 2”；推荐有多个机场时使用
          "tag_suffix": "-🛫 机场订阅 2"
        }
      }
    ]
  }
  ```

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 2. 自定义出站 outbounds.json（用于添加自定义出站 `outbounds`）
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/outbounds.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：

```json
{
  // 出站
  "outbounds": [
    // 手动选择国家或地区节点；根据“国家或地区出站”的名称对 `outbounds` 值进行增删改，须一一对应
    { "tag": "🈯 节点指定", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    // 选择`🎯 全球直连`为测试本地网络（运营商网络速度和 IPv6 支持情况），可选择其它节点用于测试机场节点速度和 IPv6 支持情况
    { "tag": "📈 网络测试", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    { "tag": "🤖 人工智能", "type": "selector", "outbounds": [ "🈯 节点指定", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点"] },
    { "tag": "📋 Trackerslist", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🎮 游戏服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🪟 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🇬 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🌍 国外媒体", "type": "selector", "outbounds": [ "🈯 节点指定", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🎮 游戏平台", "type": "selector", "outbounds": [ "🈯 节点指定", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🛡️ 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🀄️ 直连 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🈯 节点指定" ] },
    { "tag": "🧱 代理域名", "type": "selector", "outbounds": [ "🈯 节点指定", "🎯 全球直连" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🈯 节点指定", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    { "tag": "🛑 广告域名", "type": "selector", "outbounds": [ "🔴 全球拦截", "🎯 全球直连" ] },
    { "tag": "🔴 全球拦截", "type": "selector", "outbounds": [ "REJECT" ] },

    // 单个出站节点（以 vless 为例）
    {
      "tag": "🆓 免费节点",
      "type": "vless",
      "server": "example.com",
      "server_port": 443,
      "uuid": "{uuid}",
      "network": "tcp",
      "tls": { "enabled": true, "server_name": "example.com", "insecure": false },
      "transport": { "type": "ws", "path": "/?ed=2048", "headers": { "Host": "example.com" } }
    },

    // -------------------- 国家或地区出站 --------------------
    // 自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 50ms 才会切换到延迟低的那个节点；筛选出“香港”节点，支持正则表达式
    { "tag": "🇭🇰 香港节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇭🇰|港|hk|hongkong|hong kong)" ] },
    { "tag": "🇹🇼 台湾节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇹🇼|台|tw|taiwan|tai wan)" ] },
    { "tag": "🇯🇵 日本节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇯🇵|日|jp|japan)" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇸🇬|新|sg|singapore)" ] },
    { "tag": "🇺🇸 美国节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇺🇸|美|us|unitedstates|united states)" ] }
  ]
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 3. 自定义规则 route.json（用于添加自定义路由和规则 `route`）
- ① 连接 SSH 后执行命令 `mkdir -p $CRASHDIR/ruleset/`
  - 注：因 `route.rule_set` 规则集配置的 `path` 路径中含有文件夹 `ruleset`{: .filepath}，须手动新建此文件夹才能使 .srs 规则集文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）

- ② 继续执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：

  ```json
  {
    // 路由
    "route": {
      // 规则
      "rules": [
        // 自定义规则优先放前面
        { "rule_set": [ "private" ], "outbound": "🎯 全球直连" },
        { "rule_set": [ "ads" ], "outbound": "🛑 广告域名" },
        { "rule_set": [ "trackerslist" ], "outbound": "📋 Trackerslist" },
        { "rule_set": [ "microsoft-cn" ], "outbound": "🪟 微软服务" },
        { "rule_set": [ "apple-cn" ], "outbound": "🍎 苹果服务" },
        { "rule_set": [ "google-cn" ], "outbound": "🇬 谷歌服务" },
        { "rule_set": [ "games-cn" ], "outbound": "🎮 游戏服务" },
        { "rule_set": [ "media" ], "outbound": "🌍 国外媒体" },
        { "rule_set": [ "games" ], "outbound": "🎮 游戏平台" },
        { "rule_set": [ "ai" ], "outbound": "🤖 人工智能" },
        { "rule_set": [ "networktest" ], "outbound": "📈 网络测试" },
        { "rule_set": [ "proxy" ], "outbound": "🧱 代理域名" },
        { "rule_set": [ "cn" ], "outbound": "🛡️ 直连域名" },
        { "rule_set": [ "privateip" ],  "outbound": "🎯 全球直连", "skip_resolve": true },
        { "rule_set": [ "cnip" ], "outbound": "🀄️ 直连 IP" },
        { "rule_set": [ "mediaip" ], "outbound": "🌍 国外媒体", "skip_resolve": true },
        { "rule_set": [ "gamesip" ], "outbound": "🎮 游戏平台", "skip_resolve": true },
        { "rule_set": [ "telegramip" ], "outbound": "📲 电报消息", "skip_resolve": true }
      ],
      // 规则集（binary 文件每天自动更新）
      "rule_set": [
        {
          "tag": "private",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/private.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/private.srs"
        },
        {
          "tag": "ads",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/ads.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/ads.srs"
        },
        {
          "tag": "trackerslist",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/trackerslist.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/trackerslist.srs"
        },
        {
          "tag": "microsoft-cn",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/microsoft-cn.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/microsoft-cn.srs"
        },
        {
          "tag": "apple-cn",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/apple-cn.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/apple-cn.srs"
        },
        {
          "tag": "google-cn",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/google-cn.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/google-cn.srs"
        },
        {
          "tag": "games-cn",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/games-cn.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/games-cn.srs"
        },
        {
          "tag": "media",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/media.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/media.srs"
        },
        {
          "tag": "games",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/games.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/games.srs"
        },
        {
          "tag": "ai",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/ai.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/ai.srs"
        },
        {
          "tag": "networktest",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/networktest.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/networktest.srs"
        },
        {
          "tag": "proxy",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/proxy.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/proxy.srs"
        },
        {
          "tag": "cn",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/cn.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/cn.srs"
        },
        {
          "tag": "privateip",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/privateip.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/privateip.srs"
        },
        {
          "tag": "cnip",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/cnip.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/cnip.srs"
        },
        {
          "tag": "mediaip",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/mediaip.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/mediaip.srs"
        },
        {
          "tag": "gamesip",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/gamesip.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/gamesip.srs"
        },
        {
          "tag": "telegramip",
          "type": "remote",
          "format": "binary",
          "path": "./ruleset/telegramip.srs",
          "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/telegramip.srs"
        }
      ],
      "concurrent_dial": true
    }
  }
  ```

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 四、 修改出站或规则
**举例：我的机场包含有 2 个节点，分别是新加坡节点和日本节点，我想让 [Netflix](https://www.netflix.com/) 自动选择延迟最低的新加坡节点，[哔哩哔哩](https://www.bilibili.com)可以手动选择日本任一节点**  
> 一定要保证缩进对齐！一定要保证缩进对齐！一定要保证缩进对齐！
{: .prompt-warning }

### 1. 修改 outbounds.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/outbounds.json`，按一下 Ins 键（Insert 键），编辑如下内容并粘贴：

```json
{
  // 出站
  "outbounds": [
    // 默认选择新加坡节点
    { "tag": "🎥 奈飞视频", "type": "selector", "outbounds": [ "🇸🇬 新加坡节点" ] },
    // 默认选择日本节点，也可切换到直连
    { "tag": "📺 哔哩哔哩", "type": "selector", "outbounds": [ "🇯🇵 日本节点", "🎯 全球直连" ] },

    // 自动选择延迟最低的新加坡节点；容差大于 50ms 才会切换到延迟低的那个节点
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "(?i)(🇸🇬|新|sg|singapore)" ] },
    // 手动选择日本任一节点
    { "tag": "🇯🇵 日本节点", "type": "selector", "use_all_providers": true, "includes": [ "(?i)(🇯🇵|日|jp|japan)" ] }
  ]
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

### 2. 修改 route.json 文件
连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），优先在最上方编辑如下内容并粘贴：

```json
{
  // 路由
  "route": {
    // 规则
    "rules": [
      // 自定义规则优先放前面
      { "rule_set": [ "netflix", "netflixip" ], "outbound": "🎥 奈飞视频", "skip_resolve": true },
      { "rule_set": [ "bilibili" ], "outbound": "📺 哔哩哔哩" }
    ],
    // 规则集（binary 文件每天自动更新）
    "rule_set": [
      {
        "tag": "netflix",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/netflix.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/netflix.srs"
      },
      {
        "tag": "netflixip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/netflixip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/netflixip.srs"
      },
      {
        "tag": "bilibili",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/bilibili.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/bilibili.srs"
      }
    ]
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
> 若有其它需求，可进入 [MetaCubeX/meta-rules-dat/sing](https://github.com/MetaCubeX/meta-rules-dat/tree/sing) 搜索关键字，通过能够搜索到的关键字来编写出站和规则（推荐使用“*.srs”文件，`route.rule_set` 内须配置 `"format": "binary"`）
{: .prompt-tip }

## 五、 添加小规则
仅添加特定网址走直连或走代理，连接 SSH 后执行命令 `vi $CRASHDIR/jsons/route.json`，按一下 Ins 键（Insert 键），在**最上方**粘贴如下内容：  
注：
- ① 以下内容只是举例，请根据自身需要进行增删改
- ② 其它规则请参考《[sing-box Wiki](https://sing-box.sagernet.org/zh/configuration/route/rule)》

```json
{
  // 路由
  "route": {
    // 规则
    "rules": [
      // 以 googleapis.cn 为后缀（包括 googleapis.cn）的所有域名走代理
      { "domain_suffix": [ "googleapis.cn" ], "outbound": "🈯 节点指定" },
      // 与哔哩哔哩相关的所有域名走直连
      { "rule_set": [ "bilibili" ], "outbound": "DIRECT" },
      // 含有 ipv6 关键字的所有域名走直连
      { "domain_keyword": [ "ipv6" ], "outbound": "DIRECT" }
    ],
    // 规则集（binary 文件每天自动更新）
    "rule_set": [
      {
        "tag": "bilibili",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/bilibili.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset-compatible/bilibili.srs"
      }
    ]
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
