# 开源世界线变动率探测仪复刻

Built with love.

个人交流群：https://discord.gg/5eup2fzRB，如果链接挂了可以私聊我 lamadaemon (Lama3L9R#4650)

有些时候dc看不到我人捏可以来我的tg频道找我：t.me/alpbroadcastd

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu)
[![LICENSE](https://img.shields.io/badge/license-Anti%20996-blue.svg)](https://github.com/996icu/996.ICU/blob/master/LICENSE)

---

## 简介

开源的世界线变动率探测仪复刻，参考了 https://nixieclock.org/ 大佬的设计，整个项目采用5张电路板完成，总成本大约不到 8000 CNY 。

部分资料暂时并不会放出来，因为还在整理中，请给我一些时间。这其中包括剩余的三张电路板的原理图和生产文件、BOM表、控制板程序的固件、电源板程序固件。固件采用 rust 编写，然而我的 rust 技术并不是很好，所以可能需要更多时间才会放出所有内容（现在不敢放出来，太丢人了）。由于我人已经出国留学，在国外继续更新、验证、修改等非常不便，所以控制板 1.2 和电源板 2.2 版本的改动未经过测试。由于改动仅为一些很小的bugfix，并不会很影响核心功能。核心功能在我飞机的前一天验证通过。固件我还是会持续更新测试。

**Update: 至于为什么都三个月了还是没放出来，主要是因为我懒了，固件新功能还没做，代码也没整理，以及我账号下的 divergence-meter-firmware 是老版本设计（第二版设计）的固件不能使用**

**Update: 至于为什么都半年了还是没有放出来，主要是因为我懒了，固件新功能还没做，代码也没整理。**

**Update: [固件](https://github.com/Lama3L9R/oss-divergence-meter-fw-archive) | 为什么 Archive？是不维护了吗？非也！我也不知道为什么我选择用 esp-idf-hal，这玩意儿虽然有 std 支持，但是文档是真的少，研究了这么久，我也没搞定 32.768 和 wifi，所以我打算用 C 或者 rust 重写，如果我用 rust，那么我会考虑用 esp-hal 而不是 esp-idf-hal**

> **仅仅为了制作此项目，0基础入门电子，两年，终于抵达了镜界面上的命运石之门。**

## 结构

项目五张电路板分别分为：电源板、控制板、顶层控制板、顶层外观板（无线路）、外侧控件板（预留空间未作）。

### 电源板

电源板将产生一路 170V 高压可编程供电、一路 3.3V、一路 5V。3.3V 和 5V将提供给控制板作为供电，高压将作为辉光管驱动电源。由于电源板有可能需要上电时序要求，即需要先调节数字电位器后再启动高压供电，否则芯片会嗯升压直到击穿20多CNY的 mos 管。支持快充和电池充电。支持独立运行（贴板载的神他妈32），也支持直接由控制板控制。

### 控制板

控制板上一颗ESP32-S3控制四个IO扩展器，用于控制辉光管数字。一路3.3V LDO，作为 ESP32 的模拟供电电源。每个辉光管电流检测。射频因为没有时间，也没有钱去做阻抗匹配，因此性能大概差的一批，根据我的测试，一丁点信号都没，暂不清楚是因为金属外壳的原因还是其他的原因。预留了专用的 6pin debug 接口，如有需要转接板可以看我的 LamaFlash 项目（如果我能想起来去点一下公开工程）。

### 顶层

顶层控制板上仅为一堆 mos 管，因为最顶上的装饰用的板子需要打孔，所以走不了线，所以才需要这样将两张板子堆叠在一起。使用 FFC 排线和控制板链接，一条2Pin XH2.45 用于拉高压供电。

### 外侧控制板

外侧控制板，本来是用来放电源开关啊啥的，但是我在国内的时候各种摸鱼最后没时间了，就摆烂了，有电就一直开机的那种（
反正是预留了，只要做出来这块小板插上排线改改程序就能用了。

**请注意：控制板射频线路阻抗匹配按照 JDBPCB 4层标准层压结构设计，做了 50 Ohms 阻抗匹配，如果你想在 JLC 打板，请注意修改线宽**

## 已测试内容

大部分核心功能已经通过测试，由于我人润了，并不能再继续测试修复了的bug是否真的修复了。没有测试的内容：高压供电开启、温度传感器。

已知问题：电源板上的神他妈32我debugger找不到芯片，只有我贴上去的第一颗是好的，后来不小心烧了然后就再也没成功刷上去过程序。这个我已经摆烂了，怀疑是买到假芯片了。（测试芯片的时候甚至还带走了我的 JLink）

## 原件选型

在这里并不会太详细的介绍选型，只是想分享一下我在选型和确定方案的时候用的一些小技巧。

我很喜欢 TI 的芯片，TI芯片优点是质量高、文档全、工具全，缺点就是价格，部分比较新的芯片难买，容易买到假的（比如我电源板上的降压芯片我之前就买到假的了）。

电源板上的除了LDO的高压、3V 5V 全都是使用 TI 的 Webench Power designer (https://webench.ti.com/power-designer/) 一键生成的。

高压在最开始的设计本是要使用 经典 TTRN 变压器反激升压方案，但后来发现变压器功率不够，跑到 5W 就冒烟了，还会出现奇怪的问题，因此就换了效率低到爆的 Boost 方案。But it works !

快充协议芯片和充电管理芯片方案其实如果只是从半岛小芯或者立创商城直接找的话会很难找到合适的，这种时候，如果你不差钱，去搞几张新一点的笔记本的图纸，笔记本的快充和电池管理正好是高电压的，很符合目前我的需求。目前我用的充电芯片和快充协议芯片就是参考了 Surface Pro X的选型。选择笔记本方案还有一个优点是芯片好买，因为维修总会需要配件的，而且很多时候如果缺钱还有二手芯片可以选择。苏菲上用的是 TPS65988，查询同系列的 65987DDJ 可以买到（25CNY/each）而且适合我的需求（虽说还是浪费了一堆pin）。充电芯片则是一模一样的，电路方面还可以有一个参考，剩余的直接看 datasheet 即可。

## 仓库文件内容

目前只上传了电源板和控制板的原理图和 Gerber 文件，源文件采用 AD 设计，未来整理之后会放出源文件。

PD.bin 和 PDConfig.pjt 是 TPS65987DDJ 的配置文件，需要将 .bin 文件 用 SPI 编程器写到 Flash 里，.pjt 是源文件，可以用官方的 GUI 配置工具打开，我的配置文件为最高请求 20V。

case-step.7z 包含了 step 文件和 drill 信息，这也是我提交给打样商家的内容，在这里推荐一下商家（并不是广告）：科浩机械加工，老板人很好。

剩余的电路板资料、程序源代码、程序二进制等等还需要一些时间，程序因为我懒+摆烂，所以电源板独立运行（那颗神他妈32）的程序我懒得写了，但是ESP32的我是有在持续开发的，我需要稍微的清理一下代码，然后就上传到另一个仓库，大概可能叫 divergence-meter-firmware

详细制作过程等等再等几天，有空了我会上传博客的。

## OSHWHub 和照片

最后来看几张照片，还有一些关于 OSHWHub 的事

![View1](https://github.com/Lama3L9R/divergence-meter/blob/master/views/DivergenceMeterView.jpg?raw=true "View1")  
![View2](https://github.com/Lama3L9R/divergence-meter/blob/master/views/InsideViewjpg.jpg?raw=true "View2")  
![View3](https://github.com/Lama3L9R/divergence-meter/blob/master/views/PowerConsumption.jpg?raw=true "View3")  


![OSHW1](https://github.com/Lama3L9R/divergence-meter/blob/master/OSHWHub/OSHWHUB.png?raw=true "OSHW1")  
![OSHW2](https://github.com/Lama3L9R/divergence-meter/blob/master/OSHWHub/OSHWHUB2.png?raw=true "OSHW2")  
![OSHW3](https://github.com/Lama3L9R/divergence-meter/blob/master/OSHWHub/OSHWHUB3.png?raw=true "OSHW3")  
![OSHW4](https://github.com/Lama3L9R/divergence-meter/blob/master/OSHWHub/OSHWHUB4.png?raw=true "OSHW4")  

