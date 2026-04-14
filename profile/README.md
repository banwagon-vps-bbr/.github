
很多人第一次买搬瓦工，是因为看到有人说"这东西速度快，买来自己折腾"。

然后就买了。然后就发现——速度好像也就那样。晚高峰一到，加载慢得像是在用拨号上网。

其实问题不一定出在服务器本身，也不一定是机房选错了。很多时候，差的就是一个配置：**BBR 没有开**。

---

## 从一次晚高峰的崩溃，聊到 BBR 是什么

那年我在搬瓦工上跑了一个小工具，白天用起来顺滑得很，到了晚上八九点，延迟从 150ms 飙到 400ms，有时候直接断连。

起初以为是机房问题，折腾了一圈换机房，换来换去，最后一个朋友告诉我：你有没有开 BBR？

我说：BBR 是啥？

他说：谷歌出的 TCP 拥塞控制算法，专门用来在有丢包的网络环境下把带宽利用率拉满，降低延迟。

我说：听起来很厉害。

他说：开了试试，晚高峰那段时间效果特别明显。

我开了。效果确实明显。

---

## BBR 到底做了什么

简单说，传统 TCP 在遇到丢包时会把发送速率直接砍半，非常保守，结果就是——网络稍微抖一抖，速度就掉下来了。

BBR 换了一种思路：它不把丢包当成拥塞信号，而是通过测量带宽和延迟来判断真实网络状态，动态调整发送速率。在有一定丢包率的跨国线路上，这个区别特别明显。

搬瓦工的 VPS 用的是 KVM 架构，天然支持更换内核，BBR 可以正常跑起来。OpenVZ 架构就不行了，但搬瓦工早就全面切换到 KVM 了，所以新买的机器不用担心这个。

---

## 搬瓦工上怎么开 BBR

其实搬瓦工这几年已经把 BBR 的开启门槛降得很低了，有两种方法：

**方法一：重装系统，选带 `-bbr` 后缀的版本**

登录 KiwiVM 控制面板，找到 `Install new OS`，在系统列表里找带 `bbr` 后缀的版本（比如 `centos-7-x86_64-bbr`），装完就自动开启了。这是最省事的方式。

如果你用的是最新版 Ubuntu 或 Debian，内核本身就已经内置 BBR，安装完成后直接用下面的命令确认是否启用：

bash
sysctl net.ipv4.tcp_congestion_control


输出 `net.ipv4.tcp_congestion_control = bbr` 就说明成功了。

**方法二：手动安装 BBR 一键脚本**

SSH 登录后执行：

bash
wget --no-check-certificate -O /opt/bbr.sh https://github.com/teddysun/across/raw/master/bbr.sh
chmod 755 /opt/bbr.sh
/opt/bbr.sh


安装完成后按提示重启，重启后输入以下命令验证：

bash
lsmod | grep bbr


看到 `tcp_bbr` 出现在结果里，就是开启成功了。

**方法三：追求更极致？试试 BBRplus 或 BBR 魔改版**

如果觉得标准 BBR 还不够，还可以安装多合一脚本：

bash
wget --no-check-certificate -O tcpx.sh https://raw.githubusercontent.com/ylx2016/Linux-NetSpeed/master/tcpx.sh
chmod +x tcpx.sh
./tcpx.sh


脚本里可以选 BBR、BBR2、BBRplus、锐速等多种方式，挨个试一试，找到适合你那个机房、那个线路、那个运营商的最优解。

---

## BBR 开了，然后呢？机房和线路才是天花板

BBR 能把你手里的网络质量压榨到极致，但如果底层线路本身就是普通的 163 骨干，那榨出来的极致也是有上限的。

搬瓦工的套餐按线路分，差距很大：

**入门线路（CN2 GT / KVM 基础套餐）**：价格便宜，$49.99/年起。9 个机房可以随意切换，适合学 Linux、跑不敏感业务。晚高峰有时候会堵，但开了 BBR 通常能改善不少。

