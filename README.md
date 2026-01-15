# 关于
1. [本教程合集](https://proxy-tutorials.dustinwin.us.kg)基于 [mihomo 内核](https://github.com/MetaCubeX/mihomo)和 [sing-box reF1nd 版内核](https://github.com/reF1nd/sing-box)编写，请尽量使用最新版内核，可到 [DustinWin/proxy-tools](https://github.com/DustinWin/proxy-tools) 项目进行更新
2. 本教程合集分为五大类，分别为：[置顶](https://proxy-tutorials.dustinwin.us.kg/categories/%E7%BD%AE%E9%A1%B6/)、[直链配置](https://proxy-tutorials.dustinwin.us.kg/categories/%E7%9B%B4%E9%93%BE%E9%85%8D%E7%BD%AE/)、[工具配置](https://proxy-tutorials.dustinwin.us.kg/categories/%E5%B7%A5%E5%85%B7%E9%85%8D%E7%BD%AE/)、[DNS 配置](https://proxy-tutorials.dustinwin.us.kg/categories/dns-%E9%85%8D%E7%BD%AE/)和[分享配置](https://proxy-tutorials.dustinwin.us.kg/categories/%E5%88%86%E4%BA%AB%E9%85%8D%E7%BD%AE/)。伸手党可直接进入分享配置进行参考，请尽量使用我定制的[规则集文件](https://github.com/DustinWin/ruleset_geodata)
3. 本教程合集偏向于实操，基础配置的搭载内核和 Wiki 文档对应关系如下表：

|   搭载内核    |         mihomo 内核          |         sing-box reF1nd 版内核         |           sing-box 内核            |
| :-----------: | :--------------------------: | :------------------------------------: | :--------------------------------: |
| **Wiki 文档** | <https://wiki.metacubex.one> | <https://sing-boxr.dustinwin.us.kg/zh> | <https://sing-box.sagernet.org/zh> |

# 对 geodata 和 ruleset 方案的说明
1. geodata 方案更适用于路由器等设备（连接多台设备而无法判断非本机设备进程），配置文件编写简单，对小白用户友好
2. ruleset 方案适用于对分流规则要求比较严格的用户，配置文件编写复杂，可按需配置且配置灵活
3. mihomo 采用 geodata 方案，使用 `GEOSITE` 和 `GEOIP` 规则搭配 GeoSite.dat 和 GeoIP.dat（或 Country.mmdb）[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
4. mihomo 采用 ruleset 方案，使用 `RULE-SET` 规则搭配 .list 和 .mrs [规则集文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%BA%8C-ruleset-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
5. sing-box 采用 ruleset 方案，使用 `rule_set` 规则搭配 .json 和 .srs [规则集文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%BA%8C-ruleset-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

# 对下载源的说明
1. 本教程默认下载源为 [jsDelivr 源](https://www.jsdelivr.com/github)，格式为 `https://cdn.jsdelivr.net/gh/[username]/[reponame]@[branchname]/[filename]`，此源中**文件更新有 12 小时延迟**
2. 若 jsDelivr 源无法访问，可将地址改为 `https://fastly.jsdelivr.net/gh/[username]/[reponame]@[branchname]/[filename]`
3. 推荐使用 [GitHub 源](https://github.com)，格式分别为 `https://github.com/[username]/[reponame]/releases/download/[tagname]/[filename]`（推荐）、`https://raw.githubusercontent.com/[username]/[reponame]/[tagname]/[filename]` 和 `https://raw.githubusercontent.com/[username]/[reponame]/[branchname]/[filename]`
4. 若 GitHub 源无法访问，可添加 `https://ghfast.top/` 前缀，即：将地址分别改为 `https://ghfast.top/https://github.com/[username]/[reponame]/releases/download/[tagname]/[filename]`（推荐）、`https://ghfast.top/https://raw.githubusercontent.com/[username]/[reponame]/[tagname]/[filename]` 和 `https://ghfast.top/https://raw.githubusercontent.com/[username]/[reponame]/[branchname]/[filename]`

# 给作者加鸡腿
<img src="/assets/img/pin/alipay.jpg" alt="支付宝" width="30%" />

<img src="/assets/img/pin/wechat.png" alt="微信" width="30%" />

# 机场推荐
[Bitz Net](https://dir3.bnaffloop.com/#/register?code=HT0ALWZq)（仅次于一线机场，推荐打折时购买）  
85 折优惠码：`XMAS2025`（有效期至 2026 年 1 月 2 日 23:59）
