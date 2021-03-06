## 简介
VPN 连接是一种通过公网加密通道连接您的对端 IDC 和私有网络的方式。如下图所示，腾讯云私有网络 VPN 连接分为以下几个组成部分：
- VPN网关：创建的私有网络 IPsec VPN 网关
- 对端网关： IDC 端的 IPsec VPN 服务网关
- VPN通道：加密的 IPsec VPN 通道
![](//mccdn.qcloud.com/static/img/a654d376b4e4e13ae2bb65b13239cef2/image.png)

私有网络内可以建立VPN网关，每个VPN网关可以建立多个VPN通道，每个VPN通道可以打通一个本地IDC。需要注意的是，**在建立VPN连接之后，您需要在路由表中配置相关路由策略，才能真正实现通信。**
 
## VPN网关
VPN 网关是私有网络建立 VPN 连接的出口网关，与对端网关（IDC 侧的 IPsec VPN 服务网关）配合使用，主要用于腾讯云私有网络和外部 IDC 之间建立安全可靠的加密网络通信。腾讯云 VPN 网关通过软件虚拟化实现，采用双机热备策略，单台故障时自动切换，不影响业务正常运行。

VPN网关根据带宽上限分为5种设置，分别为：5M、10M、20M、50M、100M。您可以随时调整 VPN 网关带宽设置，即时生效。

## 对端网关
对端网关是指 IDC 机房的 IPsec VPN 服务网关，对端网关需与腾讯云 VPN 网关配合使用，一个VPN网关可与多个对端网关建立带有加密的VPN网络通道。

## VPN通道
VPN网关和对端网关建立后，即可建立VPN通道，用于私有网络和外部 IDC 之间的加密通信。当前 VPN 通道支持 IPsec 加密协议，可满足绝大多数 VPN 连接的需求。

VPN 通道在运营商公网中运行，公网的网络阻塞、抖动会对VPN网络质量有影响，因此也无法提供 SLA 服务协议保障。如果业务对延时、抖动敏感，建议通过专线接入私有网络，更多内容可以查看[专线接入服务](https://cloud.tencent.com/product/dc.html)。

腾讯云上的 VPN 通道在实现 IPsec 中使用 IKE（Internet Key Exchange，因特网密钥交换）协议来建立会话。IKE 具有一套自保护机制，可以在不安全的网络上安全地认证身份、分发密钥、建立 IPSec 会话。

VPN通道的建立包括以下配置信息：
- 基本信息
- SPD（Security Policy Database）策略
- IKE配置（选填）
- IPsec配置（选填）

下面详细介绍基本信息、SPD策略、IKE 配置（选填）和 IPsec 配置（选填）。

### 基本信息
协议类型：IKE/IPsec
预共享秘钥：预共享密钥是用于验证 L2TP/IPSec 连接的 Unicode 字符串，本端和对端必须使用相同的预共享密钥。

### SPD（Security Policy Database）策略
SPD（Security Policy Database）策略由一系列 SPD 规则组成，每条规则用于指定 VPC 内哪些网段可以和 IDC 中哪些网段通信。
- 每条SPD策略对应一个本端网段和多个对端网段，本段网段和对端网段不能重叠。
- 所有策略的集合中本端网段之间不可重叠
- 每个本端网段的多条对端网段不可重叠
- 对端网段不可与私有网络网段重叠

下面是一个正确的实例：

SPD策略1 本端网段 `10.0.0.0/24`，对端网段为 `192.168.0.0/24`、`192.168.1.0/24`。
SPD策略2 本端网段 `10.0.1.0/24`，对端网段为 `192.168.2.0/24`。
SPD策略3 本端网段 `10.0.2.0/24`，对端网段为 `192.168.2.0/24`。

![](//mccdn.qcloud.com/static/img/5b32174d312e31c5b5a9162a50456de8/image.png)
 
### IKE配置

|配置项|说明|
|--|--|
|版本|IKE V1|
|身份认证方法|默认预共享秘钥|
|认证算法|身份认证算法，支持MD5和SHA1|
|协商模式|支持main（主模式）和aggressive（挑战者模式）<br><br>二者的不同之处在于，aggressive 模式可以用更少的包发送更多信息，这样做的优点是快速建立连接，而代价是以清晰的方式发送安全网关的身份，使用 aggressive 模式时，配置参数如 Diffie-Hellman 和 PFS 不能进行协商，因此两端拥有兼容的配置是至关重要的|
|本端标识|支持 IP address 和 FQDN（全称域名）|
|对端标识|支持 IP address 和 FQDN|
|DH group|指定 IKE 交换密钥时使用的 DH 组，密钥交换的安全性随着 DH 组的扩大而增加，但交换的时间也增加了<br><br>Group1：采用 768-bit 模指数（Modular Exponential，MODP ）算法的 DH 组<br><br> Group2：采用 1024-bit MODP 算法的 DH 组<br><br> Group5：采用 1536-bit MODP 算法的 DH 组<br><br>Group14：采用 2048-bit MODP 算法，不支持动态 VPN 实现此选项<br><br> Group24：带 256 位的素数阶子群的 2048-bit MODP算法 DH 组，不支持组 VPN 实现此选项|
|IKE SA Lifetime|单位：秒<br><br>设置 IKE 安全提议的 SA 生存周期，在设定的生存周期超时前，会提前协商另一个 SA 来替换旧的 SA。在新的 SA 还没有协商完之前，依然使用旧的 SA；在新的 SA 建立后，将立即使用新的 SA，而旧的 SA 在生存周期超时后，被自动清除|

###  Ipsec 信息
|配置项|说明|
|--|--|
|加密算法|支持 3DES、AES-128、AES-192、AES-256、DES|
|认证算法|支持 MD5 和 SHA1|
|报文封装模式|Tunnel|
|安全协议|ESP|
|PFS|支持 disable、dh-group1、dh-group2、dh-group5、dh-group14和dh-group24|
|IPsec SA lifetime(s)|单位：秒|
|IPsec SA lifetime(KB)|单位：KB|

## 使用约束
### VPN连接约束
关于 VPN 连接，您需要注意的是:
- VPN 参数配置完成后，**您需要在子网关联路由表中添加指向 VPN 网关的路由策略**，子网内云主机访问对端网段的网络请求才会通过 VPN 通道传递至对端网关。
- 在配置完路由表之后，**您需要在 VPC 内云主机 ping 对端网段中的 IP 以激活此 VPN 通道。**
- VPN 连接稳定性依赖运营商公网质量，暂时无法提供 SLA 服务协议保障。

| 资源 | 限制（个） | 
|---------|---------|
| 每个私有网络内 VPN 网关个数 | 10 | 
| 同一地域内对端网关个数| 20 | 
| 同一个对端网关支持的VPN通道数 | 10 | 
| 同一VPN网关可创建的VPN通道数 | 20	 | 
| 每个 VPN 通道的 SPD 个数 | 10 | 
| 每个 SPD 支持的对端网段数 | 50 | 

### 对端网关IP地址约束
对端网关不支持以下IP地址：
- 全0，全255，224开头的组播地址;
- 回环地址: 127.x.x.x/8;
- IP地址中主机位为全0或者全1的地址，如:
a.以A类中1~126开头举例，1~126.0.0.0以及1~126.255.255.255;
b.以B类中128~191开头举例，128~191.x.0.0以及128~191.x.255.255;
c. 以C类中192~223开头举例，192~223.x.x.0以及192~223.x.x.255;
- 内部服务地址：169.254.x.x/16;

## 计费模式
 VPN通道和对端网关**免费**。
 VPN网关支持两种计费模式，按量付费和按月付费。
1) **按量后付费**，包含网关租用费（按小时计费）和访问公网产生的流量费用，[查看公网流量费用](https://cloud.tencent.com/document/product/213/509#.E6.8C.89.E6.B5.81.E9.87.8F.E8.AE.A1.E8.B4.B912)

| 地域 | 国内 |硅谷、法兰克福、香港、韩国|  多伦多、新加坡 |
|---------|---------|---------|---------|
| 价格 | 0.48元/h | 0.58元/h |0.75元/h |


2) **按月预付费**，单价已包含 IDC 带宽的费用，云主机无需重复购买网络带宽。具体如下表所示：

| 配置（Mbps） |除北美（多伦多）外|北美（多伦多）|
|---------|---------|
|5 |380 |480|
|10 |880 |1330|
|20 |1880 |2330|
|50 |4880 |	5330|
|100 |9880 |10330|

关于私有网络服务的更多价格，可以参考[私有网络价格总览](https://cloud.tencent.com/doc/product/215/3079)。

## 操作指南

### 快速入门
IPsec VPN 可以在控制台实现全自助配置，您需要完成以下几步才能实现使 VPN 连接生效：
1)	创建VPN网关
2)	创建对端网关
3)	创建VPN通道
4)	在自有 IPsec VPN 网关中加载配置文件
5)	**设置路由表**
6)	**VPN通道激活**

