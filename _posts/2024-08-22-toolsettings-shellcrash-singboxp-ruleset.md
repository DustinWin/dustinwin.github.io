---
title: ShellCrash 搭载 sing-boxp 内核的配置-ruleset 方案
description: 此配置搭载 sing-boxp 内核，包括 ShellCrash 的安装、配置和使用方法
date: 2024-08-22 17:34:34 +0800
categories: [工具配置, ShellCrash 配置]
tags: [sing-box, sing-boxp, ShellCrash, ruleset, rule_set, 基础, Router]
---

> 说明
{: .prompt-tip }
1. 本教程中的下载链接以 CPU 架构 ARMv8 为例，若为别的 CPU 架构，请注意修改链接后缀
2. 查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是“linux aarch64”，就是搭载的 ARMv8 架构

## 一、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)
**sing-box 内核 Linux 版下载链接后缀和 CPU 架构对应关系如下表：**

| CPU 架构     | AMD64   | ARMv5   | ARMv6   | ARMv7   | ARMv8&ARM64&AArch64 | mips-softfloat   | mipsle-softfloat   |
| ------------ | ------- | ------- | ------- | ------- | :-----------------: | ---------------- | ------------------ |
| **链接后缀** | `amd64` | `armv5` | `armv6` | `armv7` |       `armv8`       | `mips-softfloat` | `mipsle-softfloat` |

连接 SSH 后执行如下命令：

```shell
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz | tar -zx -C /tmp/
```

## 二、 添加定时任务
连接 SSH 后执行命令 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴（快捷键 Ctrl+Shift+V）如下内容：
- 注：ShellCrash 安装路径为 `/data/ShellCrash`{: .filepath}

```shell
201#curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz && $CRASHDIR/start.sh restart >/dev/null 2>&1#更新sing-box_PuerNya版内核
202#curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/ && $CRASHDIR/start.sh restart >/dev/null 2>&1#更新zashboard面板
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

## 三、 设置部分
1. 连接 SSH 后执行命令 `crash` 即可打开 ShellCrash 配置脚本
2. 新手引导
   - ① 选择 1 路由设备配置局域网透明代理
   - ② 启用推荐的自动任务配置
   - ③ 根据需要是否启用软固化（此处选择 `1`，解锁 SSH 时已成功启用软固化）
   - ④ 根据需要是否选择 1 确认导入配置文件（此处选择 `0`）
   - ⑤ 根据需要是否选择 1 立即启动服务（此处选择 `0`）
     - 注：强烈建议选择 `0`，待以下配置完成后，最后一步启动服务
   - ⑥ 此时脚本会自动“发现可用的内核文件”，选择 `1` 加载，后选择 3 Sing-Box-Puer 内核  
     <img src="/assets/img/pin/import-sing-boxp.png" alt="ShellCrash 配置 1" width="60%" />

   - ⑦ 内核加载完成后根据需要是否保留相关数据库文件（此处选择 `0`）
3. 进入主菜单 → 9 更新/卸载 → 7 切换安装源及安装版本，选择 b 切换至公测版 → 1 Jsdelivr_CDN源（推荐）
4. 进入主菜单 → 9 更新/卸载 → 4 安装本地 Dashboard 面板，选择 3 安装 MetaXD 面板（也可跳过此步，使用《[四](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxp-ruleset/#%E5%9B%9B-%E5%9C%A8%E7%BA%BF-dashboard-%E9%9D%A2%E6%9D%BF)》中的在线 Dashboard 面板）  
<img src="/assets/img/tools/install-dashboard.png" alt="安装面板" width="60%" />

5. 进入主菜单 → 2 内核功能设置，设置如下（推荐“混合模式”，其次“Tproxy 模式”，宽带在 300M 内推荐“Tun 模式”）：  
<img src="/assets/img/pin/mix-mix.png" alt="设置部分 1" width="60%" />

6. 进入主菜单 → 4 内核启动设置，选择 1 允许 ShellCrash 开机启动（若重启路由器后服务没有自动运行，可“设置自启延时”为 `30` 秒）
7. 进入主菜单 → 5 配置自动任务 → 1 添加自动任务，可以看到末尾就有《[二](https://proxy-tutorials.dustinwin.us.kg/posts/toolsettings-shellcrash-singboxp-ruleset/#%E4%BA%8C-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)》中添加的定时任务，输入对应的数字并回车后可设置执行条件
8. 进入主菜单 → 6 导入配置文件 → 2 在线获取完整配置文件，粘贴《[生成带有自定义出站和规则的 sing-boxp 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxp-ruleset)》中生成的 .json 配置文件直链，启动服务即可
9. 访问 Dashboard 面板 <http://192.168.31.1:9999/ui/>，首次打开需要添加“主机”和“端口”，分别填入 `192.168.31.1` 和 `9999` 并点击“添加”即可  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="设置部分 2" width="60%" />

10. 进入 Dashboard 面板 → 代理 → 代理提供者，点击“转圈”图标，可手动更新节点

## 四、 在线 Dashboard 面板
推荐使用在线 Dashboard 面板 [zashboard](https://github.com/Zephyruso/zashboard)，网址：<https://board.zash.run.place>
1. 若使用基于 [Chromium 项目](https://www.chromium.org/Home/)开发的浏览器打开网址去访问 Dashboard 面板时，以 [Chrome 浏览器](https://www.google.com/chrome/)为例，需要设置该网址域名“允许显示不安全内容”，进入设置 → 隐私和安全 → 网站设置 → 更多内容设置 → 不安全内容（或者直接在地址栏打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加网址域名 `board.zash.run.place`  
<img src="/assets/img/tools/chrome-setting-dashboard.png" alt="在线 Dashboard 面板 1" width="60%" />

2. 首次进入 <https://board.zash.run.place> 需要添加“主机”和“端口”，分别填入 `192.168.31.1` 和 `9999` 并点击“提交”即可访问 Dashboard 面板  
<img src="/assets/img/tools/192-9999-dashboard.png" alt="在线 Dashboard 面板 2" width="60%" />
