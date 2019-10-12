+++ 
draft = false
date = 2019-10-10T20:51:01+08:00
title = "重置 AWS EC2 的 SSH Keypair"
description = "重置 AWS EC2 的 SSH Keypair"
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++

最近创建了一个新的 EC2 实例，使用的之前的 keypair，结果登陆的时候才发现这个 keypair 的私钥文件没有了！！！而且挂载的这个 keypair 也无法再次导出。

使用 AWS 第一次遇到这个情况，按照使用其它云的经验，应该在控制台有重置密码的操作吧，找了一圈竟然没有！最后 Google 了一下，原来这个问题还真挺复杂，记录一下重置步骤。

# 1. 使用 CloudFormation 模板授权
AWS CloudFormation 通过使用预配置的模板来自动化创建 IAM 角色和策略的过程。
使用以下过程通过使用AWS CloudFormation为EC2Rescue Automation创建所需的IAM角色和策略。

1. 打开 CloudFormation 控制台: https://console.aws.amazon.com/cloudformation/home
2. 创建 Stack

    ![](/images/aws-cloudformation-1.png)

    输入 Amazon S3 URL 为 `https://s3.amazonaws.com/awssupport-ssm.us-east-2/cfn/EC2Rescue/AWSSupport-EC2RescueRole.template`
 
3. 指定 Stack 细节

    ![](/images/aws-cloudformation-2.png)

    只需输入 Stack name 即可

4. 然后一直下一步，最后创建即可

5. Stack 创建完成后复制 AssumeRole 的 ARN。在下一步运行 Automation 时将指定此 ARN。

    ![](/images/aws-cloudformation-3.png)

# 2. 运行 Automation
1. 打开 AWS Systems Manager 控制台: https://console.aws.amazon.com/systems-manager/
2. 进入 Automation -> Execute automation，选择 AWSSupport-ResetAccess

    ![](/images/aws-automation-1.png)

    选择 Document version 为最高的默认版本


3. 下一步进入 Execution mode -> Simple Execution

    ![](/images/aws-automation-2.png)

    输入参数 InstanceID 为 EC2 实例的 ID，参数 EC2RescueInstanceType 选择 t2.small
    参数 SubnetId 为 EC2 实例的 Subnet ID, 参数 Assume Role 为从 CloudFormation 控制台复制的 AssumeRole ARN

4. 执行

# 3. 重置成功
最后重新生成的私钥在 AWS Systems Manager -> Parameter Store 里面

![](/images/aws-automation-3.png)

进去就可以看到重新生成的私钥，然后就可以愉快的登陆了。


# 注意
- 确保 Instance 和 AWS Systems Manager 这两个页面是同一个 region, 在页面顶部导航栏可见

    ![](/images/aws-nav.png)

    不然在 Automation 阶段会遇到 `InvalidInstanceID.NotFound` 的错误。

    CloudFormation 页面没有强制需求，因为在 CloudFormation 阶段创建的 IAM Role 是全局资源。

# 原理
观察了下 Automation 的过程，大致原理如下:

起了一个相同配置的临时 EC2 实例，关闭原始实例，然后把原始实例的 EBS 卷挂载到临时实例上，重新生成一个 ssh keypair 到这个 EBS 卷上面，最后重新挂载卷到原始实例，重新启动和删除临时实例。

Automation 过程就是帮你自动化了这个过程。


# 参考
- How do I recover access to my EC2 instances if I've lost my SSH key pair? https://aws.amazon.com/premiumsupport/knowledge-center/recover-access-lost-key-pair/
- Reset Passwords and SSH Keys on Amazon EC2 Instances, https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-ec2reset.html