---
title: 分享 sing-boxp for Windows 采用 ruleset 方案的一套配置
description: 此方案适用于 sing-box，搭载 sing-boxp 内核，采用 `rule_set` 规则搭配 .srs 和 .json 规则集文件
date: 2024-08-22 20:01:28 +0800
categories: [分享配置, Windows]
tags: [sing-box, sing-boxp, Windows, ruleset, rule_set, 分享]
---

## 声明
1. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
2. 此方案采用**裸核**的方式运行，更加精简

## 一、 生成配置文件 .json 文件直链
具体方法请参考《[生成带有自定义出站和规则的 sing-boxp 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/link-singboxp-ruleset)》，贴一下我使用的配置：

```json
{
  "outbound_providers": [
    {
      "tag": "🛫 我的机场",
      "type": "remote",
      // 修改为你的 Clash 订阅链接
      "download_url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport.yaml",
      "download_interval": "24h",
      "download_ua": "clash.meta",
      "includes": [ "🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸" ],
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m"
    },
    {
      "tag": "🆓 免费订阅",
      "type": "remote",
      // 修改为你的 sing-box 订阅链接
      "download_url": "https://example.com/xxx/xxx",
      "path": "./providers/free.json",
      "download_interval": "24h",
      "download_ua": "sing-box",
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m"
    }
  ],
  "log": { "level": "error", "timestamp": true },
  "dns": {
    "hosts": {
      "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ],
      "miwifi.com": [ "192.168.31.1" ]
    },
    "servers": [
      { "tag": "dns_block", "address": "rcode://success" },
      { "tag": "dns_direct", "address": [ "https://doh.pub/dns-query", "https://dns.alidns.com/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://cloudflare-dns.com/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": [ "any" ], "server": "dns_direct" },
      { "clash_mode": [ "Direct" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": [ "Global" ], "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "server": "dns_block", "disable_cache": true, "rewrite_ttl": 0 },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 },
      { "fallback_rules": [ { "rule_set": [ "cnip" ], "server": "dns_direct" }, { "match_all": true, "server": "dns_fakeip", "rewrite_ttl": 1 } ], "server": "dns_direct", "allow_fallthrough": true }
    ],
    "final": "dns_proxy",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "lazy_cache": true,
    "reverse_mapping": true,
    "mapping_override": true,
    "fakeip": {
      "enabled": true,
      "inet4_range": "198.18.0.0/15",
      "inet6_range": "fc00::/18",
      "exclude_rule": { "rule_set": [ "fakeip-filter", "private" ] }
    }
  },
  "inbounds": [
    { "tag": "tun-in", "type": "tun", "interface_name": "sing-box", "address": [ "172.19.0.1/30", "fdfe:dcba:9876::1/126" ], "auto_route": true, "strict_route": true, "stack": "mixed", "sniff": true }
  ],
  "outbounds": [
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "🐟 漏网之鱼", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📈 网络测试", "type": "selector", "outbounds": [ "🎯 全球直连", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "🤖 人工智能", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🎮 游戏服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🪟 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇬 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇨🇳 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇨🇳 直连 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🧱 代理域名", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📲 电报信息", "type": "selector", "outbounds": [ "🚀 节点选择" ] },
    { "tag": "🖥️ 直连软件", "type": "selector", "outbounds": [ "🎯 全球直连" ] },
    { "tag": "🔒 私有网络", "type": "selector", "outbounds": [ "🎯 全球直连" ] },
    { "tag": "🛑 广告拦截", "type": "selector", "outbounds": [ "REJECT" ] },
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "REJECT", "type": "block" },
    { "tag": "DIRECT", "type": "direct" },
    { "tag": "GLOBAL", "type": "selector", "outbounds": [ "DIRECT", "REJECT", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点", "🆚 vless 节点" ] },
    { "tag": "dns-out", "type": "dns" },
    // 若没有单个出站节点，须删除所有 `🆚 vless 节点` 相关内容
    {
      "tag": "🆚 vless 节点",
      "type": "vless",
      "server": "example.com",
      "server_port": 443,
      "uuid": "{uuid}",
      "network": "tcp",
      "tls": { "enabled": true, "server_name": "example.com", "insecure": false },
      "transport": { "type": "ws", "path": "/?ed=2048", "headers": { "Host": "example.com" } }
    },
    { "tag": "🇭🇰 香港节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇭🇰" ] },
    { "tag": "🇹🇼 台湾节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇹🇼" ] },
    { "tag": "🇯🇵 日本节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇯🇵" ] },
    { "tag": "🇰🇷 韩国节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇰🇷" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇸🇬" ] },
    { "tag": "🇺🇸 美国节点", "type": "urltest", "tolerance": 50, "providers": [ "🛫 我的机场" ], "includes": [ "🇺🇸" ] },
    { "tag": "🆓 免费节点", "type": "urltest", "tolerance": 100, "providers": [ "🆓 免费订阅" ] }
  ],
  "route": {
    "rules": [
      { "protocol": [ "dns" ], "outbound": "dns-out" },
      { "clash_mode": [ "Direct" ], "outbound": "DIRECT" },
      { "clash_mode": [ "Global" ], "outbound": "GLOBAL" },
      { "rule_set": [ "applications" ], "outbound": "🖥️ 直连软件" },
      { "rule_set": [ "private" ], "outbound": "🔒 私有网络" },
      { "rule_set": [ "ads" ], "outbound": "🛑 广告拦截" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "🪟 微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "🍎 苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "🇬 谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "🎮 游戏服务" },
      { "rule_set": [ "ai" ], "outbound": "🤖 人工智能" },
      { "rule_set": [ "networktest" ], "outbound": "📈 网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "🧱 代理域名" },
      { "rule_set": [ "cn" ], "outbound": "🇨🇳 直连域名" },
      { "rule_set": [ "telegramip" ], "outbound": "📲 电报信息", "skip_resolve": true },
      { "rule_set": [ "privateip" ], "outbound": "🔒 私有网络", "skip_resolve": true },
      { "rule_set": [ "cnip" ], "outbound": "🇨🇳 直连 IP" }
    ],
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/fakeip-filter-lite.srs"
      },
      {
        "tag": "applications",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/applications.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/applications.srs"
      },
      {
        "tag": "private",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/private.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/private.srs"
      },
      {
        "tag": "ads",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ads.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/ads.srs"
      },
      {
        "tag": "microsoft-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/microsoft-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/microsoft-cn.srs"
      },
      {
        "tag": "apple-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/apple-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/apple-cn.srs"
      },
      {
        "tag": "google-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/google-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/google-cn.srs"
      },
      {
        "tag": "games-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/games-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/games-cn.srs"
      },
      {
        "tag": "ai",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ai.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/ai.srs"
      },
      {
        "tag": "networktest",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/networktest.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/networktest.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/proxy.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cn.srs"
      },
      {
        "tag": "telegramip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/telegramip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/telegramip.srs"
      },
      {
        "tag": "privateip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/privateip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/privateip.srs"
      },
      {
        "tag": "cnip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cnip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cnip.srs"
      }
    ],
    "final": "🐟 漏网之鱼",
    "auto_detect_interface": true,
    "concurrent_dial": true
  },
  "experimental": {
    "cache_file": { "enabled": true },
    "clash_api": {
      "external_controller": "127.0.0.1:9090",
      "external_ui": "ui",
      "external_ui_download_url": "https://github.com/Zephyruso/zashboard/releases/latest/download/dist-cdn-fonts.zip",
      "secret": "",
      "default_mode": "Rule"
    }
  }
}
```

