---
title: ShellCrash 搭载 sing-boxp 内核的配置-geodata 方案
description: 此方案适用于 sing-box，搭载 sing-boxp 内核，采用 `geosite` 和 `geoip` 规则搭配 geosite.db 和 geoip.db 路由规则文件
date: 2024-08-21 23:19:34 +0800
categories: [工具配置, ShellCrash 配置]
tags: [sing-box, sing-boxp, ShellCrash, geodata, geosite, 基础, Router]
---

## 说明
1. 本教程中的下载链接以 CPU 架构 ARMv8 为例，若为别的 CPU 架构，请注意修改链接后缀
2. 查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是 `linux aarch64`，就是搭载的 ARMv8 架构

## 一、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)
**sing-box 内核 Linux 版下载链接后缀和 CPU 架构对应关系如下表：**

| CPU 架构     | AMD64   | AMD64v3   | ARMv5   | ARMv6   | ARMv7   | ARMv8&ARM64&AArch64 | mips-softfloat   | mipsle-hardfloat   | mipsle-softfloat   |
| ------------ | ------- | --------- | ------- | ------- | ------- | :-----------------: | ---------------- | ------------------ | ------------------ |
| **链接后缀** | `amd64` | `amd64v3` | `armv5` | `armv6` | `armv7` |       `armv8`       | `mips-softfloat` | `mipsle-hardfloat` | `mipsle-softfloat` |

连接 SSH 后执行如下命令：

```shell
curl -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools@sing-box/sing-box-puernya-linux-armv8.tar.gz | tar -zx -C /tmp/
```

## 二、 导入路由规则文件
连接 SSH 后执行如下命令：

```shell
curl -o $CRASHDIR/geosite.db -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/geosite.db
curl -o $CRASHDIR/geoip.db -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/geoip.db
```

## 三、 添加定时任务
连接 SSH 后执行 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴（快捷键 Ctrl+Shift+V）如下内容：
- 注：[ShellCrash](https://github.com/juewuy/ShellCrash) 安装路径为 */data/ShellCrash*

```shell
201#curl -o /data/ShellCrash/CrashCore.tar.gz -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools@sing-box/sing-box-puernya-linux-armv8.tar.gz && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新sing-box_PuerNya版内核
202#curl -o /data/ShellCrash/geosite.db -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/geosite.db && curl -o /data/ShellCrash/geoip.db -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@sing-box/geoip.db && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新geodata路由规则文件
203#curl -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools@Dashboard/metacubexd.tar.gz | tar -zx -C $CRASHDIR/ui/ && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新metacubexd面板
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 四、 设置部分
1. 连接 SSH 后执行 `crash` 命令打开 ShellCrash 配置脚本  
首次打开会进入新手引导，路由设备配置局域网透明代理（此处选择 1）  
启用推荐的自动任务配置  
根据需要是否启用软固化（此处选择 1）  
根据需要是否选择 1 确认导入配置文件（此处选择 0）  
根据需要是否选择 1 立即启动服务（此处选择 0）  
输入 0 回车可返回到上级菜单（下同）  
1. 此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 5 Sing-Box-Puer 内核
2. 内核加载完成后根据需要是否保留相关数据库文件（此处选择 0）
3. 进入主菜单 -> 9 更新/卸载 -> 7 切换安装源及安装版本，选择 b 切换至公测版 -> 1 Jsdelivr_CDN源（推荐）
4. 进入主菜单 -> 9 更新/卸载 -> 4 安装本地 Dashboard 面板，选择 3 安装 MetaXD 面板（也可跳过此步，使用第《五》步中的在线 Dashboard 面板）
5. 进入主菜单 -> 2 内核功能设置，设置如下（有“Tproxy 模式”就选“Tproxy 模式”，否则推荐选“混合模式”，宽带在 300M 内推荐选 Tun 模式）：  
<img src="/assets/img/tools/tproxy-mix.png" alt="设置部分 1" width="60%" />

1. 进入主菜单 -> 4 内核启动设置，选择 1 允许 ShellCrash 开机启动（若重启路由器后服务没有自动运行，可“设置自启延时”为 30 秒）
2. 进入主菜单 -> 5 配置自动任务 -> 1 添加自动任务，可以看到末尾就有第《三》步添加的定时任务，输入对应的数字并回车后可设置执行条件 
3. 进入主菜单 -> 6 导入配置文件 -> 2 在线获取完整配置文件，粘贴《[生成带有自定义出站和规则的 sing-box 配置文件直链-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/link-singbox-geodata)》中生成的 .json 配置文件直链，启动服务即可
4.  访问 Dashboard 面板 http://192.168.31.1:9999/ui ，首次打开需要添加“后端地址”，输入 `http://192.168.31.1:9999` 并点击“添加”即可  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="设置部分 2" width="60%" />

1.  进入 Dashboard 面板 -> 代理 -> 代理提供者，点击“转圈图标”（Update），也可手动更新节点

## 五、 在线 Dashboard 面板
推荐使用在线 Dashboard 面板 [metacubexd](https://github.com/metacubex/metacubexd)，访问地址：<https://metacubex.github.io/metacubexd/>
1. 需要设置该网站“允许不安全内容”，以 Chrome 浏览器为例，进入设置 -> 隐私和安全 -> 网站设置 -> 更多内容设置 -> 不安全内容（或者直接打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加 `metacubex.github.io`  
<img src="/assets/img/tools/chrome-setting-dashboard.png" alt="在线 Dashboard 面板 1" width="60%" />

2. 首次进入 <https://metacubex.github.io/metacubexd/> 需要添加“后端地址”，输入 `http://192.168.31.1:9999` 并点击“添加”即可访问 Dashboard 面板  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="在线 Dashboard 面板 2" width="60%" />
