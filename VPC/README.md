# VPC

本文档主要介绍VPC主要概念和相关服务以及AWS SAA考试中的相关习题。您可以先参照[官方文档](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/what-is-amazon-vpc.html)进行学习。

## 公有子网和私有子网

AWS中没有指定公有子网和私有子网的属性，要理解这个概念需要理解`路由表`，和`Internet网关`。

### 1. 路由表

一个网络中的网络请求被发起后，需要通过路由器来分发（路由）向目标。VPC也有这么一个路由器,这个路由器负责路由VPC内的所有网络请求，包括VPC内部实例（机器）之间的网络请求，以及进出Internet的网络请求。VPC通过查找路由表中的配置，从匹配到的配置项中确定目标，然后将网络请求路由给这个目标。每个子网都需要关联一张路由表，也就是绑定一张路由表。如下表所示：

| Destination    | Target         |
|----------------|----------------|
| 192.168.0.0/16 | local          |
| 0.0.0.0/0      | igw-84yegr7643 |

每个路由表含有两个属性：

- Destination： 值是一个CIDR块，用来表示匹配目的IP地址的范围。
- Target： 值是一个AWS服务的id，表示匹配成功的数据流的目的地。

假设有一条发往`192.168.1.2`的请求，VPC路由器接收到这条请求后进行匹配，根据上表，这条请求处于`192.168.0.0/16`范围内，于是将这条请求路由到`local`（VPC本地网络）中的`192.168.1.2`主机。假设另外一条发往`202.8.1.3`的请求，VPC开始匹配，发现该请求属于`0.0.0.0/0`范围内，于是将这条请求路由到`igw-84yegr7643`（Internet网关）。这里VPC路由器遵循**最长前缀匹配的原则**匹配原则，上面的例子，虽然`192.168.1.2`与`192.168.0.0/16`和`0.0.0.0/0`都能匹配成功，但是取CIDR范围最小的进行匹配。

### 2. Internet网关

VPC内的数据流出到Internet以及Internet上的数据流入到VPC都要经过Internet网关。所以公有子网和私有子网的唯一区别就是**关联的路由表中是否配置有一条目标为Internet网关的路由规则**。私有子网关联的路由表中没有目标为Internet网关的路由规则，这意味着私有子网无法跟互联网连接。

### 3. 私有子网与Internet连接

- 私有子网与Internet的连接（例如私有子网中的软件升级）：在公有子网中配置NAT网关，并且在私有子网所关联的路由表中加入NAT网关，路由器将请求路由到NAT，紧接着NAT将请求转发到Internet网关，实现私有子网与Internet的连接。

- Internet与私有子网的连接（例如在家中使用SSH远程登录公司私有子网中的实例）：在公有子网中设置一条堡垒机（Bastion），通过堡垒机转发从Internet过来的请求。

## 共享VPC

VPC共享允许多个AWS账户在共享的集中式管理的VPC。

### 1. 先决条件

