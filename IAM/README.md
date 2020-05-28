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

5. 您的公司正在经营一套托管在AWS上的系统。您决定引入新的API网关来实现应用间的协作。因此，作为API网关的权限管理，有必要将适当级别的权限授予开发人员，IT管理员和用户以进行管理。请选择最合适的设置方法来实实现上述权限管理？（#1-25-C02）
   - [ ] A. 利用STS对不同的用户授予访问权。
   - [ ] B. 利用IAM策略对不同的用户授予访问权。
   - [ ] C. 利用AWS Config对不同的用户授予访问权。
   - [ ] D. 利用AWS用户访问密匙对不同的用户授予访问权。

6. 以下IAM策略用于设置EC2实例的权限：

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Action": "ec2:*",
         "Effect": "Allow",
         "Resource": "*"
       },
       {
         "Effect": "Deny",
         "Action": [
           "ec2:*ReservedInstances*",
           "ec2:TerminateInstances"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

   下列描述正确的是：（#1-32-C02）

   - [ ] A. 允许对预留实例（EC2 RI）的操作。
   - [ ] B. 关于EC2实例的所有操作都是被允许的。
   - [ ] C. 对于所有的EC2，终止操作是被拒绝的。
   - [ ] D. 对于所有的预留实例（EC2 RI），终止操作是被拒绝的。

7. 您是构建某移动应用程序的解决方案架构师。此移动应用程序允许您将照片存储在S3上，并使用与OpenID Connect兼容的身份提供程序登录到您的应用程序。AWS STS通过授予权限的机制用于用户的临时访问。选择如何为此应用程序配置AWS STS。（#3-38-C02）
   - [ ] A. 授予跨账户访问权限。
   - [ ] B. SAML ID联合验证。
   - [ ] C. Web ID联合验证。
   - [ ] D. IAM角色设定。

8. 您的公司使用AWS Organizations在OU的基础上实施访问控制。您在OU1中添加了一个设置，允许对EC2和S3拥有所有权。接下来，对于OU1中的IAM用户，您设置了以下策略：仅EC2的删除权限为Deny，其他EC2操作为允许。除此之外再没有其他策略。请选择该IAM用户的权限状态。（#3-41-C02）
   - [ ] A. EC2的操作中，仅删除是被禁止的。
   - [ ] B. EC2和S3的所有操作都被允许。
   - [ ] C. EC2的操作中，仅删除是被禁止的，S3的所有操作都是被允许的。
   - [ ] D. 只有EC2的所有操作是被允许的。

9. 公司A使用一个AWS账户（ID890209911），并通过划分IAM组在多个部门中使用AWS资源。我们为开发组分配一个描述性标识符，并发布在AWS管理页面的登录页面的URL上。使用AWS CLI命令`iam create-account-alias --account-alias development-group`选择创建的AWS账户的登录页面URL是？（#3-55-C02）
   - [ ] A. `https://development-group.signin.aws.amazon.com/console/`
   - [ ] B. `https://development-group.aws.amazon.com/console/`
   - [ ] C. `https://development-group.aws.signin.amazon.com/console/`
   - [ ] D. `https://890209911.signin.aws.amazon.com/console/`

10. 您的客户刚刚选用AWS进行服务托管。该公司通过Active Directory实现权限管理，但是有必要通过AWS IAM有效地将其与权限管理一起使用，怎样实现这一需求？（#3-61-C02）
    - [ ] A. Simple AD
    - [ ] B. AWS Cognito
    - [ ] C. AWS Inspector
    - [ ] D. AWS Connector