## 二、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)和配置文件并启动 sing-box
### 1. 导入内核和配置文件
- ① 编辑本文文档，粘贴如下内容：  
  注：
  - ➊ 将第《一》步生成的配置文件 .json 文件直链替换下面命令中的 `{.json 配置文件直链}`
  - ➋ 或删除此条命令，直接进入 *%PROGRAMFILES%\sing-box* 文件夹，新建 config.json 文件并粘贴配置内容

  ```shell
  rem 导入 sing-box 内核和配置文件
  md "%PROGRAMFILES%\sing-box" "%PROGRAMFILES%\sing-box\ui" "%PROGRAMFILES%\sing-box\providers" "%PROGRAMFILES%\sing-box\ruleset"
  takeown /f "%PROGRAMFILES%\sing-box" /a /r /d y
  icacls "%PROGRAMFILES%\sing-box" /inheritance:r
  icacls "%PROGRAMFILES%\sing-box" /remove[:g] "TrustedInstaller"
  icacls "%PROGRAMFILES%\sing-box" /remove[:g] "CREATOR OWNER"
  icacls "%PROGRAMFILES%\sing-box" /remove[:g] "ALL APPLICATION PACKAGES"
  icacls "%PROGRAMFILES%\sing-box" /remove[:g] "所有受限制的应用程序包"
  icacls "%PROGRAMFILES%\sing-box" /grant[:r] SYSTEM:(OI)(CI)F
  icacls "%PROGRAMFILES%\sing-box" /grant[:r] Administrators:(OI)(CI)F
  icacls "%PROGRAMFILES%\sing-box" /grant[:r] Users:(OI)(CI)F
  curl -o "%PROGRAMFILES%\sing-box\sing-box.exe" -L https://ghgo.xyz/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-windows-amd64v3.exe
  curl -o "%PROGRAMFILES%\sing-box\config.json" -L {.json 配置文件直链}
  echo 导入 sing-box 内核和配置文件成功
  pause
  ```

