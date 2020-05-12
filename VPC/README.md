# VPC

## 概述

1. [官方文档](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/what-is-amazon-vpc.html)

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
   
### ACL

- 网络访问控制列表 (ACL) 是 VPC 的一个可选安全层，可用作防火墙来控制进出一个或多个子网的流量。

## 模拟题

### 问题

1. 一家公司需要记录私有子网中所有IP包（源，目标，协议），最佳解决方案是什么？（#1-8-C01）
   - [ ] A. 使用[VPC flow logs](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/flow-logs.html)。
   - [ ] B. 使用EC2上的`source destination checkout`。
   - [ ] C. 使用[AWS CloudTrail](https://docs.aws.amazon.com/zh_cn/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)并且使用S3存储日志文件。
   - [ ] D. 使用[Amazon CloudWatch](https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)进行监控
   
2. 一个应用运行在私有子网的EC2实例上，这个应用需要读写`Amazon Kinesis Data Streams`上的数据。但是该公司要求读取数据的流不能流向万维网（Internet），最佳解决方案是什么？（#1-39-C01）
   - [ ] A. 在一个共有子网中配置一个[NAT网关（NAT Gateway）](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/vpc-nat-gateway.html)并且将读写流路由到`Kinesis`服务上。
   - [ ] B. 为`Kinesis`配置一个[网关 VPC 终端节点（Gateway VPC Endpoint）](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/vpce-gateway.html)并且通过其将读写流路由到`Kinesis`服务上。
   - [ ] C. 为`Kinesis`配置一个[接口 VPC 终端节点（Interface VPC Endpoint）](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/vpce-interface.html)并且通过其将读写流路由到`Kinesis`服务上。
   - [ ] D. 为`Kinesis`配置一个[AWS Direct Connect 虚拟接口](https://docs.aws.amazon.com/zh_cn/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html)并且通过其将读写流路由到`Kinesis`服务上。

3. 您启动了一个实例并且将它用作在公共子网中NAT设备。接着你修改路由表，在私有子网中将此NAT设备变更成互联网流的目标，当你试图使用此私有子网中的实例去连接外网（outbount connection）,你发现这并不成功，什么操作能解决这个问题？（#1-47-C01）
   - [ ] A. 为这个NAT实例添加[弹性网络接口（Elasitc Network Interface）](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-eni.html)，并把它加入到私有子网中。
   - [ ] B. 为这个NAT实例添加[弹性IP地址（Elasitc IP Address）](hhttps://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)。
   - [ ] C. 为这个NAT实例添加另外一个弹性网络接口（Elasitc Network Interface）并把它加入到公有子网中。
   - [ ] D. 停止这个NAT实例上的[Source/Destination Check](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/VPC_NAT_Instance.html)功能。

4. 作为架构师，您正在使用AWS进行数据库服务器托管，除了必要的升级文件需要下载以外，这台服务器不允许链接因特网，以下哪项服务适合该项需求？（#1-7-C02）
   - [ ] A. 将数据库服务器配置在公有子网中，使用ACL将网络设置为只允许入站流量通过。
   - [ ] B. 将数据库服务器配置在公有子网中，使用安全组将网络设置为只允许入站流量通过。
   - [ ] C. 将数据库服务器配置在私有子网中，使用安全组将网络设置为只允许出站流量通过。
   - [ ] D. 将数据库服务器配置在私有子网中，并在路由表中配置NAT实例。

5. 您所在的公司将要将本地文件服务器的数据迁移到AWS中。本地文件服务器的数据量很大，为了节省工期，数据迁移时需要50Mbps的速度，下列哪种方法最适合？（#1-13-C02）
   - [ ] A. VPC Peering。
   - [ ] B. VPN。
   - [ ] C. AWS Direct Connect。
   - [ ] D. NAT

### 解答