**主力推荐（CN2 GIA-E 套餐）**：这是搬瓦工性价比的核心产品。电信走 CN2 GIA，联通走 9929，移动走 CMIN2，三网都有高端直连线路。12 个机房可以一键切换，包括 DC6、DC9、日本大阪软银、荷兰联通等。配合 BBR，晚高峰延迟稳在 150-200ms 附近，丢包率极低。👉 [点击查看 CN2 GIA-E 套餐](https://bwh81.net/aff.php?aff=80238&pid=87)

**高端线路（香港 CN2 GIA 套餐）**：延迟直接降到 30-60ms，三网回程全程 CN2 GIA。价格也直接上天，月付 $89.99 起。适合对延迟有极致要求，或者公费报销的朋友。

2025 年搬瓦工还把 DC9 机房的宿主机从老款 Intel E5 迁移到了 AMD EPYC，CPU 单核性能提升了 2-3 倍，硬盘换成了 NVMe RAID-10 阵列。这意味着即使是入门套餐，底层硬件性能也今非昔比了。

---

## 搬瓦工全套餐对比表

以下是搬瓦工目前主要在售套餐的完整对比，价格均为原价，可搭配优惠码 **BWHCGLUKKB**（6.77% 循环折扣）使用：

| 套餐系列 | 内存 | CPU | 硬盘 | 带宽 | 月流量 | 价格 | 购买 |
|---------|------|-----|------|------|--------|------|------|
| KVM / CN2 GT（基础款 A） | 1 GB | 2 核 | 20 GB SSD | 1 Gbps | 1 TB | $49.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=44) |
| KVM / CN2 GT（基础款 B） | 2 GB | 3 核 | 40 GB SSD | 1 Gbps | 2 TB | $52.99/半年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=45) |
| KVM / CN2 GT（基础款 C） | 4 GB | 4 核 | 80 GB SSD | 1 Gbps | 3 TB | $19.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=46) |
| KVM / CN2 GT（基础款 D） | 8 GB | 5 核 | 160 GB SSD | 1 Gbps | 4 TB | $39.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=47) |
| KVM / CN2 GT（基础款 E） | 16 GB | 6 核 | 320 GB SSD | 1 Gbps | 5 TB | $79.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=48) |
| CN2 GIA-E（推荐款 A） | 1 GB | 2 核 | 20 GB SSD | 2.5 Gbps | 1 TB | $49.99/季·$169.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=87) |
| CN2 GIA-E（推荐款 B） | 2 GB | 3 核 | 40 GB SSD | 2.5 Gbps | 2 TB | $89.99/季·$299.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=88) |
| CN2 GIA-E（推荐款 C） | 4 GB | 4 核 | 80 GB SSD | 2.5 Gbps | 3 TB | $39.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=89) |
| CN2 GIA-E（推荐款 D） | 8 GB | 5 核 | 160 GB SSD | 2.5 Gbps | 5 TB | $69.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=90) |
| 香港 CN2 GIA（高端 A） | 2 GB | 2 核 | 40 GB SSD | 1 Gbps | 500 GB | $89.99/月·$899.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=95) |
| 香港 CN2 GIA（高端 B） | 4 GB | 4 核 | 80 GB SSD | 1 Gbps | 1 TB | $155.99/月·$1559.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=96) |
| 电商 SLA（企业级 A） | 1 GB | 2 核 AMD | 20 GB NVMe | 2.5 Gbps | 1 TB | $65.89/季 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=164) |
| 电商 SLA（企业级 B） | 2 GB | 3 核 AMD | 40 GB NVMe | 2.5 Gbps | 2 TB | $116.99/季 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=165) |
| 电商 SLA（企业级 C） | 3 GB | 4 核 AMD | 80 GB NVMe | 2.5 Gbps | 3 TB | $69.99/月 |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=166) |
| MINICHICKEN 限量版 | 1 GB | 1 核 | 20 GB SSD | 1 Gbps | 2 TB | $19.99/年（限量） |  [立即购买](https://bwh81.net/aff.php?aff=80238&pid=158) |

> 限量版套餐随时可能缺货，看到有货建议直接下手。CN2 GIA-E 套餐是最值得长期持有的常规套餐，没有之一。

---

## 搬瓦工 VPS + BBR：不同场景下的搭配建议

**新手、预算有限、主要学习用**：KVM 基础套餐 $49.99/年，装完系统第一件事就是开 BBR，然后安心折腾。

**建站、需要稳定高速线路**：CN2 GIA-E 套餐，选 DC6 或 DC9 机房，电信用 CN2 GIA，移动切 CMIN2，联通走 9929。BBR 是标配，开完基本不需要再折腾。👉 [直接选 CN2 GIA-E 起步套餐](https://bwh81.net/aff.php?aff=80238&pid=87)

**追求极致低延迟、预算够**：香港 CN2 GIA 套餐，延迟 30-60ms，开不开 BBR 区别都不大，因为线路本身就已经足够好了。

**企业/电商场景，需要 SLA 保障**：洛杉矶 DC5 SLA 套餐，99.99% 在线率保证，AMD CPU + NVMe 硬盘，三网高端直连，每两周还可以免费换一次 IP。

---

## 最后聊几句

搬瓦工这个平台，用了这么多年，该有的槽点都有：偶尔缺货、部分机房限量、主域名有时候被污染需要用镜像站。但在 CN2 GIA 这条线路的稳定性和易用性上，它确实是国内用户买海外 VPS 时绕不开的选项。

BBR 是免费的，开启成本几乎为零，但它能把你现有机器的网络潜力多挖出一块来，尤其是在跨国线路上。如果你现在用的搬瓦工还没开，不妨现在就去试一下。

一个小建议：开了 BBR 之后，也别忘了看看机房选对了没有。电信用 CN2 GIA，联通推荐 9929，移动走 CMIN2——线路选对，BBR 才能发挥出真正的效果。两件事加在一起，比单独做其中一件，体感差距是很明显的。

👉 [去看看搬瓦工最新套餐和价格](https://bwh81.net/aff.php?aff=80238)
