---
title: 全网最详细的解锁 SSH ShellCrash 搭载 sing-boxr 内核搭配 AdGuard Home 安装和配置教程
description: 此教程使用 ShellCrash 搭载 sing-boxr 内核并搭配 AdGuard Home 作下游，配有详细图文
date: 2024-08-21 22:27:53 +0800
categories: [置顶]
tags: [sing-box, sing-boxr, ShellCrash, AdGuard Home, 解锁, SSH]
pin: true
---

> 说明
{: .prompt-tip }
1. 本教程基于 REDMI AX6000 [官方固件](https://www1.miwifi.com/miwifi_download.html) v1.0.70 版，[ShellCrash](https://github.com/juewuy/ShellCrash) v1.9.4 版，[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) v0.108.0 版编写
2. 恢复 SSH，安装 ShellCrash 和 AdGuard Home 的方法也适用于其它已解锁 SSH 的路由器
3. 安装 [sing-box reF1nd 版内核](https://github.com/reF1nd/sing-box) 内核和 AdGuard Home 时须注意路由器 CPU 架构，查看 CPU 架构可连接 SSH 后执行命令 `uname -ms`，若执行结果是“linux aarch64”，就下载 armv8 或 arm64 版安装包；若是其它架构请下载相匹配的安装包
4. ShellCrash 和 AdGuard Home 中所有没有提到的配置保持默认即可
5. 使用本教程时，不建议开启 ShellCrash 配置脚本 → 2 功能设置 → 3 透明路由流量过滤 → 2 过滤局域网设备，因不经过内核的设备在访问 `漏网之鱼` 域名时会遇到无法访问的情况
6. ShellCrash 和 AdGuard Home 快速安装方法请看《[ShellCrash 和 AdGuard Home 快速安装教程](https://proxy-tutorials.dustinwin.us.kg/posts/pin-toolsinstall)》

## 一、 资源下载
打包下载：<https://dustinwinvip.lanzoum.com/b01qd6p3a>  
密码：`zyxz`  
注：
- ① 没有对文件进行任何处理，请自行操作使用
- ② 不保证实时更新，想用新版请安装后自行升级
- ③ 版本信息请查看打包文件内的 Readme.txt 文本

### 1. ShellCrash
官方下载：<https://raw.githubusercontent.com/juewuy/ShellCrash/master/ShellCrash.tar.gz>

### 2. sing-box reF1nd 版内核
第三方下载：<https://github.com/DustinWin/proxy-tools/releases/tag/sing-box>  
下载 sing-box-ref1nd-dev-linux-armv8.tar.gz 文件

### 3. PuTTY
官方下载：<https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html>  
下载 putty-64bit-[version]-installer.msi 文件

### 4. AdGuard Home
官方下载：<https://github.com/AdguardTeam/AdGuardHome/releases>  
下载 AdGuardHome_linux_arm64.tar.gz 文件

### 5. UPX
官方下载：<https://github.com/upx/upx/releases>  
下载 upx-[version]-win64.zip 文件

### 6. WinSCP
官方下载：<https://winscp.net/eng/downloads.php>  
下载 WinSCP-[version]-Setup.exe 文件

## 二、 添加 SSH 支持
### 1. 给 Windows 操作系统添加 SSH 支持（任选一）
- ① 启用 Telnet 客户端
  - ➊ 进入设置 → 系统 → 可选功能 → 更多 Windows 功能，勾选“Telnet Client”并点击“确定”  
    <img src="/assets/img/pin/add-windows-telnet-1.png" alt="启用 Telnet 客户端 1" width="60%" />

  - ➋ 点击“让 Windows 更新为你下载文件”  
    <img src="/assets/img/pin/add-windows-telnet-2.png" alt="启用 Telnet 客户端 2" width="60%" />

- ② 添加 OpenSSH 客户端和 OpenSSH 服务器
  - ➊ 进入设置 → 系统 → 可选功能 → 查看功能，搜索“ssh”，在搜索结果中勾选“OpenSSH 客户端”和“OpenSSH 服务器”并点击“下一步”  
    <img src="/assets/img/pin/add-windows-openssh-1.png" alt="启用 OpenSSH 服务器 1" width="60%" />

  - ➋ 点击“添加”  
    <img src="/assets/img/pin/add-windows-openssh-2.png" alt="启用 OpenSSH 服务器 2" width="60%" />

- ③ 连接 Telnet

> 在成功完成《[三、 2](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#2-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF-telnet)》后才能进行此操作
{: .prompt-warning }

  - ➊ 以管理员身份运行 PowerShell 或 CMD，执行命令 `telnet 192.168.31.1`
    - 注：首次登录不需要用户名和密码，解锁或恢复 SSH 后用户名为 `root`，密码为《[三、 3](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#3-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF%E5%B9%B6%E5%9B%BA%E5%8C%96-ssh)》中设置的登录密码

    <img src="/assets/img/pin/connect-telnet-windows-1.png" alt="连接 Telnet 1" width="60%" />

  - ➋ 输入密码 `12345678`（输入时密码不可见，下同）并回车，显示“ARE U OK”表示成功连接 Telnet  
    <img src="/assets/img/pin/connect-telnet-windows-2.png" alt="连接 Telnet 2" width="60%" />

- ④ 连接 SSH

> 在成功完成《[三、 3](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#3-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF%E5%B9%B6%E5%9B%BA%E5%8C%96-ssh)》后才能进行此操作
{: .prompt-warning }

  - ➊ 以管理员身份运行 PowerShell 或 CMD，执行命令 `ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.31.1` 以允许 SSH 客户端接受“ssh-rsa”密钥，输入 `yes` 并回车
    - 注：若当前电脑登录过 SSH，后路由器经过重新解锁或恢复 SSH，须进入 `C:\Users\[用户名]\.ssh`{: .filepath} 文件夹，删除“known_hosts”文件，否则登录会报错

    <img src="/assets/img/pin/connect-ssh-windows-1.png" alt="连接 SSH 1" width="60%" />

  - ➋ 输入密码 `12345678` 并回车  
    <img src="/assets/img/pin/connect-ssh-windows-2.png" alt="连接 SSH 2" width="60%" />

  - ➌ 显示“ARE U OK”表示成功登录 SSH  
    <img src="/assets/img/pin/connect-ssh-windows-3.png" alt="连接 SSH 3" width="60%" />

### 2. 通过 SSH 工具添加 SSH 支持（任选一）
- ① 连接 Telnet

> 在成功完成《[三、 2](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#2-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF-telnet)》后才能进行此操作
{: .prompt-warning }

  - ➊ 安装 PuTTY 并打开，按图输入和选择，点击“Open”即可成功连接 Telnet
    <img src="/assets/img/pin/connect-telnet-1.png" alt="连接 Telnet 1" width="60%" />

  - ➋ 显示“ARE U OK”表示成功登录 Telnet  
    <img src="/assets/img/pin/connect-telnet-2.png" alt="连接和添加 Telnet 2" width="60%" />

- ② 连接 SSH

> 在成功完成《[三、 3](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#3-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF%E5%B9%B6%E5%9B%BA%E5%8C%96-ssh)》后才能进行此操作
{: .prompt-warning }

  - ➊ 打开 PuTTY，然后按图输入，点击“Open”即可成功连接 SSH
    <img src="/assets/img/pin/connect-ssh-1.png" alt="连接和添加 SSH 1" width="60%" />

  - ➋ “login as”输入“root”并回车，“password”为解锁或恢复 SSH 时设置的密码，输入后再次回车，显示“ARE U OK”表示成功连接 SSH  
    <img src="/assets/img/pin/connect-ssh-2.png" alt="连接和添加 SSH 2" width="60%" />

### 3. 通过 WinSCP 连接路由器文件管理
> 在成功完成《[三、 3](https://proxy-tutorials.dustinwin.us.kg/posts/pin-shellcrashadguardhome-singboxr/#3-%E6%B0%B8%E4%B9%85%E5%BC%80%E5%90%AF%E5%B9%B6%E5%9B%BA%E5%8C%96-ssh)》后才能进行此操作
{: .prompt-warning }

- ① 安装 WinSCP 并打开，“文件协议”选择“SCP”，其它按图输入，“密码”为 SSH 登录密码，点击“保存”后再点击“登录”  
  <img src="/assets/img/pin/login-winscp.png" alt="通过 WinSCP 连接路由器文件管理 1" width="60%" />

- ② 左侧为电脑本地文件，右侧为路由器文件  
  <img src="/assets/img/pin/show-winscp.png" alt="通过 WinSCP 连接路由器文件管理 2" width="60%" />

---

>配置免密码连接 SSH 和 WinSCP
{: .prompt-tip }
1. 配置免密码连接 SSH
- ① 打开 PuTTYgen，直接点击“Generate”（期间鼠标必须在此窗口内不停移动）  
  <img src="/assets/img/pin/puttygen-generate.png" alt="生成 key" width="60%" />

- ② 生成后复制完整的“Key”值备用，点击“Save private key”（可在“Key comment”输入 `root@192.168.31.1 - REDMI AX6000`）  
  <img src="/assets/img/pin/puttygen-save.png" alt="保存 key" width="60%" />

- ③ “保存”文件到 `C:\Users\[用户名]\.ssh\rsa_key.ppk`{: .filepath} 中
- ④ 打开 PuTTY，进入 Connection → SSH → Auth → Credentials，点击“Private key file for authentication”的“Browser”，定位到 `C:\Users\[用户名]\.ssh\rsa_key.ppk`{: .filepath} 文件并“打开”
- ⑤ 进入 Connections → Data，“Auto-login username”输入 `root`  
  <img src="/assets/img/pin/putty-setting-1.png" alt="设置登录用户名" width="60%" />

- ⑥ 进入 Session，按图输入后，**先点击“Default Settings”，后点击“Save”**  
  <img src="/assets/img/pin/putty-setting-2.png" alt="保存配置" width="60%" />

2. 配置免密码连接 WinSCP
- ① 打开 WinSCP，进入标签页 → 站点 → 站点管理器，点击“编辑”，删除密码后点击“高级”
- ② 进入 SSH → 验证，点击“密钥文件”的“...”图标，定位到 `C:\Users\[用户名]\.ssh\rsa_key.ppk`{: .filepath} 文件并“打开”，点击“确定”，再点击“保存”

3. 导入 Key 并加入开机启动  
连接 SSH，执行如下命令：
- 注：将《1. ②》中复制的 Key 值替换下面命令中的 `{key}`

```shell
echo "{key}" > /data/auto_ssh/authorized_keys
chmod 600 /data/auto_ssh/authorized_keys
ln -sf /data/auto_ssh/authorized_keys /etc/dropbear/
echo -e "\nln -sf /data/auto_ssh/authorized_keys /etc/dropbear/" >> /data/auto_ssh/auto_ssh.sh
```

---

## 三、 解锁 SSH
### 1. 开启调试模式
- ① 进入路由器管理页面 <http://192.168.31.1>，登录后复制地址栏中的 stok 值  
  <img src="/assets/img/pin/copy-stok.png" alt="复制 stok 值" width="60%" />

- ② 将复制的 stok 值替换如下网址的 `{stok}` 并访问：

  ```text
  http://192.168.31.1/cgi-bin/luci/;stok={stok}/api/misystem/set_sys_time?timezone=%20%27%20%3B%20zz%3D%24%28dd%20if%3D%2Fdev%2Fzero%20bs%3D1%20count%3D2%202%3E%2Fdev%2Fnull%29%20%3B%20printf%20%27%A5%5A%25c%25c%27%20%24zz%20%24zz%20%7C%20mtd%20write%20-%20crash%20%3B%20
  ```

  网页内容显示 `{"code":0}` 表示成功开启调试模式
- ③ 继续将复制的 stok 值替换如下网址的 `{stok}` 并访问：

  ```text
  http://192.168.31.1/cgi-bin/luci/;stok={stok}/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
  ```

  网页内容显示 `{"code":0}`，此时路由器会重启

### 2. 永久开启 Telnet
- ① 重启完成后进入路由器管理页面并登录，再次复制 stok 值
- ② 将复制的 stok 值替换如下网址的 `{stok}` 并访问：

  ```text
  http://192.168.31.1/cgi-bin/luci/;stok={stok}/api/misystem/set_sys_time?timezone=%20%27%20%3B%20bdata%20set%20telnet_en%3D1%20%3B%20bdata%20set%20ssh_en%3D1%20%3B%20bdata%20set%20uart_en%3D1%20%3B%20bdata%20commit%20%3B%20
  ```

  网页内容显示 `{"code":0}` 表示成功设置 Bdata 永久开启 Telnet
- ③ 继续将复制的 stok 值替换如下网址的 `{stok}` 并访问：

  ```text
  http://192.168.31.1/cgi-bin/luci/;stok={stok}/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
  ```

  网页内容显示 `{"code":0}`，此时路由器会再次重启

**开启 Telnet 成功！**

### 3. 永久开启并固化 SSH
- ① 连接 Telnet，执行如下命令：
  - 注：第一行命令是将 Telnet 和 SSH 登录密码设置为 `12345678`，可自定义

  ```shell
  echo -e '12345678\n12345678' | passwd root
  nvram set telnet_en=1
  nvram set ssh_en=1
  nvram set uart_en=1
  nvram set boot_wait=on
  nvram commit
  /etc/init.d/dropbear enable & /etc/init.d/dropbear start
  mkdir -p /data/auto_ssh
  curl -o /data/auto_ssh/auto_ssh.sh -L https://cdn.jsdelivr.net/gh/lemoeo/AX6S@main/auto_ssh.sh
  chmod +x /data/auto_ssh/auto_ssh.sh
  /data/auto_ssh/auto_ssh.sh install
  uci set system.@system[0].timezone='CST-8'
  uci set system.@system[0].webtimezone='CST-8'
  uci set system.@system[0].timezoneindex='2.84'
  uci commit
  mtd erase crash
  reboot
  ```

  <img src="/assets/img/pin/unlock-ssh.png" alt="永久开启并固化 SSH" width="60%" />

- ② 最后一行 `reboot` 命令需要手动回车（下同），回车后路由器会重启

**解锁 SSH 成功！**

## 四、 恢复 SSH
> 若已解锁并固化过 SSH 的路由器在升级固件或恢复出厂设置后导致 SSH 丢失，可快速再次解锁 SSH
{: .prompt-tip }
### 1. 计算 Telnet 登录密码
打开网站 <https://miwifi.dev/ssh>，在 SN 处输入路由器背面的 SN 号，点击“Calc”后再点击“Copy”即可复制密码
- 注：复制的密码即 Telnet 和 SSH 登录密码

<img src="/assets/img/pin/caculate-ssh-password.png" alt="计算 Telnet 登录密码" width="60%" />

### 2. 永久开启并固化 SSH
连接 Telnet，执行如下命令：
- 注：最后一行命令是将 Telnet 或 SSH 登录密码设置为 `12345678`，可自定义

```shell
mkdir -p /data/auto_ssh
curl -o /data/auto_ssh/auto_ssh.sh -L https://cdn.jsdelivr.net/gh/lemoeo/AX6S@main/auto_ssh.sh
chmod +x /data/auto_ssh/auto_ssh.sh
/data/auto_ssh/auto_ssh.sh install
echo -e '12345678\n12345678' | passwd root
```

### 3. 更改 Telnet 和 SSH 登录密码（可选）
执行命令 `passwd root`，输入密码如：`12341234`，回车后输入同样的密码，再次回车即可

**恢复 SSH 成功！**

## 五、 ShellCrash 安装和配置
### 1. ShellCrash 安装
- ① 打开 WinSCP，将下载的 ShellCrash.tar.gz 文件移动到路由器的 `/tmp`{: .filepath} 目录中  
  <img src="/assets/img/pin/move-shellcrash.png" alt="ShellCrash 安装 1" width="60%" />

- ② 连接 SSH 后执行如下命令：

  ```shell
  mkdir -p /tmp/SC_tmp && tar -zxf '/tmp/ShellCrash.tar.gz' -C /tmp/SC_tmp/ && source /tmp/SC_tmp/init.sh
  ```

- ③ 选择 1 安装到 /data 目录（推荐，支持软固化功能）
- ④ 根据需要自定义别名（此处选择 `2`）
- ⑤ 将下载的 sing-box-ref1nd-dev-linux-armv8.tar.gz 文件复制到桌面，以管理员身份运行 PowerShell，依次执行如下命令：

  ```
  cd C:\Users\[用户名]\Desktop
  tar -zxvf sing-box-ref1nd-dev-linux-armv8.tar.gz
  ```

  .tar.gz 压缩文件成功解压到桌面上，目录结构为 `C:\Users\[用户名]\Desktop\CrashCore`{: .filepath}
- ⑥ 将 CrashCore 文件移动到路由器的 `/tmp`{: .filepath} 目录中  
  <img src="/assets/img/pin/move-sing-boxr.png" alt="ShellCrash 安装 2" width="60%" />

**安装 ShellCrash 成功！**

### 2. ShellCrash 配置
- ① 连接 SSH 后执行命令 `sc` 即可打开 ShellCrash 配置脚本
- ② 新手引导
  - ➊ 选择 1 路由设备配置局域网透明代理
  - ➋ 根据需要是否开启小内存模式（此处选择 `0`）
  - ➌ 启用推荐的自动任务配置
  - ➍ 根据需要是否启用软固化（此处选择 `1`，解锁 SSH 时已成功启用软固化）
  - ➎ 根据需要是否选择 1 确认导入配置文件（此处选择 `0`）
  - ➏ 根据需要是否选择 1 立即启动服务（此处选择 `0`）
    - 注：强烈建议选择 `0`，待以下配置完成后，最后一步启动服务
  - ➐ 此时脚本会自动“发现可用的内核文件”，选择 `1` 加载，后选择 5 Sing-Box-reF1nd 内核  
    <img src="/assets/img/pin/import-sing-boxr.png" alt="ShellCrash 配置 1" width="60%" />

  - ➑ 内核加载完成后根据需要是否保留相关数据库文件（此处选择 `0`）
- ③ 功能设置
  - ➊ 进入 ShellCrash 配置脚本 → 2 功能设置 → 1 路由模式设置（推荐“混合模式”，其次“Tproxy 模式”，宽带在 300M 内推荐“Tun 模式”）
    - 注：使用“Tun 模式”前须进入主菜单 → 8 工具与优化，开启 8 小米设备 Tun 模块修复

  - ➋ 进入 2 功能设置 → 2 DNS 设置（推荐“MIX 模式”）
  - ➌ 进入 2 DNS 设置 → 7 DNS 劫持端口，设置为 `5353`（AdGuard Home 的“DNS 服务器端口”须设置为 `5353`）
  - ➍ 进入 2 DNS 设置 → 9 修改 DNS 服务器，选择 4 一键配置加密 DNS
    - 注：推荐设置 DNS 分流（单独使用 ShellCrash 以及 ShellCrash 搭配 AdGuard Home 都适用），请看《[搭载 sing-boxr 内核进行 DNS 分流教程-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/dnsbypass-singboxr-ruleset)》

  - ➎ 进入 2 功能设置，选择 5 启用域名嗅探
  - ➏ 进入 2 功能设置 → 8 ipv6 设置，若机场节点不支持 IPv6，可关闭 1 ipv6 透明路由  
    <img src="/assets/img/pin/ipv6-setting.png" alt="ShellCrash 配置 2" width="60%" />

- ④ 进入主菜单 → 4 启动设置，选择 1 允许 ShellCrash 开机启动（若重启路由器后服务没有自动运行，可“设置自启延时”为 `30` 秒，然后在《[六、 1. ⑥](https://proxy-tutorials.dustinwin.us.kg//posts/pin-shellcrashadguardhome-mihomo/#1-adguard-home-%E5%AE%89%E8%A3%85)》，将 `sleep 10s` 改为 `sleep 40s`）
- ⑤ 进入主菜单 → 5 设置自动任务 → 1 添加自动任务，输入对应的数字并回车后可设置执行条件
- ⑥ 进入主菜单 → 9 更新与支持 → 7 切换安装源及安装版本，选择 b 切换至公测版-master → 1 Jsdelivr_CDN 源，追求新版可选择 c 切换至开发版（可能不稳定）  
  <img src="/assets/img/pin/select-update-source.png" alt="ShellCrash 配置 5" width="60%" />

- ⑦ 进入主菜单 → 9 更新与支持 → 4 安装本地 Dashboard 面板，选择 1 安装 zashboard 面板  
  注：
    - ➊ 启动服务后，面板 Dashboard 访问链接为：<http://192.168.31.1:9999/ui/>
    - ➋ 初次打开需要添加“主机”和“端口”，分别填入 `192.168.31.1` 和 `9999` 并点击“添加”即可访问 Dashboard 面板

  <img src="/assets/img/pin/install-dashboard.png" alt="ShellCrash 配置 6" width="60%" />

- ⑧ 进入主菜单 → 6 管理配置文件  
  注：
    - ➊ 选择 1 在线生成配置文件，粘贴你的订阅链接并回车，输入 `1` 并再次回车即可
    - ➋ 选择 2 在线获取配置文件，需要一定的 sing-boxr 知识储备，请查看《[生成带有自定义出站和规则的 sing-boxr 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.us.kg/posts/link-singboxr-ruleset/)》
    - ➌ 若 1 或 2 无法正确获取配置文件，可进入 9 自定义浏览器 UA 选择合适的 UA

  导入配置文件完成后，选择 1 启动/重启服务

**配置 ShellCrash 成功！**

**ShellCrash 常用命令：**
1. 打开配置：`sc`
2. 启动服务：`$CRASHDIR/start.sh start`
3. 停止服务：`$CRASHDIR/start.sh stop`
4. 重启服务：`$CRASHDIR/start.sh restart`
5. 更新订阅：`$CRASHDIR/task/task.sh update_config`
6. 查看帮助和说明：`sc -h`

### 3. ShellCrash 升级
进入主菜单 → 9 更新与支持，查看“管理脚本”、“内核文件”和“数据库文件”有无新版本，有则选择对应的数字进行升级即可  
<img src="/assets/img/pin/update-shellcrash-singboxr.png" alt="ShellCrash 升级" width="60%" />

### 4. ShellCrash 卸载
- ① 通过脚本命令进行卸载（任选一）  
  连接 SSH 后执行命令 `$CRASHDIR/start.sh stop && sc -u`
- ② 通过 ShellCrash 配置进行卸载（任选一）  
  进入主菜单 → 9 更新与支持，选择 9 卸载 ShellCrash

## 六 、 AdGuard Home 安装和配置
### 1. AdGuard Home 安装
- ① 将下载的 upx-[version]-win64.zip 文件解压到桌面，目录结构为 `C:\Users\[用户名]\Desktop\upx`{: .filepath}
- ② 将下载的 AdGuardHome_linux_arm64.tar.gz 文件复制到桌面，以管理员身份运行 PowerShell，依次执行如下命令：

  ```shell
  cd C:\Users\[用户名]\Desktop
  tar -zxvf AdGuardHome_linux_arm64.tar.gz
  ```

  .tar.gz 压缩文件成功解压到桌面的 `AdGuardHome`{: .filepath} 文件夹内，目录结构为 `C:\Users\[用户名]\Desktop\AdGuardHome`{: .filepath}
- ③ 进入 `AdGuardHome`{: .filepath} 文件夹，将里面的“AdGuardHome”文件移动到 `C:\Users\[用户名]\Desktop\upx`{: .filepath} 文件夹中  
  依次执行如下命令：

  ```shell
  cd C:\Users\[用户名]\Desktop\upx
  .\upx AdGuardHome
  ```

- ④ 将压缩后的“AdGuardHome”文件移动到路由器的 `/data/AdGuardHome`{: .filepath} 目录（没有此目录就新建）中  
  <img src="/assets/img/pin/move-adguardhome.png" alt="AdGuard Home 安装 1" width="60%" />

- ⑤ 进入路由器文件管理的 `/data/auto_ssh`{: .filepath} 目录，右击“auto_ssh.sh”文件并点击“编辑”
  - 注：若没有此目录和文件，可新建，且须连接 SSH 后执行命令 `chmod +x /data/auto_ssh/auto_ssh.sh`

  <img src="/assets/img/pin/edit-task.png" alt="AdGuard Home 安装 2" width="60%" />

> AdGuard Home 的“DNS 服务器端口”须设置为 `5353`
{: .prompt-warning }

- ⑥ 在最下方添加如下内容并保存：  
  - 注： 若 ShellCrash 设置了自启延时如 `30` 秒，须将 `sleep 10s` 修改为 `sleep 40s`（即 +10s）

  ```shell
  sleep 10s
  /data/AdGuardHome/AdGuardHome -s install
  /data/AdGuardHome/AdGuardHome -s start
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ```

- ⑦ 连接 SSH 后直接粘贴如下所有命令：

  ```shell
  chmod +x /data/AdGuardHome/AdGuardHome
  /data/AdGuardHome/AdGuardHome -s install
  /data/AdGuardHome/AdGuardHome -s start
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ```

**安装 AdGuard Home 成功！**

### 2. AdGuard Home 配置
- ① 引导设置
  - ➊ 打开网页 <http://192.168.31.1:3000> 后点击“开始配置”
  - ➋ **“网页管理界面端口”输入 `3000`，“DNS 服务器端口”输入 `5353`**，点击“下一步”  
  - ➌ “身份认证”设置用户名和密码
  - ➍ 点击“打开仪表盘”后输入刚才设置的用户名和密码“登入”，就可以进入 AdGuard Home 管理页面
- ② 进入设置 → 常规设置，取消勾选“启用日志”并点击“保存”（日志非常占用空间）
- ③ DNS 设置
  - ➊ 进入设置 → DNS 设置，“上游 DNS 服务器”设置为 `localhost:1053`，并选择“并行请求”
    - 注：此时页面右下角可能会弹出报错信息，但不用理会

    <img src="/assets/img/pin/adguardhome-up-dns.png" alt="AdGuard Home 配置 1" width="60%" />

  - ➋ “后备 DNS 服务器”设置为：

    ```text
    quic://dns.alidns.com:853
    https://doh.pub/dns-query
    ```

  - ➌ “Bootstrap DNS 服务器”设置为：

    ```text
    223.5.5.5
    119.29.29.29
    ```

  - ➍ 直接点击“应用”即可  
    <img src="/assets/img/pin/adguardhome-dns.png" alt="AdGuard Home 配置 2" width="60%" />

  - ➎ “速度限制”输入 `0`，然后点击下方的“保存”  
    <img src="/assets/img/pin/adguardhome-dns-service.png" alt="AdGuard Home 配置 3" width="60%" />

  - ➏ 勾选“乐观缓存”，并点击“保存”  
    <img src="/assets/img/pin/adguardhome-cache.png" alt="AdGuard Home 配置 4" width="60%" />

- ④ 进入过滤器 → DNS 黑名单 → 添加黑名单 → 从列表中选择，推荐勾选“区域”里的“CHN: anti-AD”，然后点击“保存”
  - 注：若等待 10 分钟仍下载失败，可手动将下载地址 URL 更改为 `https://anti-ad.net/easylist.txt`

  <img src="/assets/img/pin/adguardhome-blacklist.png" alt="AdGuard Home 配置 5" width="60%" />

  添加成功  
  <img src="/assets/img/pin/adguardhome-blacklist-success.png" alt="AdGuard Home 配置 6" width="60%" />

- ⑤ 进入过滤器 → DNS 重写 → 添加 DNS 重写，“输入域”填写 `miwifi.com`，“输入 IP 地址或域名”填写 `192.168.31.1`，然后点击“保存”  
  注：
    - ➊ 此步骤可解决访问 <http://miwifi.com> 时无法打开小米或红米路由器管理页面的问题，其它型号路由器请根据自身需要填写
    - ➋ 若已在 ShellCrash 配置文件自行添加了此域名相关 `hosts`，仍可执行此步骤，防止其停止服务后无法正常访问

  添加成功  
  <img src="/assets/img/pin/adguardhome-dns-rewrite.png" alt="AdGuard Home 配置 7" width="60%" />

**配置 AdGuard Home 成功！**

**AdGuard Home 常用命令：**
1. 启动服务：`/data/AdGuardHome/AdGuardHome -s start`
2. 停止服务：`/data/AdGuardHome/AdGuardHome -s stop`
3. 重启服务：`/data/AdGuardHome/AdGuardHome -s restart`
4. 显示当前服务状态：`/data/AdGuardHome/AdGuardHome -s status`

### 3. AdGuard Home 升级
为了节约路由器内存，请按照如下步骤进行操作：
- ① 执行《[六、 1. ① ② ③ ④（替换）](https://proxy-tutorials.dustinwin.us.kg//posts/pin-shellcrashadguardhome-singboxr/#1-adguard-home-%E5%AE%89%E8%A3%85)》的操作步骤
- ② 连接 SSH 后执行命令 `/data/AdGuardHome/AdGuardHome -s restart`

### 4. AdGuard Home 卸载
- ① 删除开机启动项
  执行《[六、 1. ⑥](https://proxy-tutorials.dustinwin.us.kg//posts/pin-shellcrashadguardhome-singboxr/#1-adguard-home-%E5%AE%89%E8%A3%85)》的操作步骤，删除添加的内容：

  ```shell
  sleep 10s
  /data/AdGuardHome/AdGuardHome -s install
  /data/AdGuardHome/AdGuardHome -s start
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 5353
  ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 5353
  ```

  并保存
- ② 卸载 AdGuard Home
  连接 SSH 后直接粘贴如下所有命令：

  ```shell
  /data/AdGuardHome/AdGuardHome -s stop && /data/AdGuardHome/AdGuardHome -s uninstall && rm -rf /data/AdGuardHome
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53
  iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
  ip6tables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53
  ip6tables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
  ```

- ③ 重启路由器

## 七、 效果图
### 1. IPv6 效果
<img src="/assets/img/pin/show-ipv6-1.png" alt="IPv6 效果 1" /><img src="/assets/img/pin/show-ipv6-2.png" alt="IPv6 效果 2" />

### 2. BT 下载效果
UDP 连接正常，使用的是移动 1000M 带宽  
<img src="/assets/img/pin/show-bt.png" alt="BT 下载效果" />

### 3. ShellCrash 效果
使用的是移动 1000M 带宽  
<img src="/assets/img/pin/show-speedtest.png" alt="ShellCrash 效果" />

### 4. AdGuard Home 效果
<img src="/assets/img/pin/show-adguardhome.png" alt="AdGuard Home 效果" />
