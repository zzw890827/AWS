# Related knowledges

## VPC

### 概述

1. [官方文档](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/what-is-amazon-vpc.html)

### CIDR

[CIDR计算器](http://www.subnet-calculator.com/cidr.php)

### 相关习题

1. 一家公司需要记录私有子网中所有IP包（源，目标，协议），最佳解决方案是什么？
  - [ ] A. 使用[VPC fow logs](https://docs.aws.amazon.com/zh_cn/vpc/latest/userguide/flow-logs.html)。
  - [ ] B. 使用EC2上的`source destination checkout`。
  - [ ] C. 使用[AWS CloudTrail](https://docs.aws.amazon.com/zh_cn/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)并且使用S3存储日志文件。
  - [ ] D. 使用[Amazon CloudWatch](https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)进行监控

## AWS Outposts

### 概述

AWS Outposts 是一项完全托管的服务，可将 AWS 基础设施、AWS 服务、API 和工具扩展到几乎任何数据中心、共处空间或本地设施，以实现真正一致的混合体验。AWS Outposts 非常适合**需要低延迟访问本地系统、本地数据处理或本地数据存储的工作负载**。

### 应用

AWS Outposts 解决了多种行业中的低延迟应用程序需求和本地数据处理需求。