示例：
通过IPsec VPN连接打通您在**广州**的私有网络TomVPC中子网A`192.168.1.0/24`与您的IDC中子网`10.0.1.0/24`，而您IDC中VPN 网关的公网 IP 是`202.108.22.5`。
![](//mc.qcloudimg.com/static/img/0cfc46cf11e4d53164219b1c386509a1/1.png)

您需要完成以下几个步骤：
#### 第一步：创建VPN网关
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【VPN连接】-【VPN网关】选项卡。
3) 在列表的上端选择私有网络myVPC所在**广州**和私有网络`TomVPC`，点击【新建】。
4) 填写VPN网关名称（如：TomVPNGw）选择合适的带宽配置并付款后，VPN网关创建完成之后，系统随机了分配公网IP，如：`203.195.147.82`。

#### 第二步：创建对端网关
在VPN通道创建前，需要创建对端网关：
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【VPN连接】-【对端网关】选项卡。
3)	在列表的上端选择地域：**广州**，点击【新建】。
4)	填写对端网关名称（如：TomVPNUserGw）和 IDC 的 VPN 网关的公网 IP `202.108.22.5 `。
5)	点击【创建】，即可在对端网关列表查看到新建的对端网关。

####  第三步：创建VPN通道
创建VPN 通道分为以下几个步骤：

