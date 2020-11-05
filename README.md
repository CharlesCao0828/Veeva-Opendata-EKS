# Veeva-Opendata-EKS

## 实验内容
熟悉并了解EKS服务的管理和运维，在AWS云平台上搭建EKS集群，在集群上部署示例应用，并完成集群版本升级、计算节点实例类型调整、计算节点数量调整、工作节点组添加、网络与存储管理、集群日志管理、集群删除等任务。

## 实验环境
创建cloud9实例，将cloud9实例当作本实验的客户端，完成与EKS集群相关的任务。

## 实验步骤
### 一、创建Cloud9实例
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
### 二、创建EKS集群
- 步骤一
安装eksctl客户端，eksctl是EKS服务提供的客户端工具，通过eksctl，用户可以对EKS集群以及工作节点的生命周期进行管理，如添加、删除、升级、参数配置等。
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
- 步骤二
安装kubectl客户端，EKS提供与开源Kubernetes一致的API，用户在完成Kubernetes集群创建后，可以利用kubectl命令行对集群管理。
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

- 步骤三

安装“aws-iam-authenticator”，EKS集群内置AWS开发的安全插件，改安全插件用来控制用户对EKS集群的访问与资源的操作。改插件可以将IAM Role与Kubernetes自身的角色控制机制相结合，从而通过IAM Role来限制用户对集群的访问。“aws-iam-authenticator”工具可以用来与AWS IAM以及EKS安全插件交互，“aws-iam-authenticator”与kubectl进行集成，用户在kubeconfig文件中配置与“aws-iam-authenticator”相关的参数，当用户运行kubectl命令行时，kubectl调用“aws-iam-authenticator”完成集群对用户的认证。
```
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv ./aws-iam-authenticator /usr/bin/aws-iam-authenticator
```

- 步骤四
利用eksctl命令创建EKS集群，eksctl支持命令行创建集群，也支持通过定义配置文件创建EKS集群。利用eksctl命令创建集群的好处是简单、快捷。利用配置文件创建集群的好处是可以定义复杂配置、同时文件可重用。
```
eksctl create cluster --name=eks-workshop-<“yourname”> --nodes-min=3 --nodes-max=5 --node-type=m5.xlarge  
```
- 步骤五
集群的创建大概耗时20分钟，待集群成功创建后，eksctl会自动在客户端的～/.kube路径下配置config文件。用户可以直接通过kubectl与EKS集群进行交互,从而验证集群是否安装成功。
```
## 查看Kubernetes集群内工作节点数量
kubectl get node
## 查看Kubernetes集群信息
kubectl cluster-info
## 查看集群内服务
kubectl get svc
```

### 三、集群规模调整
- EKS集群实例类型调整
- EKS集群实例数量调整
- 新增EKS工作节点组

### 四、集群版本升级


### 五、网络通信管理

### 六、存储资源管理

### 七、集群日志管理

