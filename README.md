# Veeva-Opendata-EKS

## Lab1、创建cloud9实例并安装、配置客户端
```
cat >> ~/.bashrc << EOF
export AWS_ACCESS_KEY_ID=XXX
export AWS_SECRET_ACCESS_KEY=XXX
export AWS_SESSION_TOKEN=XXX
export AWS_DEFAULT_REGION=XXX
EOF
source ~/.bashrc
```

## Lab2、创建EKS集群
### 二、创建EKS集群
- 步骤一：安装eksctl客户端，eksctl是EKS服务提供的客户端工具，通过eksctl，用户可以对EKS集群以及工作节点的生命周期进行管理，如添加、删除、升级、参数配置等。
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
- 步骤二：安装kubectl客户端，EKS提供与开源Kubernetes一致的API，用户在完成Kubernetes集群创建后，可以利用kubectl命令行对集群管理。
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

- 步骤三：安装“aws-iam-authenticator”，EKS集群内置AWS开发的安全插件，改安全插件用来控制用户对EKS集群的访问与资源的操作。改插件可以将IAM Role与Kubernetes自身的角色控制机制相结合，从而通过IAM Role来限制用户对集群的访问。“aws-iam-authenticator”工具可以用来与AWS IAM以及EKS安全插件交互，“aws-iam-authenticator”与kubectl进行集成，用户在kubeconfig文件中配置与“aws-iam-authenticator”相关的参数，当用户运行kubectl命令行时，kubectl调用“aws-iam-authenticator”完成集群对用户的认证。
```
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv ./aws-iam-authenticator /usr/bin/aws-iam-authenticator
```

- 步骤四：利用eksctl命令创建EKS集群，eksctl支持命令行创建集群，也支持通过定义配置文件创建EKS集群。利用eksctl命令创建集群的好处是简单、快捷。利用配置文件创建集群的好处是可以定义复杂配置、同时文件可重用。
```
eksctl create cluster --name=eks-workshop-<“yourname”> --nodes-min=3 --nodes-max=5 --node-type=m5.xlarge  
```
- 步骤五：集群的创建大概耗时20分钟，待集群成功创建后，eksctl会自动在客户端的～/.kube路径下配置config文件。用户可以直接通过kubectl与EKS集群进行交互,从而验证集群是否安装成功。
```
## 查看Kubernetes集群内工作节点数量
kubectl get node
## 查看Kubernetes集群信息
kubectl cluster-info
## 查看集群内服务
kubectl get svc
```

## Lab3、集群节点数量调整
- 步骤一：查看当前节点数量。
```
kubectl get node 
```
- 步骤二：运行命令，将集群节点数量由三台扩展至五台。
```
# 获取集群nodegroup名称
eksctl get nodegroup --cluster <your-cluster-name>
# 扩展集群节点数量
eksctl scale nodegroup <nodegroup-name>   <your-cluster-name>
```
- 步骤三：确认新增节点已成功添加至EKS集群。
```
kubectl get node 
```
## Lab4、新创建EKS工作节点组
- 步骤一：为当前EKS集群新添加一个实例类型为“t2.medium”，数量为二的新的工作节点组。
```
eksctl create nodegroup --name new-node-group --node-type t2.medium --nodes-min 2 --nodes 2 --cluster <your-cluster-name>
```
- 步骤二：查看新增节点组，并确认新增节点组已成功添加至EKS集群。
```
# 查看新创建的nodegroup
eksctl get nodegroup --cluster  <your-cluster-name>    
# 确认新增节点已经成功添加至EKS集群
kubectl get node -o wide
```
- 步骤三：删除新增加的节点组。
```
eksctl delete nodegroup new-node-group   --cluster <your-cluster-name> 
```

## Lab5、查看EKS集群管理节点日志
- 步骤一：启用EKS集群管理节点日志，启用日志后，可以到EKS控制界面进行查看。
```
步骤一：启用EKS集群管理节点日志，启用日志后，可以到EKS控制界面进行查看。
```