1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)  点击左导航栏中【VPN连接】-【VPN通道】选项卡。
3)  在列表的上端选择私有网络myVPC所在**广州**和私有网络`TomVPC`，点击【新建】。
4)  输入通道名称（如：TomVPNConn），选择VPN网关`TomVPNGw`与对端网关`TomVPNUserGw`，并输入预共享密钥（如：`123456`）。
5)  输入 SPD 策略来限制本段哪些网段和对端哪些网段通信，在本例中本端网段即为子网A的网段`192.168.1.0/24`，对端网段为`10.0.1.0/24`，点击【下一步】。
6) （可选）第三步配置 IKE 参数（可选），如果您不需要高级配置可直接点击【下一步】。
7) （可选）第四步配置 IPsec 参数（可选），如果您不需要配置，可直接点击【完成配置】。
8)  点击完成VPN通道，下载配置文件。

####  第四步：在自有 IPsec VPN 网关中加载配置文件
将第三步生成的配置文件在您 IDC 的 IPsec VPN 网关中加载配置，才可实现 VPN 通道的网络互通。

####  第五步：修改路由表
截止至第第四步，我们已经将一条VPN通道配置成功，但是由于您还未将子网A中的流量路由至VPN网关上，子网A中的网段还不能与IDC中的网段通信。现在配置路由：
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【子网】，在列表的上端选择私有网络myVPC所在**广州**和私有网络`TomVPC`，点击子网A所关联的路由表 ID 进入该路由表的详情页。
3)	点击【编辑按钮】，点击【新增一行】，输入目的端网段（`10.0.1.0/24`），下一跳类型选择【VPN网关】，再选择刚刚创建的 VPN 网关 `TomVPNGw`。
4)	点击【保存】，即完成需要通信的子网的出包路由设定。

####  第六步：VPN 通道激活
用VPC内的云服务器 ping 对端网段中的 IP 以激活 VPN 通道。如：`TomVPC`内的子网A中的云服务器`ping 10.0.1.1`

### 查看监控数据
VPN 通道和 VPN 网关提供监控数据查看功能。
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【VPN连接】-【VPN网关】或者【VPN通道】选项卡。
3)  点击列表页中监控一列的图标查看监控数据。

