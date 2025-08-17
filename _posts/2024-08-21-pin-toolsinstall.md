---
title: ShellCrash 和 AdGuard Home 快速安装教程
description: 此教程包括 ShellCrash、AdGuard Home、mihomo 内核、sing-box 内核和 Dashboard 面板的安装方法
date: 2024-08-21 17:13:12 +0800
categories: [置顶]
tags: [ShellCrash, AdGuard Home, mihomo, sing-box, sing-boxp, 安装, Dashboard]
pin: true
---

> 说明
{: .prompt-tip }
1. 本教程中 **[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 安装目录为 `/data/AdGuardHome`{: .filepath}**
2. 本教程中的下载链接以 CPU 架构 ARMv8 为例，请注意修改链接后缀
3. 查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是“linux aarch64”，就是搭载的 ARMv8 架构
4. 以下所有命令均可全部复制后直接粘贴执行（若出现无法下载的情况，可更换[下载源](https://proxy-tutorials.dustinwin.us.kg/about/#%E5%AF%B9%E4%B8%8B%E8%BD%BD%E6%BA%90%E7%9A%84%E8%AF%B4%E6%98%8E)）

## 一、 安装 [ShellCrash](https://github.com/juewuy/ShellCrash)
### 1. 本地安装
连接 SSH 后执行如下命令：

```shell
curl -o /tmp/ShellCrash.tar.gz -L https://cdn.jsdelivr.net/gh/juewuy/ShellCrash@master/bin/ShellCrash.tar.gz
mkdir -p /tmp/SC_tmp/ && tar -zxf '/tmp/ShellCrash.tar.gz' -C /tmp/SC_tmp/ && source /tmp/SC_tmp/init.sh
```

### 2. 在线安装
连接 SSH 后执行如下命令：

```shell
export url='https://cdn.jsdelivr.net/gh/juewuy/ShellCrash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```

## 二、 导入 [mihomo 内核](https://github.com/MetaCubeX/mihomo) 或 [sing-box 内核](https://github.com/SagerNet/sing-box)
**mihomo 内核和 sing-box 内核 Linux 版下载链接后缀和 CPU 架构对应关系如下表：**

| CPU 架构     | AMD64   | AMD64v3   | ARMv5   | ARMv6   | ARMv7   | ARMv8&ARM64&AArch64 | mips-softfloat   | mipsle-hardfloat   | mipsle-softfloat   |
| ------------ | ------- | --------- | ------- | ------- | ------- | :-----------------: | ---------------- | ------------------ | ------------------ |
| **链接后缀** | `amd64` | `amd64v3` | `armv5` | `armv6` | `armv7` |       `armv8`       | `mips-softfloat` | `mipsle-hardfloat` | `mipsle-softfloat` |

### 1. 首次导入
连接 SSH 后执行如下命令：

```shell
# mihomo 内核 Meta 版
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-armv8.tar.gz | tar -zx -C /tmp/ && crash
# mihomo 内核 Alpha 版
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-alpha-linux-armv8.tar.gz | tar -zx -C /tmp/ && crash
# sing-box 内核 PuerNya 版
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz | tar -zx -C /tmp/ && crash
# sing-box 内核 Release 版
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-release-linux-armv8.tar.gz | tar -zx -C /tmp/ && crash
# sing-box 内核 Dev 版
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-dev-linux-armv8.tar.gz | tar -zx -C /tmp/ && crash
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择对应的内核类型

### 2. 升级导入（ShellCrash → 9 更新/卸载 → 2 切换内核文件，内核版本不会刷新）
连接 SSH 后执行如下命令：

```shell
# mihomo 内核 Meta 版
curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-meta-linux-armv8.tar.gz && $CRASHDIR/start.sh restart
# mihomo 内核 Alpha 版
curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/mihomo/mihomo-alpha-linux-armv8.tar.gz && $CRASHDIR/start.sh restart
# sing-box 内核 PuerNya 版
curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz && $CRASHDIR/start.sh restart
# sing-box 内核 Release 版
curl -o $CRASHDIR/CrashCore.tar.gz -L https://cdn.jsdelivr.net/gh/DustinWin/proxy-tools/@sing-box/sing-box-release-linux-armv8.tar.gz && $CRASHDIR/start.sh restart
# sing-box 内核 Dev 版
curl -o $CRASHDIR/CrashCore.tar.gz -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/sing-box/sing-box-dev-linux-armv8.tar.gz && $CRASHDIR/start.sh restart
```

## 三、 安装 Dashboard 面板
**Dashboard 面板对应文件名和网址关系如下表：**

| 面板名称        | 文件名              | 网址                                      |
| --------------- | ------------------- | ----------------------------------------- |
| yacd 面板       | `yacd.tar.gz`       | <https://yacd.haishan.me>                 |
| Yacd-meta 面板  | `Yacd-meta.tar.gz`  | <https://yacd.metacubex.one>              |
| metacubexd 面板 | `metacubexd.tar.gz` | <https://metacubex.github.io/metacubexd/> |
| zashboard 面板  | `zashboard.tar.gz`  | <https://board.zash.run.place>            |

连接 SSH 后执行如下命令：

```shell
curl -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/Dashboard/zashboard.tar.gz | tar -zx -C $CRASHDIR/ui/ && $CRASHDIR/start.sh restart
```

- 注：若使用基于 [Chromium 项目](https://www.chromium.org/Home/)开发的浏览器打开网址去访问 Dashboard 面板时，以 [Chrome 浏览器](https://www.google.com/chrome/)为例，需要设置该网址域名“允许显示不安全内容”。方法如下：  
进入设置 → 隐私和安全 → 网站设置 → 更多内容设置 → 不安全内容（或者直接在地址栏打开 chrome://settings/content/insecureContent 进行设置），在“允许显示不安全内容”内添加网址域名如：`board.zash.run.place`

## 四、 安装 AdGuard Home
### 1. 安装 AdGuard Home
**AdGuard Home Linux 版 CPU 架构和链接后缀对应关系如下表：**

| CPU 架构     | AMD64   | ARMv5   | ARMv6   | ARMv7   | ARMv8   | mips-softfloat   | mipsle-softfloat   |
| ------------ | ------- | ------- | ------- | ------- | ------- | ---------------- | ------------------ |
| **链接后缀** | `amd64` | `armv5` | `armv6` | `armv7` | `armv8` | `mips-softfloat` | `mipsle-softfloat` |

连接 SSH 后执行如下命令：

```shell
mkdir -p /data/AdGuardHome/
# AdGuard Home Release 版
curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_release_linux_armv8
# AdGuard Home Beta 版
curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_beta_linux_armv8
chmod +x /data/AdGuardHome/AdGuardHome
/data/AdGuardHome/AdGuardHome -s install
/data/AdGuardHome/AdGuardHome -s start
# 将所有发往 53 端口的流量重定向到本地的 5353 端口
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
# 添加开机启动
cat <<EOF >> /data/auto_ssh/auto_ssh.sh

sleep 10s
/data/AdGuardHome/AdGuardHome -s install
/data/AdGuardHome/AdGuardHome -s start
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
EOF
```

### 2. 升级 AdGuard Home
- 注：留意链接后缀是否与 CPU 架构匹配

连接 SSH 后执行如下命令：

```shell
# AdGuard Home Release 版
curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_release_linux_armv8
# AdGuard Home Beta 版
curl -o /data/AdGuardHome/AdGuardHome -L https://ghfast.top/https://github.com/DustinWin/proxy-tools/releases/download/AdGuardHome/AdGuardHome_beta_linux_armv8
/data/AdGuardHome/AdGuardHome -s restart
```