- ② 另存为 .bat 文件，右击并选择“以管理员身份运行”

### 2. 启动 sing-box
- ① 编辑本文文档，粘贴如下内容：

  ```shell
  cd "%PROGRAMFILES%\sing-box"
  start /min sing-box.exe run
  ```

- ② 另存为 run.bat 文件并复制到 *%PROGRAMFILES%\sing-box* 文件夹中
- ③ 右击 run.bat 文件并选择“以管理员身份运行”即可  
  小窍门：
  - ➊ 右击 run.bat 文件并选择“发送到桌面快捷方式”
  - ➋ 右击快捷方式并点击“属性” -> “高级”，勾选“以管理员身份运行”并“确定”
  - ➌ 若想开机启动 sing-box，可搜索“Windows 添加任务计划”教程自行添加

## 三、 更新 sing-box PuerNya 版内核和配置文件
编辑本文文档，粘贴如下内容：  
注：
- ① 将第《一》步生成的配置文件 .json 文件直链替换下面命令中的 `{.json 配置文件直链}`
- ② 或者删除此条命令，直接进入 *%PROGRAMFILES%\sing-box* 文件夹，修改 config.json 文件内的配置内容

```shell
@echo off
rem 下载 sing-box 相关文件
curl -o "%USERPROFILE%\Downloads\sing-box.exe" -L https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-windows-amd64v3.exe
curl -o "%USERPROFILE%\Downloads\config.json" -L {.json 配置文件直链}
echo 下载 sing-box 相关文件成功

rem 结束 sing-box 相关进程
taskkill /f /t /im sing-box*
echo 结束 sing-box 相关进程成功

rem 更新 sing-box 内核和配置文件
copy /y "%USERPROFILE%\Downloads\sing-box.exe" "%PROGRAMFILES%\sing-box"
copy /y "%USERPROFILE%\Downloads\config.json" "%PROGRAMFILES%\sing-box"
echo 更新 sing-box 内核和配置文件成功

rem 更新 sing-box 内核和配置文件成功，等待 10 秒启动 sing-box 服务
timeout /t 10 /nobreak
cd "%PROGRAMFILES%\sing-box"
start /min sing-box.exe run
echo 启动 sing-box 服务成功
pause
```

另存为 .bat 文件，右击并选择“以管理员身份运行”

## 四、 访问 Dashboard 面板
.json 文件已配置 [zashboard 面板](https://github.com/Zephyruso/zashboard)  
打开 http://127.0.0.1:9090/ui/ 后可直接点击“提交”，即可访问 Dashboard 面板  
<img src="/assets/img/share/127-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />
