# MongoDB on Kubernetes

## 实验内容

通过业界常用的两种MongoDB Operator（MongoDB Enterprise Operator 以及 Percona Kubernetes Operator）创建并管理MongoDB集群，完成MongoDB集群创建、容量调整、集群使用、数据备份、集群监控等任务，同时并对比两种Operator的优劣势。

## 实验一
### 利用MongoDB Enterprise Operator管理MongoDB集群

### MongoDB Enterprise Operator描述
[MongoDB Enterprise Operator](https://docs.mongodb.com/kubernetes-operator/master/)可用于在Kubernetes平台上管理MongoDB集群的生命周期，包括集群的创建、容量的调整、服务通信、数据备份、快照恢复等任务。MongoDB Enterprise Operator基于标准Kubernetes API开发而成，具有平台无关性，可以安装在各种云厂商以及软件发行商的提供的Kubernetes平台上。其基于标准Kubernetes API开发而成，并提供CRD，开发运维人员仅需要按照MongoDB Enterprise Operator提供的的CRD进行定义并提交任务即可，MongoDB Enterprise Operator会自动地创建Kubernetes Statefulset/PVC/ConfigMap等资源自动地完成MongoDB集群的创建和配置。

### 实验前提
- 完成EKS集群的创建
- 客户端已经完成kubectl工具的安装和配置
- 客户端已经完成helm工具的安装

### 实验步骤
- 步骤一：安装MongoDB Enterprise Operator
  - 1.1 克隆MongoDB Enterprise Operator gihub链接.
  ```
  git clone https://github.com/mongodb/mongodb-enterprise-kubernetes.git
  ```
  - 1.2 创建Kubernetes Namespace，MongoDB Operator与MongoDB集群都将运行在此命名空间下。
  ```
  kubectl create namespace mongodb
  ```
  - 1.3 通过helm工具安装MongoDB Enterprise Operator
  ```
  ## 切换路径
  cd mongodb-enterprise-kubernetes/
  ## 通过helm安装MongoDB Enterprise Operator
  helm install mongodb helm_chart \
     --values helm_chart/values.yaml
  ```
  - 1.4 运行命令查看MongoDB Enterprise Operator安装结果，MongoDB Enterprise Operator会以pod的形式运行在mongodb namespace下。
  ```
   kubectl get pod -n mongodb 
  ```
- 步骤二：部署OpsManager实例
  - 2.1 设置kubectl context信息，让kubectl默认工作在mongodb命名空间下。
  ```
  kubectl config set-context $(kubectl config current-context) --namespace=mongodb
  ```
  - 2.2 创建Kubernetes Secret，新创建的两个Secret用于存放OpsManager用户名和密码，以及OpsManager Applicaton Database密码。当OpsManager成功创建后，用户可用该OpsManager的用户名和密码登录并OpsManager，可用OpsManager Application Database的密码对OpsManager的Databse进行维护。
  ```
  ## 存放OpsManager用户名密码
  kubectl create secret generic om-admin-secret \
  --from-literal=Username=admin \
  --from-literal=Password=Admin@123 \
  --from-literal=FirstName=San\
  --from-literal=LastName=Zhang
  ## 存放OpsManager Applicaton Database
  kubectl create secret generic om-db-user-secret \
  --from-literal=password=Admin@12345
  ```
  - 2.3 通过OpsManager yaml配置文件创建OpsManager实例。若需要进一步定制化配置OpsManager，请查看[详细配置](https://docs.mongodb.com/kubernetes-operator/master/reference/k8s-operator-om-specification/)。
  ```
  kubectl apply -f kubernetes/mongodb-ops-manager.yml
  ```
  - 2.4 查看OpsManager部署状态。
  ```
  kubectl get om mongodb-ops-manager -o yaml
  kubectl get pod
  ```
  - 2.5 查询OpsManager url，访问OpsManager界面。默认情况下，MongoDB Enterprise Operator会自动为OpsManager创建类型为LoadBalancer的Kubernetes Service，用户可通过该Service对应的ELB URL访问OpsManager图形界面。利用2.2步创建的OpsManager用户名和密码登录OpsManager，查看OpsManager支持的功能。
  ```
  kubectl get svc |grep mongodb-ops-manager-svc-ext 
  ```

- 步骤三：部署MongoDB Database
  - 3.1 登录OpsManager创建API KEY，API KEY的作用范围可以是global、organization或者project。通过API KEY，第三方应用程序可以在与OpsManager进行交互，同时也会收到相应的权限控制。
  - 3.2 利用API KEY创建Kubernetes的Secret，在创建MongoDB数据库之前，MongoDB Operator会引用该Secret，向OpsManager注册即将创建的MongoDB数据库集群信息，待MongoDB集群创建成功后，MongoDB集群的信息会自动显示在OpsManager的界面上。
  - 3.3 通过MongoDB yaml文件创建MongoDB集群，MongoDB Enterprise Operator支持以[Standalone](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-standalone/)、[Replica Set](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-replica-set/)、[Sharded Cluster](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-replica-set/)三种模式部署集群，用户可以在MongoDB yaml文件中定义相应的类型。除了集群类型，用户也可以在yaml文件中定义其它与集群相关的信息，如MongoDB版本、集群数量、存储卷大小、启动参数等信息，若需要对集群进行进一步定制化配置，请参考[详细信息](https://docs.mongodb.com/kubernetes-operator/master/reference/k8s-operator-specification/#replica-set-settings)。本步骤会创建一个类型为Replica Set，节点数量为三的MongoDB集群。
  - 3.4 查看MongoDB集群部署的状态

- 步骤四：MongoDB数据备份与快照恢复
  - 4.1 创建S3 Bucket
  - 4.2 获取AWS IAM accessKey与secretKey的信息，并利用该信息创建Kubernetes Secret，MongoDB OpsManager与Backup Agent会通过该Secret值与AWS S3交互，并将MongoDB集群数据备份的快照存放在S3存储桶上。
  - 4.3 启用MongoDB备份功能，备份功能的启用与参数的定义可以在OpsManager yaml文件中进行定义，用户可在该文件中定义MongoDB备份的存储介质以及目标集群等信息，如需更加详细地定义备份任务，请参考详细配置。
  - 4.4 连接MongoDB集群，并在集群上创建数据集
  - 4.5 登录OpsManager，为MongoDB集群创建备份任务、置备份策略并对集群创建快照。
  - 4.6 连接MongoDB集群，并删除MongoDB数据。
  - 4.7 登录OpsManager，恢复MongoDB集群快照。
  - 4.8 连接MongoDB集群，验证MongoDB集群数据是否恢复。
  
  