### 设置告警
VPN 通道提供告警功能：
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击顶部导航条【云产品】-【监控与管理】-[【云监控】](https://console.cloud.tencent.com/monitor/overview)，选择左导航栏内的【我的告警】-[【告警策略】](https://console.cloud.tencent.com/monitor/policylist)，点击：新增告警策略。
2)	填写告警策略名称，在策略类型中选择【VPN通道】，然后添加告警触发条件。
3)	**关联告警对象**：选择告警接收组，保存后即可在告警策略列表中查看已设置的告警策略。
4)	**查看告警信息**：告警条件被触发后，您将接受到短信/邮件/站内信等通知，同时可以在左导航【我的告警】-【告警列表】中查看。有关告警的更多信息，请参考[创建告警](https://cloud.tencent.com/doc/product/248/1073)。

### 查看 VPN 网关详细信息
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【VPN连接】-【VPN网关】选项卡。
3)  点击 VPN 网关 ID 即可进入 VPN 网关详情页查看 VPN 网关信息。

### 修改VPN通道配置
1)	登录[腾讯云控制台](https://console.cloud.tencent.com/)点击导航条【私有网络】，进入[私有网络控制台](https://console.cloud.tencent.com/vpc/vpc?rid=8)。
2)	点击左导航栏中【VPN连接】-【VPN通道】选项卡。
3)  点击 VPN 网关 ID 即可进入 VPN 网关详情页查看 VPN 网关信息。
4)  您可以在基本信息页中修改基本信息和SPD策略，或者您可以在高级配置修改IKE和Ipsec配置。
 

## API概览
您可以使用API操作来设置和管理您的VPN连接，私有网络的更多相关API可以参考[私有网络所有 API 概览。](https://cloud.tencent.com/doc/api/245/909)
### VPN 相关接口
| 接口功能 | Action ID |  功能描述 |
|---------|---------|---------|
| 查询VPN网关价格 | [InquiryVpnPrice](http://cloud.tencent.com/doc/api/245/5104) | 查询VPN网关价格。 |
| 购买VPN网关 | [CreateVpn](http://cloud.tencent.com/doc/api/245/5106) | 购买VPN网关。 |
| 修改VPN网关属性 | [ModifyVpnGw](http://cloud.tencent.com/doc/api/245/5107) | 修改指定VPN网关信息，例如名称。|
| 查询VPN网关列表 | [DescribeVpnGw](http://cloud.tencent.com/doc/api/245/5108) | 根据用户信息，如VPN网关ID，名称，查询对应VPN网关的信息。|
| 续费VPN网关 | [RenewVpn](http://cloud.tencent.com/doc/api/245/5109) | 续费VPN网关。 |

### 对端网关相关接口
| 接口功能 | Action ID |  功能描述 |
|---------|---------|---------|
| 创建对端网关 | [AddUserGw](http://cloud.tencent.com/doc/api/245/5116) | 创建要连接的对端网关。 |
| 删除对端网关 | [DeleteUserGw](http://cloud.tencent.com/doc/api/245/5117) | 删除指定对端网关。 |
| 修改对端网关名称 | [ModifyUserGw](http://cloud.tencent.com/doc/api/245/5118) | 修改对端网关名称。 |
| 查询对端网关列表 | [DescribeUserGw](http://cloud.tencent.com/doc/api/245/5119) | 根据用户信息，如对端网关ID，名称，查询对应对端网关的信息。|
| 获取可支持的对端网关厂商信息 | [DescribeUserGwVendor](http://cloud.tencent.com/doc/api/245/5120) | 查询腾讯云vpn网关可支持的对端网关厂商信息。 |


### VPN通道相关接口

| 接口功能 | Action ID |  功能描述 |
|---------|---------|---------|
| 创建VPN通道 | [AddVpnConn](http://cloud.tencent.com/doc/api/245/5110) | 创建VPN加密通道，将VPC接入其他网络资源。 |
| 删除VPN通道 | [DeleteVpnConn](http://cloud.tencent.com/doc/api/245/5111) | 删除指定VPN通道。|
| 修改VPN通道 | [ModifyVpnConn](http://cloud.tencent.com/doc/api/245/5112) | 修改指定VPN通道的信息，如名称。 |
| 查询VPN通道列表 | [DescribeVpnConn](http://cloud.tencent.com/doc/api/245/5113) | 根据用户信息，如通道ID，名称，查询对应通道的信息。|
| 下载VPN通道配置 | [GetVpnConnConfig](http://cloud.tencent.com/doc/api/245/5114) | 下载VPN通道配置，对通道配置做调整。 |
| 获取VPN通道的监控数据 | [DescribeVpnConnMonitor](http://cloud.tencent.com/doc/api/245/5115) |  获取VPN通道的监控数据。 |