## Lab6、新增Windows类型节点，并安装Windows应用。
- 步骤一：开启集群对Windows系统的支持。
```
sudo yum install jq
kubectl apply -f https://amazon-eks.s3.us-west-2.amazonaws.com/manifests/us-west-2/vpc-resource-controller/latest/vpc-resource-controller.yaml

curl -o webhook-create-signed-cert.sh https://amazon-eks.s3.us-west-2.amazonaws.com/manifests/us-west-2/vpc-admission-webhook/latest/webhook-create-signed-cert.sh
curl -o webhook-patch-ca-bundle.sh https://amazon-eks.s3.us-west-2.amazonaws.com/manifests/us-west-2/vpc-admission-webhook/latest/webhook-patch-ca-bundle.sh
curl -o vpc-admission-webhook-deployment.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/manifests/us-west-2/vpc-admission-webhook/latest/vpc-admission-webhook-deployment.yaml

chmod +x webhook-create-signed-cert.sh webhook-patch-ca-bundle.sh
./webhook-create-signed-cert.sh
cat ./vpc-admission-webhook-deployment.yaml | ./webhook-patch-ca-bundle.sh > vpc-admission-webhook.yaml
kubectl apply -f vpc-admission-webhook.yaml
cat > eks-kube-proxy-windows-crb.yaml << EOF
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: eks:kube-proxy-windows
  labels:
    k8s-app: kube-proxy
    eks.amazonaws.com/component: kube-proxy
subjects:
  - kind: Group
    name: "eks:kube-proxy-windows"
roleRef:
  kind: ClusterRole
  name: system:node-proxier
  apiGroup: rbac.authorization.k8s.io
EOF
kubectl apply -f eks-kube-proxy-windows-crb.yaml
```
- 步骤二：创建操作系统类型为Windows的工作节点组。
```
eksctl create nodegroup \
  --cluster <your-cluster-name> \
  --name ng-windows \
  --node-type t2.large \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --node-ami-family WindowsServer2019FullContainer
```
- 步骤三：查看Windows工作节点组安装状态。
```
eksctl get nodegroup ng-windows --cluster <your-cluster-name>
```

- 步骤四：将Windows应用容器部署至Windows类型的工作节点上。
```
cat > windows-server-iis.yaml << "EOF"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: windows-server-iis
spec:
  selector:
    matchLabels:
      app: windows-server-iis
      tier: backend
      track: stable
  replicas: 1
  template:
    metadata:
      labels:
        app: windows-server-iis
        tier: backend
        track: stable
    spec:
      containers:
      - name: windows-server-iis
        image: mcr.microsoft.com/windows/servercore:1809
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
        command:
        - powershell.exe
        - -command
        - "Add-WindowsFeature Web-Server; Invoke-WebRequest -UseBasicParsing -Uri 'https://dotnetbinaries.blob.core.windows.net/servicemonitor/2.0.1.6/ServiceMonitor.exe' -OutFile 'C:\\ServiceMonitor.exe'; echo '<html><body><br/><br/><marquee><H1>Hello EKS!!!<H1><marquee></body><html>' > C:\\inetpub\\wwwroot\\default.html; C:\\ServiceMonitor.exe 'w3svc'; "
      nodeSelector:
        kubernetes.io/os: windows
---
apiVersion: v1
kind: Service
metadata:
  name: windows-server-iis-service
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: windows-server-iis
    tier: backend
    track: stable
  sessionAffinity: None
  type: LoadBalancer
EOF

kubectl apply -f windows-server-iis.yaml 
```
- 步骤五：等待3～5分钟复制Kubernetes Service链接，访问Windows应用。
```
kubectl get svc  | grep windows-server-iis-service
```

- 步骤六：删除Windows工作节点组。
```
eksctl delete nodegroup ng-windows --cluster <your-cluster-name>
```

## Lab7、创建Fargate容器
- 步骤一：创建Fargate Profile，fargate profile可以通过命令行的形式创建也可以通过图形界面的方式创建。
```
eksctl create fargateprofile --cluster <your-cluster-name> --name fargateprofile --namespace fargate-test
```
- 步骤二：查询当前计算节点数量，并创建Fargate容器，之后对比节点数量的变化。
```
kubectl get node 
kubectl create ns fargate-test
kubectl create deployment web --image=nginx -n fargate-test
```
- 步骤三：扩展Fargate容器的数量，观察节点数量的变化。
```
kubectl scale deployment web --replicas=10 -n fargate-test
kubectl get pod
kubectl get node 
```



