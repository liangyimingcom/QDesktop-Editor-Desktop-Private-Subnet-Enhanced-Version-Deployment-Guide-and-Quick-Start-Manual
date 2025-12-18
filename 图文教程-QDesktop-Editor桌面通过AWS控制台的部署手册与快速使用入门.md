# QDesktop-Editor桌面，通过AWS控制台的部署手册（图文指南）与快速使用入门

## 1、了解您的AWS账号下的部署环境，确认要部署的目标vpc和私有子网

1）建议用管理员IAM权限账号登录AWS控制台，然后选择您已经创建了私有子网的AWS Region（建议考虑 us-east-1 美东1 region）。

![image-20251217181152773](./assets/image-20251217181152773.png)



2）然后打开VPC服务，点击《VPC dashboard》，查看你要部署的VPC，从里面找到合适的 private-subnet 私有子网（如下图）。

![image-20251217175830889](./assets/image-20251217175830889.png)

这个示例你能看到：

1. 创建的VPC为 CIDRs  ”10.162.237.64/26“ 的网段（可以理解为：业务部门申请了私有网段并且已经接入企业内网 - 通过DX专线互联）； 
2. QDesktop 可以被部署在“10.162.237.80/28”  - 名字为“private-subnet-1”的私有子网；
3. “private-subnet-1”这个私有子网，已经配置了一个NAT网关 - 这意味私有子网里面的EC2可以从公网下载并更新开发组件包（非常重要！）；



## 2、通过CloudFormation服务进行图形化部署QDesktop

1）打开cloudformation服务，点击“CloudFormation -> Stacks -> Create stack”，点击《Choose file》后选择《qdesktop-editor-privatesubnet-fullfeatures-enhanced-20250918.yaml》，如下图：

![image-20251217224658535](./assets/image-20251217224658535.png)

2）Specify stack details 的关键属性：

1. Stack name & Instance name 建议用员工变化或姓名命名，如：qdesktop-username-**zhangsan**
2. Network Configuration： VPC ID & Private Subnet ID 指定了QDesktop部署的目标VPC和子网，选择与上面《VPC dashboard》 一致的内容；VPC CIDR 指的是安全组的访问来源范围，不清楚如何配置就用缺省参数；
3. 其他参数：可以保留缺省参数，无需变更；

示例截图如下：

![image-20251217225650191](./assets/image-20251217225650191.png)

3）点击下一步后，然后圈选“I acknowledge that AWS CloudFormation might create IAM resources.”后点击下一步；![image-20251217225750431](./assets/image-20251217225750431.png)

4）点击Submit提交，然后等待部署状态从“ CREATE_IN_PROGRESS” 变为“CREATE_COMPLETE”，整个过程需要10-20分钟；

![image-20251217225828785](./assets/image-20251217225828785.png)

![image-20251217225906845](./assets/image-20251217225906845.png)

![image-20251217233215519](./assets/image-20251217233215519.png)

4）点击"Outputs" 分页栏，获取 “InstanceId”  = i-0e77c0893f7aa2470 ，并copy记录下来；

![image-20251217233353785](./assets/image-20251217233353785.png)

![image-20251217233419119](./assets/image-20251217233419119.png)



## 3、对QDesktop-Editor桌面进行权限配置

在AWS控制台中打开 EC2 服务，然后查找  “InstanceId”  = i-0e77c0893f7aa2470 ，然后鼠标右键 Security -> Modify IAM role.

![image-20251217233538780](./assets/image-20251217233538780.png)

如果您使用的AWS账号为测试环境，**由于您暂时不确认QDesktop会调用哪些AWS服务，建议可以临时 给Admin的IAM权限。** 等待任务范围确认后，更换为权限更小的IAM Role。 操作如下：

![image-20251217233746364](./assets/image-20251217233746364.png)

![image-20251217233815424](./assets/image-20251217233815424.png)

![image-20251217233842146](./assets/image-20251217233842146.png)

![image-20251217233858491](./assets/image-20251217233858491.png)

QDesktop-EC2-Admin

![image-20251217234001687](./assets/image-20251217234001687.png)

创建QDesktop-EC2-Admin Role成功后，返回刚才的EC2页面，然后点击刷新后，就可以选择《QDesktop-EC2-Admin》，然后点击“Update IAM role”；

![image-20251217234126860](./assets/image-20251217234126860.png)



## 4、登录 QDesktop-Editor桌面 ，完成初始化后正常使用

打开 CloudFormation -> Stacks -> qdesktop-username-zhangsan，然后打开Outputs分页栏：

![image-20251217234312507](./assets/image-20251217234312507.png)

点击CodeEditorURL后，就可以获得私有IP地址的登录网址**（请分发给最终用户这个地址使用即可。由于是内网IP，EC2关机后重启内网IP不变化 - 可以自由关机降低成本）**。

此时，您在本地网络已经配置了DX专线（或VPN）连接到 AWS VPC，输入您的浏览器后 - 直接通过浏览器访问：

```
http://PRIVATE_IP:8080/?folder=/workshop&tkn=PASSWORD
```

[QDesktop-Editor登录界面]

![image-20251217171143476](./assets/image-20251217171143476.png)

建议操作：

1. 第一次开机，运行以下命令更新 Amazon Q CLI  --> Kiro-CLI (功能一样，升级后名字改了)：

   ```bash
   $ q update
   ```
1. 运行以下命令登录 Amazon Q CLI：

   ```bash
   $ q login
   ```
1. 运行以下命令验证 Amazon Q CLI 是否正常工作：

   ```bash
   $ q chat
   ```

![image-20251218153819980](./assets/image-20251218153819980.png)





更多教程：因为当前QDeskop的EC2已经配置了 IAM Role Admin 的权限，对当前测试AWS账户有完整的访问权限。意味着QCLI可以创建/销毁AWS资源，您可以通过这些 《[基础实验](https://catalog.us-east-1.prod.workshops.aws/workshops/aa2f7dda-789c-4602-aebb-adbd2071db85/zh-CN/basic-lab)》来快速掌握QCLI基本功能和常见使用场景，以及如何使用Q CLI完成AWS操作中的基本问题。



## 5、结束



---

**版本信息：** v3.0.0 - 私有子网增强版  
**最后更新：** 2025年12月18日  
**兼容性：** AWS CloudFormation, Amazon Linux 2023, Ubuntu 22/24

