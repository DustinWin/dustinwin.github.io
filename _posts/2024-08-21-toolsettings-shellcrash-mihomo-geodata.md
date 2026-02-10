---
title: ShellCrash 搭载 mihomo 内核的配置-geodata 方案
description: 此配置搭载 mihomo 内核，包括 ShellCrash 的安装、导入路由规则文件、配置和使用方法
date: 2024-08-21 06:30:11 +0800
categories: [工具配置, ShellCrash 配置]
tags: [Clash, mihomo, ShellCrash, geodata, geosite, 基础, Router]
---

> 说明
{: .prompt-tip }
1. 本教程中的下载链接以 CPU 架构 ARM64 为例，若为别的 CPU 架构，请注意修改链接后缀
2. 查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是“linux aarch64”，就是搭载的 ARM64 架构

## 一、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo)
连接 SSH 后执行如下命令：

```shell
curl -o /tmp/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-arm64.upx
```

## 二、 导入路由规则文件
连接 SSH 后执行如下命令：

```shell
curl -o $CRASHDIR/GeoSite.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/geosite-all.dat
curl -o $CRASHDIR/Country.mmdb -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/Country.mmdb
```

## 三、 添加定时任务
连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴（快捷键 Ctrl+Shift+V）如下内容：  
注：
- 1. [ShellCrash](https://github.com/juewuy/ShellCrash) 安装路径为 `/data/ShellCrash`{: .filepath}
- 2. 须重启 ShellCrash 服务后生效

```shell
201#curl -o $CRASHDIR/CrashCore.upx -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-arm64.upx >/dev/null 2>&1#更新mihomo内核
202#curl -o $CRASHDIR/GeoSite.dat -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/geosite-all.dat && curl -o $CRASHDIR/Country.mmdb -L https://cdn.jsdelivr.net/gh/DustinWin/ruleset_geodata@mihomo-geodata/Country.mmdb >/dev/null 2>&1#更新geodata路由规则文件
203#curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/ >/dev/null 2>&1#更新zashboard面板
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 四、 设置部分
1. 连接 SSH 后执行命令 `sc` 即可打开 ShellCrash 配置脚本（若安装 ShellCrash 后自定义别名选择的是“2) 【 sc 】”）
2. 新手引导
  - ① 选择“1) 路由设备配置局域网透明代理”
  - ② 根据需要是否启用小内存模式（此处选择“0”）
  - ③ 启用推荐的自动任务配置
  - ④ 根据需要是否开始导入配置文件（此处选择“0”）
  - ⑤ 此时脚本会自动“发现可用的内核文件”，选择“1) 立即加载”，后选择“1) Mihomo(Meta)”  
    <img src="/assets/img/pin/import-mihomo.png" alt="ShellCrash 配置 1" width="60%" />

  - ⑥ 内核加载完成后根据需要是否保留相关数据库文件（此处选择“0) 不保留”）

3. 功能设置
  - ① 进入 2) DNS 设置 → 9) 修改 DNS 服务器，选择“4) 一键配置加密 DNS”
    - 注：推荐设置 DNS 分流（单独使用 ShellCrash 以及 ShellCrash 搭配 AdGuard Home 都适用），请看《[搭载 mihomo 内核进行 DNS 分流教程-geodata 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsbypass-mihomo-geodata)》

  - ② 进入 2) 功能设置 → 5) 启用域名嗅探，选择“1) 是”

4. 进入主菜单 → 4 启动设置，启用“1) 开机自启动”
5. 进入主菜单 → 5) 自动任务 → 1) 添加自动任务，可以看到末尾就有《[三](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-mihomo-geodata/#%E4%B8%89-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中添加的定时任务，输入对应的数字并回车后可设置执行条件
6. 进入主菜单 → 8) 工具与优化，选择“6) 小米设备软固化 SSH”（无需输入需要还原的 SSH 密码）
7. 进入 8) 工具与优化 → 8) 小米设备Tun模块修复，选择“1) 我已知晓，出现问题会自行承担！”
8. 导入配置文件
  - ① 进入主菜单 → 6) 配置文件管理 → a) 添加提供者 → 1) 设置名称或代号，如输入“mihomo”；后进入 2) 设置链接或路径，粘贴《[生成带有自定义策略组和规则的 mihomo 配置文件直链-geodata 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-mihomo-geodata)》中生成的 .yaml 配置文件直链，选择“a) 保存此提供者”
  - ② 进入 6) 配置文件管理 → c) 在线生成配置文件 → 6) 自定义浏览器 UA，选择“2) 不使用 UA”
  - ③ 进入 6) 配置文件管理 → 1) mihomo，选择选择“e) 在线获取此配置文件”

9. 访问 Dashboard 面板 <http://192.168.31.1:9999/ui/>，首次打开需要添加“主机”和“端口”，分别填入 `192.168.31.1` 和 `9999` 并点击“添加”即可  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="设置部分 2" width="60%" />

10. 进入 Dashboard 面板 → 代理 → 代理提供者，点击“转圈”图标，可手动更新节点

## 五、 在线 Dashboard 面板
在线 Dashboard 面板 [zashboard](https://github.com/Zephyruso/zashboard)，网址：<https://board.zash.run.place>
1. 若使用基于 [Chromium 项目](https://www.chromium.org/Home/)开发的浏览器打开网址去访问 Dashboard 面板时，以 [Chrome 浏览器](https://www.google.com/chrome/)为例，需要设置该网址域名“允许显示不安全内容”，进入设置 → 隐私和安全 → 网站设置 → 更多内容设置 → 不安全内容（或者直接在地址栏打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加网址域名 `board.zash.run.place`  
<img src="/assets/img/tools/chrome-setting-dashboard.png" alt="在线 Dashboard 面板 1" width="60%" />

2. 首次进入 <https://board.zash.run.place> 需要添加“主机”和“端口”，分别填入 `192.168.31.1` 和 `9999` 并点击“提交”即可访问 Dashboard 面板  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="在线 Dashboard 面板 2" width="60%" />
