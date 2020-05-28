# IAM

## 概述

AWS Identity and Access Management(IAM)是一种Web服务，可以帮助您安全地控制对AWS资源的访问。您可以使用IAM控制对哪个用户进行身份验证(登录)和授权 (具有权限)以使用资源。[官方文档传送门](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/introduction.html)。

## 模拟题

1. 关于以下IAM策略：

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Action": "*",
         "Resource": "*",
         "Condition": {
           "NotIpAddress": {
             "aws:SourceIp": [
               "172.101.1.36/32"
             ]
           }
         }
       }
     ]
   }
   ```

   下列描述正确的是：(#2-8-C02)

   - [ ] A. 除IP地址`172.101.1.36`以外的所有资源的访问都会被拒绝。
   - [ ] B. IP地址`172.101.1.36`可以访问一切资源。
   - [ ] C. 除IP地址`172.101.1.36`以外的所有资源的访问都会被允许。
   - [ ] D. 以上说法都不对。

2. 以下SCP用于为特定的OU中的AWS资源设置权限：

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:*",
           "cloudwatch:*"
         ],
         "Resource": "*",
       }
     ]
   }
   ```

   以下描述正确的是：（#2-31-C02）
   - [ ] A. 成员账户中设置此SCP的所有OU用户都拥有完整的EC2特权。
   - [ ] B. 成员账户中设置此SCP的所有OU用户都可以设置EC2和CloudWatch的完全访问权。
   - [ ] C. 成员账户中设置此SCP的所有OU用户所有的的EC2特权都不拥有。
   - [ ] D. 成员账户中设置此SCP的所有OU用户都无法设置EC2和CloudWatch的完全访问权。

3. 一家公司正在构建语音响应系统，该系统可在接到查询电话时将电话路由到适当的人。该系统在多可用区部署了EC2实例，并配置了Auto Scaling组，ALB和RDS实例的组合。您需要利用身份验证令牌来保护您的客户数据，以便您只能通过存储于EC2实例的配置文件中的认证信息访问RDS数据库。怎样实现这一需求？（#2-38-C02）
   - [ ] A. 使用IAM认证数据库来授予RDS的访问权。
   - [ ] B. 利用SSL对RDS的访问进行加密。
   - [ ] C. 通过对EC2实例上设置IAM角色来授予对RDS的访问权。
   - [ ] D. 通过STS临时授予RDS的访问权，

4. 一家公司在本地和AWS云中都托管系统。该公司需要使用存储在Active Directory中的内部凭据来确保所有用户都可以在两种环境中访问资源。能实现这一需求的方法是什么？（#2-42-C02）
   - [ ] A. IAM组。
   - [ ] B. AWS Cognito。
   - [ ] C. AWS Organization中的Active Directory服务。
   - [ ] D. AWS SAML。
