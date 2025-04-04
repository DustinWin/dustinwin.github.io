---
title: ShellCrash 搭载 mihomo 内核的配置-geodata 方案
description: 此配置搭载 mihomo 内核，包括 ShellCrash 的安装、导入路由规则文件、配置和使用方法
date: 2024-08-21 06:30:11 +0800
categories: [工具配置, ShellCrash 配置]
tags: [Clash, mihomo, ShellCrash, geodata, geosite, 基础, Router]
---

> 说明
{: .prompt-tip }
1. 本教程中的下载链接以 CPU 架构 ARMv8 为例，若为别的 CPU 架构，请注意修改链接后缀
2. 查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是“linux aarch64”，就是搭载的 ARMv8 架构

## 一、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)
**mihomo 内核 Linux 版下载链接后缀和 CPU 架构对应关系如下表：**

| CPU 架构     | AMD64   | AMD64v3   | ARMv5   | ARMv6   | ARMv7   | ARMv8&ARM64&AArch64 | mips-softfloat   | mipsle-hardfloat   | mipsle-softfloat   |
| ------------ | ------- | --------- | ------- | ------- | ------- | :-----------------: | ---------------- | ------------------ | ------------------ |
| **链接后缀** | `amd64` | `amd64v3` | `armv5` | `armv6` | `armv7` |       `armv8`       | `mips-softfloat` | `mipsle-hardfloat` | `mipsle-softfloat` |

连接 SSH 后执行如下命令：

```shell
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-armv8.tar.gz | tar -zx -C /tmp/
```

## 二、 导入路由规则文件
连接 SSH 后执行如下命令：

```shell
curl -o $CRASHDIR/GeoSite.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat
curl -o $CRASHDIR/Country.mmdb -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country.mmdb
```

## 三、 添加定时任务
连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴（快捷键 Ctrl+Shift+V）如下内容：
- 注：ShellCrash 安装路径为 `/data/ShellCrash`{: .filepath}

```shell
201#curl -o /data/ShellCrash/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-armv8.tar.gz && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新mihomo内核
202#curl -o /data/ShellCrash/GeoSite.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/geosite-all.dat && curl -o /data/ShellCrash/Country.mmdb -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo/Country.mmdb && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新geodata路由规则文件
203#curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/metacubexd.tar.gz | tar -zx -C $CRASHDIR/ui/ && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新metacubexd面板
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 四、 设置部分
1. 连接 SSH 后执行命令 `crash` 即可打开 ShellCrash 配置脚本
2. 新手引导
   - ① 选择 1 路由设备配置局域网透明代理
   - ② 启用推荐的自动任务配置
   - ③ 根据需要是否启用软固化（此处选择 `1`，解锁 SSH 时已成功启用软固化）
   - ④ 根据需要是否选择 1 确认导入配置文件（此处选择 `0`）
   - ⑤ 根据需要是否选择 1 立即启动服务（此处选择 `0`）
     - 注：强烈建议选择 `0`，待以下配置完成后，最后一步启动服务
   - ⑥ 此时脚本会自动“发现可用的内核文件”，选择 `1` 加载，后选择 3 Clash-Meta 内核  
     <img src="/assets/img/pin/import-mihomo.png" alt="ShellCrash 配置 1" width="60%" />

   - ⑦ 内核加载完成后根据需要是否保留相关数据库文件（此处选择 `0`）
3. 进入主菜单 → 9 更新/卸载 → 7 切换安装源及安装版本，选择 b 切换至公测版 → 1 Jsdelivr_CDN源（推荐）
4. 进入主菜单 → 9 更新/卸载 → 4 安装本地 Dashboard 面板，选择 3 安装 MetaXD 面板（也可跳过此步，直接使用《[五](https://proxy-tutorials.dustinwin.top/posts/toolsettings-shellcrash-mihomo-geodata/#%E4%BA%94-%E5%9C%A8%E7%BA%BF-dashboard-%E9%9D%A2%E6%9D%BF)》中的在线 Dashboard 面板）
5. 进入主菜单 → 2 内核功能设置，设置如下（有“Tproxy 模式”就选“Tproxy 模式”，否则推荐选“混合模式”，宽带在 300M 内推荐选 Tun 模式）：  
<img src="/assets/img/pin/tproxy-mix.png" alt="设置部分 1" width="60%" />

6. 进入主菜单 → 4 内核启动设置，选择 1 允许 ShellCrash 开机启动（若重启路由器后服务没有自动运行，可“设置自启延时”为 `30` 秒）
7. 进入主菜单 → 5 配置自动任务 → 1 添加自动任务，可以看到末尾就有《[三](https://proxy-tutorials.dustinwin.top/posts/toolsettings-shellcrash-mihomo-geodata/#%E4%B8%89-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中添加的定时任务，输入对应的数字并回车后可设置执行条件
8. 进入主菜单 → 7 内核进阶设置，启用 4 启用域名嗅探
9.  进入主菜单 → 6 导入配置文件 → 2 在线获取完整配置文件，粘贴《[生成带有自定义策略组和规则的 mihomo 配置文件直链-geodata 方案](https://proxy-tutorials.dustinwin.top/posts/link-mihomo-geodata)》中生成的 .yaml 配置文件直链，启动服务即可
10. 访问 Dashboard 面板 <http://192.168.31.1:9999/ui/>，首次打开需要添加“后端地址”，输入 `http://192.168.31.1:9999` 并点击“添加”即可  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="设置部分 2" width="60%" />

11. 进入 Dashboard 面板 → 代理 → 代理提供者，点击“转圈图标”（Update），也可手动更新节点

## 五、 在线 Dashboard 面板
推荐使用在线 Dashboard 面板 [metacubexd](https://github.com/metacubex/metacubexd)，网址：<https://metacubex.github.io/metacubexd/>
1. 若使用基于 [Chromium 项目](https://www.chromium.org/Home/)开发的浏览器打开网址去访问 Dashboard 面板时，以 [Chrome 浏览器](https://www.google.com/chrome/)为例，需要设置该网址域名“允许显示不安全内容”，进入设置 → 隐私和安全 → 网站设置 → 更多内容设置 → 不安全内容（或者直接在地址栏打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加网址域名 `metacubex.github.io`  
<img src="/assets/img/tools/chrome-setting-dashboard.png" alt="在线 Dashboard 面板 1" width="60%" />

2. 首次进入 <https://metacubex.github.io/metacubexd/> 需要添加“后端地址”，输入 `http://192.168.31.1:9999` 并点击“添加”即可访问 Dashboard 面板  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="在线 Dashboard 面板 2" width="60%" />
