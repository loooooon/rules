资料来源：
[CustomOpenClashRules](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)
[七尺宇](https://qichiyu.blogspot.com/2024/07/openclash.html)
# 1.准备工作
## 1.OpenClash 仓库

- 仓库地址：https://github.com/vernesong/OpenClash/releases
- 依赖安装
```
#iptables
opkg update
opkg install coreutils-nohup bash iptables dnsmasq-full curl ca-certificates ipset ip-full iptables-mod-tproxy iptables-mod-extra libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip luci-compat luci luci-base
```

```
#nftables
opkg update
opkg install coreutils-nohup bash dnsmasq-full curl ca-certificates ip-full libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
```

## 2.网络-LAN接口(旁路由模式)
![image.png](https://s2.loli.net/2024/12/10/B8hUjITX7c2QmsN.png)

- 协议：静态地址
- iPv4：本机访问地址（设置为主路由的同局域网段内）
![image.png](https://s2.loli.net/2024/12/10/CLDWu3PtwUMG4rA.png)
- 高级设置：自定义DNS服务器（主路由iP，运营商iP）
- DHCP服务器：常规设置-忽略此接口，高级设置-关闭动态DHCP，iPv6-取消勾选指定的主接口

## 3.网络-WAN接口(主路由模式)
![image.png](https://s2.loli.net/2024/12/10/K1Mvdg5qnf9Rzrx.png)
- 然后在OpenWrt的首页查看是否取得了运营商下发的 DNS，如果你打算使用其他的国内第三方 DNS，可以跳过此步骤。
![image.png](https://s2.loli.net/2024/12/10/nhKpIcELS3oiWmB.png)
# 2.OpenClash常规设置
## 1.关闭Dnsmasq自带的 DNS 重定向功能
- 网络 > DHCP/DNS 页面中,若不关闭，有可能会引起 DNS 解析问题，并会导致本方案的广告拦截设置无法拦截国外域名
![image.png](https://s2.loli.net/2024/12/10/UyBHPeNiLxDkF9X.png)

## 2.确保OpenWrt可以正常访问Github
![image.png](https://s2.loli.net/2024/12/10/ZuF3dwlfmtHxqnc.png)

## 3.模式设置
- 插件设置：启动Meta内核
- 运行模式：Fake-iP（增强）模式
- UDP流量转发（前提订阅的机场节点支持此服务）
- 代理模式：Rule【策略模式】
- 旁路网关（旁路由）兼容：勾选启动（前提是旁路由工作模式）

## 4.流量控制
![image.png](https://s2.loli.net/2024/12/10/fv7KIzCrnaES6Aq.png)

- 路由本机代理：启用
- 禁用 QUIC：启用
- 绕过服务器地址：启用
- 实验性绕过指定区域 IP：绕过中国大陆（提升性能）
- WAN 接口名称：主路由选WAN口，没有WAN口的旁路由选LAN口（不可留空）
- 绕过指定区域 IPv4 黑名单(如果没有此字段则添加)
```
  baidu.com
  #114.114.114.114
  ##解决绕过大陆后谷歌商店无法更新
  services.googleapis.cn
  googleapis.cn
  xn--ngstr-lra8j.com
```

## 5.DNS设置
- 本地 DNS 劫持：使用Dnsmasq 转发（顺手点下Fake-IP 持久化缓存清理）
- 禁止Dnsmasq缓存 DNS：启用
## 6.流媒体增强(可选)
此处设置主要用于使 OpenClash 可在流媒体分流时在众多节点中自动选择解锁对应区域的流媒体服务的节点，此功能主要用于在一些流媒体解锁不稳定且混乱的杂牌机场中自动寻找对应的节点。  
**如果你所使用的机场的流媒体解锁服务相对比较稳定，或者已经知晓你所使用的机场哪些节点可以解锁你所需要的区域的流媒体服务，则可以跳过此页面的设置，设置完成后在 Clash 的控制面板中自行选择即可。** 如果你要使用自动选择节点的功能，首先勾选你要使用的流媒体服务，比如 Netflix，然后按照本方案的策略组名称在“策略组筛选”中进行填写，本方案中流媒体相关的策略组包括 Netflix、YouTube、Disney+等等  
解锁区域填写你要解锁的流媒体服务区域，比如你要解锁新加坡区就填写SG。解锁节点筛选填写需要测试的节点名称的关键词，比如填写“香港|新加坡”就会在包含以上关键词的节点中进行筛选  
例如设置了 Netflix 和 SG，如此设置后，OpenClash 启动后就会在你订阅的节点的清单中自动寻找解锁新加坡（SG）区域的 Netflix 服务的节点作为分流策略组“Netflix”的指定节点  
设置后记得点击页面下方的“保存配置”，再次提醒此页是可选功能，非必要不使用
![image.png](https://s2.loli.net/2024/12/10/lO5i2Sdsa4Tuh9U.png)
## 7.IPv6 设置
建议关闭，如要设置和流量控制同理

## 8.GEO 数据库订阅

OpenClash一些兜底的分流数据库，保持最新没有坏处。按照图中设置即可，具体用途不多做解释，可以自行查找相关资料。  
注意，每次数据库更新成功后 OpenClash会自动重启，建议设置更新时间为不用网的时候，比如凌晨。  
设置完后点击页面下方的“保存设置”，然后顺手把三个“检查并更新”按钮都点一遍。在 OpenClash的“运行日志”页面可以查看更新结果，此操作可以顺带验证你的 OpenWrt是否能顺利访问Github 或者你在之前设置的 CDN 比如testingcf
![image.png](https://s2.loli.net/2024/12/10/VuAn16BWkUKrEtM.png)

## 9.大陆白名单
![image.png](https://s2.loli.net/2024/12/10/ocrl91J4dnPxtWb.png)

## 10.版本更新
此页面用于更新 OpenClash 的内核以及 OpenClash 自身  
**务必**选择 dev 版本，然后点击下方的一键更新。在 OpenClash 的“运行日志”页面可以查看更新结果，此操作可以顺带验证你的 OpenWrt 是否能顺利访问 Github 或者你在之前设置的 CDN 比如 testingcf  
如果使用过程中遇到错误，可以选择 master 版本，再点击一键更新即可切换回稳定版，目前来看一些新特性必须 dev 版本的 OpenClash 和 Meta 才能支持，所以非特殊情况强烈建议使用 dev 版本  
**如果你使用 dev 版本，下图中的界面将只显示 Meta 更新选项，这是正常的，因为另外两个内核早已停止更新（删库跑路）**
![image.png](https://s2.loli.net/2024/12/10/mIM1w23yQXtsNnz.png)

# 3.OpenClash覆写设置
## 1.DNS设置
OpenWrt 是主路由的情况下，此处勾选按照图中进行勾选。使用运营商通告的 DNS，勾选“追加上游 DNS”。  
旁路由以及坚持使用其他 DNS 的用户，则不要勾选“追加上游 DNS”。  
如果你有 DDNS 服务的域名，填写进下方的 Filter 中。
![image.png](https://s2.loli.net/2024/12/10/1Wuy6Oib9aEcgvY.png)

**页面下方有 NameServer、Fallback 和 Default-NameServer 三个服务器分组，在本方案中应当取消全部的服务器勾选。** **如果你坚持不愿意使用运营商提供的 DNS**，则不要勾选“追加上游 DNS”，并且在 NameServer 中保留第一个勾选，填入你要使用的国内第三方 DNS 的地址，例如 223.5.5.5。  
**如果你使用 DNS 优化插件（如 SmartDNS 等）**，则不要勾选“追加上游 DNS”，并且在 NameServer 中保留第一个勾选，填入 127.0.0.1:[Smart DNS 端口号]。  
**如果你的 OpenWrt 是旁路由**， 则不要勾选“追加上游 DNS”，并且在 NameServer 中保留第一个勾选，填入运营商 DNS 或者你要使用的其他国内第三方 DNS。  
注意在本方案中，NameServer 只用做 OpenClash 的规则判断以及所有的直连域名解析，且本方案中 OpenClash 配置了绕过大陆功能，所以此处填入多个服务器并没有任何意义，更不要自作聪明的填写国外 DNS 服务器。  
**再一次强烈建议此处全部取消勾选，配合上面一页设置的“追加上游 DNS”来使用运营商 DNS 从而提高解析速度！（仅限 OpenWrt 是主路由的情况下）**  
为什么要取消 Fallback 服务器？在取消 Fallback 服务器勾选的情况下，OpenClash 会把域名发送到远端的机场服务器上进行解析，只有这样才会根据不同区域的节点取得理论上的最佳解析结果  
设置完成后，点击页面下方的“保存配置”按钮

![image.png](https://s2.loli.net/2024/12/10/JcYfQe8XFkvUZmT.png)
## 2.Meta 设置
![image.png](https://s2.loli.net/2024/12/10/hd8yYrDfQbCIgBc.png)

## 3.规则设置
按照图中勾选，该功能有一定的副作用，具体见 [vernesong/OpenClash#3942](https://github.com/vernesong/OpenClash/issues/3942)  
个人建议：有 BT、PT、P2P 下载的较强需求的开启，没有的则关闭  
如果开启后仍然有相关流量走了代理，可以尝试将漏网之鱼策略组指定为直连，但是漏网之鱼策略组指定为直连会导致 DNS 泄露问题（仅仅是不能通过测试网站的测试，并非真正泄露），建议自行取舍。  
开启后，日志中可能出现少量 MATCH 报错，无需担心，不影响日常使用。
![image.png](https://s2.loli.net/2024/12/10/RPIw2FpcW67H8Gn.png)
## 4.开发者选项
此项设置来自恩山论坛相关教程贴  
按照图中内容，找到 Hash Demo 下的对应代码，取消注释并修改 true 为 false  
`ruby_edit "$CONFIG_FILE" "['experimental']" "{'sniff-tls-sni'=>false}"`  
新版本中似乎没有这一行选项，忽略本条即可。
![image.png](https://s2.loli.net/2024/12/10/VTxSr8u9whlU7aW.png)


# 4.配置订阅
## 1.subconverter后端 Docker安装：
安装命令：

docker run -d --restart=always -p 25500:25500 tindy2013/subconverter:latest

检查命令：（SSH连接openwrt执行命令）

curl http://localhost:25500/version

如果出现 subconverter vx.x.x backend 则说明容器已经成功运行。

如果容器无法启动  重启docker ！

[http://localhost:25500/sub](http://localhost:25500/sub)   openwrt本机docker地址

[https://sub.qichiyu.com/sub](https://sub.qichiyu.com/sub)  VPS 域名反代 地址

[http://192.168.10.5:25500/sub](http://192.168.10.5:25500/sub) 局域网其他设备地址

## 2.订阅模板
模板地址：[https://raw.githubusercontent.com/Aethersailor/Custom_OpenClash_Rules/main/cfg/Custom_Clash.ini](https://raw.githubusercontent.com/Aethersailor/Custom_OpenClash_Rules/main/cfg/Custom_Clash.ini)
（暂时用着，后续根据自己需求更改）
![image.png](https://s2.loli.net/2024/12/10/tpUonwGYrIeD8Rl.png)


## 3.更新配置并启动

点击配置订阅页面中的“更新配置”按钮，OpenClash 即可开始更新配置并启动
![image.png](https://s2.loli.net/2024/12/10/ZUmAhalYX839xpG.png)

更新后切换到运行日志页面观察 OpenClash 的启动情况  
出现“OpenClash 启动成功，请等待服务器上线！”后，即表示 OpenClash 已经启动成功
![image.png](https://s2.loli.net/2024/12/10/GuAMU7wWcLPB1Tq.png)

## 检查 DNS 是否存在泄漏

访问 ([IPLEAK.NET](https://ipleak.net/))检查 DNS 是否存在泄漏  
正常情况下，页面上方应当出现你的机场节点的 IPv4 和 IPv6 地址，页面下方无中国大陆 DNS 出现即为 DNS 无泄漏情况
![image.png](https://s2.loli.net/2024/12/10/ZWQ9b5tqaRiNfxo.png)


# 5.个人踩的坑
## 1.无法访问国内网站

检查下常规设置--流量设置--wan口设置是不是没选或者是选错了