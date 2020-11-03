# Veeva-Opendata-EKS

## 实验内容
熟悉并了解EKS服务的管理和运维，在AWS云平台上搭建EKS集群，在集群上部署示例应用，并完成集群版本升级、计算节点实例类型调整、计算节点数量调整、工作节点组添加、网络与存储管理、集群日志管理、集群删除等任务。

## 实验环境
创建cloud9实例，将cloud9实例当作本实验的客户端，完成与EKS集群相关的任务。

## 实验步骤
### 创建Cloud9实例
- 步骤一
登录AWS Console，单击进入cloud9服务界面。
- 步骤二
单击创建环境，开始cloud9实例的创建。
- 步骤三
为cloud9实例命名，命名规则为myide-<"your-name">
- 步骤四
在网络设置中，选择帐号下默认的VPC，并任意选择一个subnet。
- 步骤五
在完成Cloud9的创建后，单击中央面板处上方的+图标，并选择终端。
- 步骤六
单击cloud9面板右侧上方的“Preferences”设置图标，单击AWS Settings并禁用“AWS managed temporary credentials”。
- 步骤七
获取AWS_ACCESS_KEY与AWS_SECRET_ACCESS_KEY的信息，运行“aws configure”命令对key进行配置。
```
    aws configure
```