通过[AWS Organizations](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html#getting-started-sharing-orgs)启用共享。

### 2. 权限

1. 拥有者权限
   - 负责创建、管理和删除所有 VPC 级别的资源。
   - 无法修改或删除参与者资源。

2. 参与者权限
   - 负责创建、管理和删除其资源。
   - 无法查看或修改属于其他参与者账户的资源。

## CIDR

[CIDR计算器](http://www.subnet-calculator.com/cidr.php)

## VPC中的数据保护

1. AWS建议：
   - 对每个账户使用 Multi-Factor Authentication (MFA)。
   - 使用 SSL/TLS 与 AWS 资源进行通信。
   - 使用 AWS CloudTrail 设置 API 和用户活动日志记录。
   - 使用高级托管安全服务（例如 Amazon Macie），它有助于发现和保护存储在 Amazon S3 中的个人数据。
 
 2. 提高和监控VPC的安全性的四种服务：
    - 安全组：安全组用作相关 Amazon EC2 实例的防火墙，可在实例级别控制入站和出站的数据流。
    - 网络访问控制列表 (ACL)：用作关联的子网的防火墙，在子网级别同时控制入站和出站流量。
    - 流日志：流日志捕获有关在您的 VPC 中传入和传出网络接口的 IP 流量的信息。您可以为 VPC、子网或各个网络接口创建流日志。流日志数据将发布到 CloudWatch Logs 或 Amazon S3，这可帮助您诊断过于严格或过于宽松的安全组和网络 ACL 规则。
    - 流量镜像：您可以从 Amazon EC2 实例的弹性网络接口复制网络流量。然后将流量发送到带外安全和监控设备。

### 安全组

- 安全组充当实例的虚拟防火墙以控制入站和出站流量。当您在 VPC 中启动实例时，您可以为该实例最多分配5个安全组。安全组在实例级别运行，而不是子网级别。因此，在您的VPC的**子网中的每个实例都归属于不同的安全组集合**。

- 基本特征
   - 您可以指定允许规则，但不可指定拒绝规则。
   - 您可以为入站和出站流量指定单独规则。
   - 当您创建一个安全组时，它没有入站规则。
   - 与安全组关联的实例无法彼此通信，除非您添加了允许流量的规则。
   - 安全组与网络接口关联。
   
- 关于安全组的源：所谓源（Source），就是源头，允许这个源头通过。当源头是一个CIDR块时，表示允许这个CIDR块代表IP地址的范围的数据流通过，当安全组的源是另外一个安全组的id时，表示允许该id下的所有实例访问。
   
### ACL

- 网络访问控制列表 (ACL) 是 VPC 的一个可选安全层，可用作防火墙来控制进出一个或多个子网的流量。

## VPC Endpoint

VPC终端节点使您能够将VPC私密地连接到支持的AWS服务和VPC终端节点服务（由AWS PrivateLink提供支持），而无需Internet网关、NAT设备、VPN连接或AWS Direct Connect连接。VPC中的实例无需公有IP地址便可与服务中的资源通信。VPC和其他服务之间的通信不会离开Amazon网络。

一句话概括：VPC Endpoint可以使处在私有子网的EC2不经过Internet连接Amazon中的其他服务。

### 1. 接口VPC终端节点（Interface VPC Endpoint）

利用接口VPC终端节点（接口终端节点），您可连接到由AWS PrivateLink提供支持的服务。

注意事项：
   - 对于每个接口终端节点，每个可用区您只能选择一个子网。
   - 可能无法在所有可用区中通过接口终端节点使用服务。
   - 如果子网的网络 ACL 限制流量，您可能无法通过终端节点网络接口发送流量。
   - 确保与终端网络接口关联的安全组允许终端网络接口与 VPC 中与此服务通信的资源之间进行通信。
   - 接口终端节点仅支持 TCP 流量。
   - 仅在同一地区内支持终端节点。无法在 VPC 和其他区域内的服务之间创建终端节点。
   - 终端节点仅支持 IPv4 流量。
   - 无法将终端节点从一个 VPC 转移到另一个 VPC，也无法将终端节点从一项服务转移到另一项服务。
   
### 2. 网关VPC终端节点（Gateway VPC Endpoint）

适用于S3和DynamoDB的VPC终端节点。

### 3. VPC终端节点服务

在VPC中创建自己的应用程序并将其配置为AWS PrivateLink支持的服务。比如我自己做了一个叫`Big Daddy S3`的存储服务并希望其他用户能通过VPC终端节点连接，在这里我是服务提供商。因此`VPC终端节点服务`是面向服务提供商的服务。

## 模拟题

### 问题

1. 一家公司需要记录私有子网中所有IP包（源，目标，协议），最佳解决方案是什么？（#1-8-C01）
   - [ ] A. 使用`VPC flow logs`。
   - [ ] B. 使用EC2上的`source destination checkout`。
   - [ ] C. 使用`AWS CloudTrail`。
   - [ ] D. 使用`Amazon CloudWatch`。
   <details>
   <summary>查看答案</summary>
   
   - [x] A. 使用`VPC flow logs`。
   - [ ] B. 使用EC2上的`source destination checkout`。
   - [ ] C. 使用`AWS CloudTrail`。
   - [ ] D. 使用`Amazon CloudWatch`。
   
   </details>
   
2. 一个应用运行在私有子网的EC2实例上，这个应用需要读写`Amazon Kinesis Data Streams`上的数据。但是该公司要求读取数据的流不能流向万维网（Internet），最佳解决方案是什么？（#1-39-C01）
   - [ ] A. 在一个共有子网中配置一个`NAT网关（NAT Gateway）`并且将读写流路由到`Kinesis`服务上。
   - [ ] B. 为`Kinesis`配置一个`网关 VPC 终端节点（Gateway VPC Endpoint）`并且通过其将读写流路由到`Kinesis`服务上。
   - [ ] C. 为`Kinesis`配置一个`接口 VPC 终端节点（Interface VPC Endpoint）`并且通过其将读写流路由到`Kinesis`服务上。
   - [ ] D. 为`Kinesis`配置一个`AWS Direct Connect 虚拟接口`并且通过其将读写流路由到`Kinesis`服务上。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 在一个共有子网中配置一个`NAT网关（NAT Gateway）`并且将读写流路由到`Kinesis`服务上。
   - [ ] B. 为`Kinesis`配置一个`网关 VPC 终端节点（Gateway VPC Endpoint）`并且通过其将读写流路由到`Kinesis`服务上。
   - [x] C. 为`Kinesis`配置一个`接口 VPC 终端节点（Interface VPC Endpoint）`并且通过其将读写流路由到`Kinesis`服务上。
   - [ ] D. 为`Kinesis`配置一个`AWS Direct Connect 虚拟接口`并且通过其将读写流路由到`Kinesis`服务上。
   
   </details>

3. 您启动了一个实例并且将它用作在公共子网中NAT设备。接着你修改路由表，在私有子网中将此NAT设备变更成互联网流的目标，当你试图使用此私有子网中的实例去连接外网（outbount connection）,你发现这并不成功，什么操作能解决这个问题？（#1-47-C01）
   - [ ] A. 为这个NAT实例添加`弹性网络接口（Elasitc Network Interface）`，并把它加入到私有子网中。
   - [ ] B. 为这个NAT实例添加`弹性IP地址（Elasitc IP Address）]`。
   - [ ] C. 为这个NAT实例添加另外一个`弹性网络接口（Elasitc Network Interface）`并把它加入到公有子网中。
   - [ ] D. 停止这个NAT实例上的`Source/Destination Check`功能。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 为这个NAT实例添加`弹性网络接口（Elasitc Network Interface）`，并把它加入到私有子网中。
   - [ ] B. 为这个NAT实例添加`弹性IP地址（Elasitc IP Address）]`。
   - [ ] C. 为这个NAT实例添加另外一个`弹性网络接口（Elasitc Network Interface）`并把它加入到公有子网中。
   - [x] D. 停止这个NAT实例上的`Source/Destination Check`功能。
   
   </details>

4. 作为架构师，您正在使用AWS进行数据库服务器托管，除了必要的升级文件需要下载以外，这台服务器不允许链接因特网，以下哪项服务适合该项需求？（#1-7-C02）
   - [ ] A. 将数据库服务器配置在公有子网中，使用ACL将网络设置为只允许入站流量通过。
   - [ ] B. 将数据库服务器配置在公有子网中，使用安全组将网络设置为只允许入站流量通过。
   - [ ] C. 将数据库服务器配置在私有子网中，使用安全组将网络设置为只允许出站流量通过。
   - [ ] D. 将数据库服务器配置在私有子网中，并在路由表中配置NAT实例。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 将数据库服务器配置在公有子网中，使用ACL将网络设置为只允许入站流量通过。
   - [ ] B. 将数据库服务器配置在公有子网中，使用安全组将网络设置为只允许入站流量通过。
   - [ ] C. 将数据库服务器配置在私有子网中，使用安全组将网络设置为只允许出站流量通过。
   - [x] D. 将数据库服务器配置在私有子网中，并在路由表中配置NAT实例。
   
   </details>

5. 您所在的公司将要将本地文件服务器的数据迁移到AWS中。本地文件服务器的数据量很大，为了节省工期，数据迁移时需要维持50Mbps的高速专用线路，下列哪种方法最适合？（#1-13-C02）
   - [ ] A. VPC Peering。
   - [ ] B. VPN。
   - [ ] C. AWS Direct Connect。
   - [ ] D. NAT
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. VPC Peering。
   - [ ] B. VPN。
   - [x] C. AWS Direct Connect。
   - [ ] D. NAT
   
   </details>

6. B公司的基干业务均在AWS中，最近您所在的A公司收购了B公司，由于B公司的基干业务很重要，所以公司决定保留。现在A公司的VPC内的实例需要访问B公司内的VPC中的资源。以下解决方案中最合适的是什么？（#1-21-C02）
   - [ ] A. 设置NAT实例，实现VPC间的通信。
   - [ ] B. 设置NAT网关，实现VPC间的通信。
   - [ ] C. 使用AWS Organizations。
   - [ ] D. 通过VPC Peering实现VPC间的通信。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 设置NAT实例，实现VPC间的通信。
   - [ ] B. 设置NAT网关，实现VPC间的通信。
   - [ ] C. 使用AWS Organizations。
   - [x] D. 通过VPC Peering实现VPC间的通信。
   
   </details>

7. 您的公司于星期二临时决定将本地文件服务器中的1TB的文件迁移到AWS中，迁移作业在当周星期五开始下周一结束，合计72小时。此次迁移需要确保数据的安全。请选择最合适解决方案（#1-39-C02）
   - [ ] A. 使用Snowball。
   - [ ] B. 使用Direct Connect。
   - [ ] C. 与AWS站点建立VPN，并且使用VPN传输。
   - [ ] D. 使用Storag Gateway。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 使用Snowball。
   - [ ] B. 使用Direct Connect。
   - [x] C. 与AWS站点建立VPN，并且使用VPN传输。
   - [ ] D. 使用Storag Gateway。
   
   </details>

8. 您所在的公司使用的VPC拥有一个私有子网和一个公有子网，专有子网中有一个EC2实例充当数据库服务器，公有子网中有一个NAT实例，供私有子网中的服务器与因特网通信。作为维护人员，最近您发现NAT可用性不是很高，那么请问您怎样改善？（#1-55-C02）
   - [ ] A. 扩充带宽。
   - [ ] B. 使用VPC Endpoint。
   - [ ] C. 用NAT Gateway取代NAT实例。
   - [ ] D. 提高NAT实例的性能。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 扩充带宽。
   - [ ] B. 使用VPC Endpoint。
   - [x] C. 用NAT Gateway取代NAT实例。
   - [ ] D. 提高NAT实例的性能。
   
   </details>

9. 您所在的公司员工通过部署在私有子网中的EC2实例访问S3并获取机密文件。公司的政策是这一过程不允许通过因特网，请问怎样解决？（#1-59-C02）
   - [ ] A. 设置安全组以实现需求。
   - [ ] B. 对S3的数据进行加密。
   - [ ] C. 使用VPC Endpoint。
   - [ ] D. 使用NAT实例。
   <details>
   <summary>查看答案</summary>
   
   - [ ] A. 设置安全组以实现需求。
   - [ ] B. 对S3的数据进行加密。
   - [x] C. 使用VPC Endpoint。
   - [ ] D. 使用NAT实例。
   
   </details>

10. 您正在为公司构筑VPC。该VPC拥有一个公共子网，并且使用Internet Gateway。但是您发现通过SSH无法连接公共子网中的EC2，但是您已经为该EC2设置了公共IP，请问如何解决上述问题？（#1-64-C02）
    - [ ] A. 检查路由表是否正确。
    - [ ] B. 设置弹性IP。
    - [ ] C. 为EC2设置第二IP。
    - [ ] D. 设置NAT Gateway。
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 检查路由表是否正确。
    - [ ] B. 设置弹性IP。
    - [ ] C. 为EC2设置第二IP。
    - [ ] D. 设置NAT Gateway。
   
    </details>

11. 您正在为公司系统构筑VPC，其包含两个公共子网和两个私有子网，目前公司规模200人，不想过多并且合适的VPC的CIDR设置应该是？（#2-27-C02）
    - [ ] A. /21。
    - [ ] B. /22。
    - [ ] C. /23。
    - [ ] D. /24。
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. /21。
    - [ ] B. /22。
    - [ ] C. /23。
    - [x] D. /24。
    
    </details>

12. （多选）您的公司正在开发一款利用神经网络预测故障系统，您正在为该系统设计VPC，该VPC含有两个私有子网和一个公共子网，其中一个私有子网中含有EC2实例作为神经网络计算服务器对数据进行计算，另一个私有子网中含有一个RDS用来存计算数据，公共子网中EC2实例作为堡垒机，将预测结果展示到浏览器中，这时您这台nginx服务器无法与私有子网中的计算服务器通信，解决这个问题的可行性方案有哪些？。（#2-28-C02）
    - [ ] A. 在公共子网中设置NAT网关。
    - [ ] B. 重新设置一个子网并且设置Internet网关，连接nginx服务器。
    - [ ] C. 在私有子网中设置NAT网关。
    - [ ] D. 在私有子网中设置NAT实例。
    - [ ] E. 在私有子网中设置Internet网关。
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 在公共子网中设置NAT网关。
    - [x] B. 重新设置一个子网并且设置Internet网关，连接nginx服务器。
    - [ ] C. 在私有子网中设置NAT网关。
    - [ ] D. 在私有子网中设置NAT实例。
    - [ ] E. 在私有子网中设置Internet网关。
    
    </details>

13. （多选）能实现本地数据迁移到AWS的方法有哪些？。（#2-57-C02）
    - [ ] A. Direct Connect。
    - [ ] B. IPsec VPN。
    - [ ] C. VPC Peering。
    - [ ] D. Storage Gateway。
    - [ ] E. VPN CloudHub。
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. Direct Connect。
    - [x] B. IPsec VPN。
    - [ ] C. VPC Peering。
    - [ ] D. Storage Gateway。
    - [x] E. VPN CloudHub。
    
    </details>

14. （多选）您正在构筑一套基于AWS的WEB应用，这个Web应用可以通过Internet访问，此系统使用了RDS服务的MySQL数据库，和一个EC2实例作为Web服务器，并连接RDS获取数据。公司规定数据库必须与Internet隔离，以下方法中安全性高的解决方案有哪些？。（#3-9-C02）
    - [ ] A. 在公有子网中加入NAT网关。
    - [ ] B. 在私有子网中加入NAT网关。。
    - [ ] C. 将EC2实例加入到私有子网中，安全组进站访问的HTTP的CIRD设置为0.0.0.0/0，DB的安全组中的MySQL访问设置为允许状态。
    - [ ] D. 将EC2实例加入到私有子网中，安全组进站访问的HTTPS的CIRD设置为10.0.0.0/16，DB的安全组中的SSL访问设置为允许状态。
    - [ ] E. 将EC2实例加入到私有子网中，安全组中MySQL访问设置成允许状态，DB的安全组中的加入EC2的IP地址。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. 在公有子网中加入NAT网关。
    - [ ] B. 在私有子网中加入NAT网关。。
    - [ ] C. 将EC2实例加入到私有子网中，安全组进站访问的HTTP的CIRD设置为0.0.0.0/0，DB的安全组中的MySQL访问设置为允许状态。
    - [x] D. 将EC2实例加入到私有子网中，安全组进站访问的HTTPS的CIRD设置为10.0.0.0/16，DB的安全组中的SSL访问设置为允许状态。
    - [x] E. 将EC2实例加入到私有子网中，安全组中MySQL访问设置成允许状态，DB的安全组中的加入EC2的IP地址。
    
    </details>

15. 您的公司要为现有VPC的子网进行扩容，要求同时能连接VPC的主机数扩大24台，请问IPv4的CIDR应该是哪一个？（#3-43-C02）
    - [ ] A. 168.0.0.0/27。
    - [ ] B. 168.0.0.0/29。
    - [ ] C. 168.0.0.0/30。
    - [ ] D. 168.0.0.0/32。
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 168.0.0.0/27。
    - [ ] B. 168.0.0.0/29。
    - [ ] C. 168.0.0.0/30。
    - [ ] D. 168.0.0.0/32。
    
    </details>

16. （多选）您的公司拥有两套VPC（A和B）,这两个VPC通过`VPC Peering`进行通信，A只有私有子网，B只有公有子网。现在公司为了将本地服务器与A相连，已经使用了一个Direct Connect和虚拟接口。现在为了提高容错率，您需要怎么解决？（#4-4-C02）
    - [ ] A. B同另外一个Direct Connect和虚拟接口相连。
    - [ ] B. A同另外一个Direct Connect和虚拟接口相连。
    - [ ] C. 本地服务器通过硬件VPN与A相连。
    - [ ] D. 本地服务器通过硬件VPN与B相连。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. B同另外一个Direct Connect和虚拟接口相连。
    - [x] B. A同另外一个Direct Connect和虚拟接口相连。
    - [x] C. 本地服务器通过硬件VPN与A相连。
    - [ ] D. 本地服务器通过硬件VPN与B相连。
    
    </details>

17. 您正在利用AWS构建Web应用，该应用将通过VPC与DynamoDB相连接，实现这一需求的服务是什么？（#4-16-C02）
    - [ ] A. 网关VPC终端节点。
    - [ ] B. VPC Peering。
    - [ ] C. VPC API Gateway。
    - [ ] D. VPC终端节点服务（AWS PrivateLink）。
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 网关VPC终端节点。
    - [ ] B. VPC Peering。
    - [ ] C. VPC API Gateway。
    - [ ] D. VPC终端节点服务（AWS PrivateLink）。
    
    </details>

18. （多选）您的公司托管的Web应用利用了两个部署在不同子网的EC2中，一个EC2实例运行数据库，另一个EC2实例是连接到数据库的Web服务器。这两个EC2必须相互通信才能使应用正常运行。为了使这两个EC2互相通信，您需要确认哪些事项？（#4-58-C02）
    - [ ] A. 这两个子网的ACL有没有配置好。
    - [ ] B. 安全组的许可设定有没有配置好。
    - [ ] C. 安全组的拒绝设定有没有配置好。
    - [ ] D. 这两个EC2有没有配置公共IP地址。
    - [ ] E. 这两个EC2的放置群组有没有配置好。
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 这两个子网的ACL有没有配置好。
    - [x] B. 安全组的许可设定有没有配置好。
    - [ ] C. 安全组的拒绝设定有没有配置好。
    - [ ] D. 这两个EC2有没有配置公共IP地址。
    - [ ] E. 这两个EC2的放置群组有没有配置好。
    
    </details>

19. 您正在加强公司VPC的安全性，为了避免非法访问，你在VPC的ACL中设置了多个规则，规则是如何让评估的？（#4-62-C02）
    - [ ] A. 从编号最高的规则开始进行，只要有一条规则与流量匹配，即应用该规则，并忽略与之冲突的任意更小编号的规则。
    - [ ] B. 从编号最小的规则开始进行，只要有一条规则与流量匹配，即应用该规则，并忽略与之冲突的任意更大编号的规则。
    - [ ] C. 所有规则都会被应用。
    - [ ] D. 从编号最高的规则开始进行，只要有一条规则与流量匹配，规则通过认证后应用该规则，并忽略与之冲突的任意更小编号的规则。
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. 从编号最高的规则开始进行，只要有一条规则与流量匹配，即应用该规则，并忽略与之冲突的任意更小编号的规则。
    - [ ] B. 从编号最小的规则开始进行，只要有一条规则与流量匹配，即应用该规则，并忽略与之冲突的任意更大编号的规则。
    - [ ] C. 所有规则都会被应用。
    - [ ] D. 从编号最高的规则开始进行，只要有一条规则与流量匹配，规则通过认证后应用该规则，并忽略与之冲突的任意更小编号的规则。
    
    </details>

20. 您正在为公司构筑VPC。此VPC中有两个公共子网和两个私有子网，私有子网中的EC2安装了MySQL充当数据库服务器，公共子网中的EC2安装了Apache服务器充当前端服务器，您需要在家中通过SSH登录到数据服务器，请问您应该怎样设置？ （#4-63-C02）
    - [ ] A. 在私有子网中加入NAT网关。
    - [ ] B. 为数据库服务器配置弹性IP。
    - [ ] C. 正确配置路由表。
    - [ ] D. 在公有子网中加入NAT网关。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. 在私有子网中加入NAT网关。
    - [ ] B. 为数据库服务器配置弹性IP。
    - [x] C. 正确配置路由表。
    - [ ] D. 在公有子网中加入NAT网关。
    
    </details>

21. 作为架构助理，您接手了某知名电商网站迁移到VPC的任务。之前的架构师已经部署了三层VPC，它们ID分别是：
    - VPC： vpc-2f8tLC447
    - IGW： igw-2d8bc445
    - ACL： acl-2080c448
    - Web服务器子网： subnet-258bc44d
    - 后端服务器子网： subnet-248bc44c
    - 数据库服务器： subnet-9189c6f9
    - 路由表： rtb-2i8bc449和rtb-238bc44b
    
    子网和路由表的关联关系是：
    
    - subnet-258bc44d关联rtb-2i8bc449
    - subnet-248bc44c关联rtb-238bc44b
    - subnet-9189c6f9关联rtb-238bc44b
    
    数据库服务器不允许直接访问Internet。以下哪些配置可以实现远程SSH登录后端服务器和数据库服务器以及数据库服务器和后端服务器的包更新？（#2-9-C01）
    - [ ] A. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到`subnet-258bc44d`的路由。
    - [ ] B. 在子网`subnet-248DC44c`中创建一套堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到`igw-2d8bc445`的路由。
    - [ ] C. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例,并且加入从`rtb-238bc44d`到`igw-2d8bc445`的路由，并且在ACL加入允许`subnet-258bc44d`和`subnet-248bc44c`相互通信的规则。
    - [ ] D. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到NAT实例的路由。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到`subnet-258bc44d`的路由。
    - [ ] B. 在子网`subnet-248DC44c`中创建一套堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到`igw-2d8bc445`的路由。
    - [ ] C. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例,并且加入从`rtb-238bc44d`到`igw-2d8bc445`的路由，并且在ACL加入允许`subnet-258bc44d`和`subnet-248bc44c`相互通信的规则。
    - [x] D. 在子网`subnet-258bc44d`中创建一台堡垒机和一个NAT实例，并且加入从`rtb-238bc44b`到NAT实例的路由。
    
    </details>

22. 作为助理架构师您正在为某电商网站做架构。该网络服务器运行在横跨多个可用区的Auto Scaling group的EC2中，并且需要实时从数据库服务器读写数据。该数据库服务器不允许直接连接Internet但是需要定期更新软件包，下列哪些设计能满足以上需求？（#2-11-C01）
    - [ ] A. 将网络服务器和数据库服务器加入到公共子网中。
    - [ ] B. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网。
    - [ ] C. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网，在公有子网中加入NAT网关。
    - [ ] D. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网，在私有子网中加入NAT网关。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. 将网络服务器和数据库服务器加入到公共子网中。
    - [ ] B. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网。
    - [x] C. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网，在公有子网中加入NAT网关。
    - [ ] D. 将网络服务器加入到公共子网，将数据库服务器加入到私有子网，在私有子网中加入NAT网关。
    
    </details>
    
 
23. 某银行系统后端服务器需要配置到私有子网中。该服务器需要时常获取S3中的数据并且将新数据保存到S3中，但是不允许与Internet相连，最佳的解决方案是什么？（#2-20-C01）
    - [ ] A. Gateway VPC Endpoint。
    - [ ] B. NAT Gateway。
    - [ ] C. 为该服务器部署一台代理服务器
    - [ ] D. 使用私有S3桶
    
    <details>
    <summary>查看答案</summary>
   
    - [x] A. Gateway VPC Endpoint。
    - [ ] B. NAT Gateway。
    - [ ] C. 为该服务器部署一台代理服务器
    - [ ] D. 使用私有S3桶
    
    </details>
    
24. 作为助理架构师，您发现您的公司现有的安全组的配置过于宽松，可能会允许本应被拒绝的访问。现在的数据库的安全组配置如下：

    | 类型  | 协议 | 端口范围 | 源        |
    |-------|------|----------|-----------|
    | MSSQL | TCP  | 1433     | 0.0.0.0/0 |
    
    唯一需要连接到数据库的是由EC2实例组成的Auto Scaling组。怎样的设置能让用户拥有最低限的权限？（#2-59-C01）
    - [ ] A. 将源设置为-1。
    - [ ] B. 将源设置为该VPC的CIDR。
    - [ ] C. 将源设置为Auto Scaling组的id。
    - [ ] D. 将源设置为Auto Scaling组的安全组的id。
    
    <details>
    <summary>查看答案</summary>
   
    - [ ] A. 将源设置为-1。
    - [ ] B. 将源设置为该VPC的CIDR。
    - [ ] C. 将源设置为Auto Scaling组的id。
    - [x] D. 将源设置为Auto Scaling组的安全组的id。
    
    </details>
    

### 解答
