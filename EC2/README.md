# EC2

本文档主要介绍EC2主要概念和相关服务以及AWS SAA考试中的相关习题。您可以先参照[官方文档](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/concepts.html
)进行学习。

## 区域、可用区和本地区域

### 本地区域

- CloudFront
- Lambda

### 区域间连接

区域间连接一般为了完成故障排除以及备份等工作，并且需要专用网络，以下是能够实现区域间连接或者复制的服务。

1. AWS Direct Connect Gateway
2. Inter Region VPC Peering
3. EC2区域间AMI复制
4. S3区域间替代
5. RDS区域间复制
6. DynamoDB区域间复制
7. 通过Route53进行故障转移（Failover）