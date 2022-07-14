# 第三章 资源管理

本章节主要介绍 yaml 语法和 Kubernetes 的资源管理方式

## 3.1 资源管理介绍

在 kubernetes 中，所有的内容都抽象为资源，用户需要通过操作资源来管理 kubernetes。

> kubernetes 的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在 kubernetes 集群中运行一个个的容器，并将指定的程序跑在容器中。
>
> kunernetes 的最小管理单元就是 pod 而不是容器，所有只能将容器放在 **pod** 中，而 kubernetes 一般也不会直接管理 pod，而是通过 **pod 控制器**来管理 pod 的。
>
> pod 可以提供服务之后，就要考虑如何访问 pod 中服务， kubernetes 提供了 **service** 资源实现这个功能。
>
> 当然，如果 pod 中程序的数据需要持久化，kubernetes 还提供了各种 **存储** 系统

![](./chapter3/chapter3-1.png)

> 学习 kubernetes 的核心，就是学习如何对集群上的 **pod 、pod 控制器、service、存储** 等各种资源进行操作。



## 3.2 YAML 语言介绍

​	YAML 是一个类似 XML、JSON 的标记性语言。它强调以数据为中心，并不是以标识语言为重点。因而 YAML 本身的定义比较简单，号称“一种人性化的数据格式语言”。

YAML 的语法比较简单，主要有下面几个：

* 大小写敏感
* 使用缩进表示层级关系
* 缩进不允许使用 tab，只允许空格
* 缩进的空格数不重要，只要相同层级的元素左对齐即可
* ‘#’ 表示注释

YAML 支持一下几种数据类型：

* 纯量：单个的、不可再分的值
* 对象：键值对的集合，又称为映射（mapping）/ 哈希（hash）/ 字典（dictionary）
* 数组：一组按次序排列的值，又称为序列（sequence）/ 列表（list）

```yaml
# 纯量，就是指的一个简单的值，字符串、布尔值、整数、浮点数、Null、时间、日期
# 1 布尔值
c1: true (或者 True)
# 2 整型
c2: 234
# 3 浮点数
c3: 3.14
# 4 null 类型
c4: ~ # 使用 ～ 表示 null
# 5 日期类型
c5: 2022-07-06 # 日期必须使用 ISO 8601 格式，即 yyyy-MM-dd
# 6 时间类型
c6: 2022-07-06T16:06:06+08:00 # 时间使用 ISO 8601 格式，时间和日期之间使用 T 连接，最后使用 + 代表时区
# 7 字符串类型
c7: haha
c8: line1
	  line2  # 字符串过多的情况可以拆成多行，每一行会被转化成一个空格
```

```yaml
# 对象
# 形式一（推荐）
xiaoming:
  age: 15
  address: guangzhou
# 形式二（了解）
xiaoming: {age: 15,address: guangzhou}
```

```yaml
# 数组
# 形式一（推荐）
address:
  - 天河
  - 黄埔
# 形式二（了解）
address: [天河,黄埔]
```

> 小提示：
>
> 1 书写 yaml 切记 : 后面要加一个空格
>
> 2 如果需要将多段 yaml 配置放在一个文件中，中间要使用 --- 分隔
>
> 3 下面是一个 yaml 转 json 的网站：https://www.json2yaml.com/convert-yaml-to-json



## 3.3 资源管理方式

* 命令式对象管理：直接使用命令去操作 kubernetes 资源

  ```shell
  kubectl run nginx-pod --image=nginx:1.17.1 --port=80
  ```

* 命令式对象配置：通过命令配置和配置文件去操作 kubernetes 资源

  ```
  kubectl create/patch -f nginx-pod.yaml
  ```

* 声明式对象配置：通过 apply 命令和配置文件去操作 kubernetes 资源

  ```
  kubectl apply -f nginx-pod.yaml  // 创建 更新
  ```

|      类型      | 操作对象 | 适用环境 |      优点      |               缺点               |
| :------------: | :------: | :------: | :------------: | :------------------------------: |
| 命令式对象管理 |   对象   |   测试   |      简单      | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 |   文件   |   开发   | 可以审计、跟踪 |  项目大时，配置文件多，操作麻烦  |
| 声明式对象配置 |   目录   |   开发   |  支持目录操作  |        意外情况下难以调试        |

### 3.3.1 命令式对象管理

#### kubectl 命令

 	kubectl 是 Kubernetes 集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl 命令的语法如下：

```md
kubectl [command] [type] [name] [flags]
```

**command**： 指定要对资源执行的操作，例如 create、get、delete

**type**：指定资源类型，比如 deployment、pod、service

**name**：指定资源的名称，名称大小写敏感

**flags**：指定额外的可选参数

```powershell
# 查看所有 pod
kubectl get pod

# 查看某个 pod
kubectl get pod pod_name

# 查看某个 pod,以 yaml 格式展示结果
kubectl get pod pod_name -o yaml
```

#### 资源类型

kubernetes 中所有的内容都抽象为资源，可以通过下面的命令进行查看：

```powershell
kubectl api-resources
```

经常使用的资源有下面这些：

<table>
   <tr>
      <td>资源分类</td>
      <td>资源名称</td>
      <td>缩写</td>
      <td>资源作用</td>
   </tr>
   <tr>
      <td rowspan="2">集群级别资源</td>
      <td>nodes</td>
      <td>no</td>
      <td>集群组成部分</td>
   </tr>
   <tr>
      <td>namespaces</td>
      <td>ns</td>
      <td>隔离 pod</td>
   </tr>
   <tr>
      <td>pod 资源</td>
      <td>pods</td>
      <td>po</td>
      <td>装载容器</td>
   </tr>
   <tr>
      <td rowspan="8">pod 资源控制器</td>
      <td>replicationcontrollers</td>
      <td>rc</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>replicasets</td>
      <td>rs</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>deployments</td>
      <td>deploy</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>daemonsets</td>
      <td>ds</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>jobs</td>
      <td></td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>cronjobs</td>
      <td>cj</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>horizontalpodautoscalers</td>
      <td>hpa</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td>statefulsets</td>
      <td>sts</td>
      <td>控制 pod 资源</td>
   </tr>
   <tr>
      <td rowspan="2">服务发现资源</td>
      <td>services</td>
      <td>svc</td>
      <td>统一 pod 对外接口</td>
   </tr>
   <tr>
      <td>ingress</td>
      <td>ing</td>
      <td>统一 pod 对外接口</td>
   </tr>
   <tr>
      <td rowspan="3">存储资源</td>
      <td>volumeattachments</td>
      <td></td>
      <td>存储</td>
   </tr>
   <tr>
      <td>persistentvolumes</td>
      <td>pv</td>
      <td>存储</td>
   </tr>
   <tr>
      <td>persistentvolumeclaims</td>
      <td>pvc</td>
      <td>存储</td>
   </tr>
   <tr>
      <td rowspan="2">配置资源</td>
      <td>configmaps</td>
      <td>cm</td>
      <td>配置</td>
   </tr>
   <tr>
      <td>secrets</td>
      <td></td>
      <td>配置</td>
   </tr>
</table>

#### 操作

Kubernetes 允许对资源进行多种操作，可以通过 --help 查看详细的操作命令

```powershell
kubectl --help
```

经常使用的操作有下面这些：

<table>
   <tr>
      <td>命令分类</td>
      <td>命令</td>
      <td>翻译</td>
      <td>命令作用</td>
   </tr>
   <tr>
      <td rowspan="6">基本命令</td>
      <td>create</td>
      <td>创建</td>
      <td>创建一个资源</td>
   </tr>
   <tr>
      <td>edit</td>
      <td>编辑</td>
      <td>编辑一个资源</td>
   </tr>
   <tr>
      <td>get</td>
      <td>获取</td>
      <td>获取一个资源</td>
   </tr>
   <tr>
      <td>patch</td>
      <td>更新</td>
      <td>更新一个资源</td>
   </tr>
   <tr>
      <td>delete</td>
      <td>删除</td>
      <td>删除一个资源</td>
   </tr>
   <tr>
      <td>explain</td>
      <td>解释</td>
      <td>解释资源文档</td>
   </tr>
  <tr>
      <td rowspan="10">运行与调试</td>
      <td>run</td>
      <td>运行</td>
      <td>在集群中运行一个指定的镜像</td>
   </tr>
   <tr>
      <td>expose</td>
      <td>暴露</td>
      <td>暴露资源为 service</td>
   </tr>
   <tr>
      <td>describe</td>
      <td>描述</td>
      <td>显示资源内部信息</td>
   </tr>
   <tr>
      <td>logs</td>
      <td>日志</td>
      <td>输出容器在 pod 中的日志</td>
   </tr>
   <tr>
      <td>attach</td>
      <td>缠绕</td>
      <td>进去运行中的容器</td>
   </tr>
   <tr>
      <td>exec</td>
      <td>执行</td>
      <td>执行容器中的一个命令</td>
   </tr>
   <tr>
      <td>cp</td>
      <td>复制</td>
      <td>在 pod 内外复制文件</td>
   </tr>
   <tr>
      <td>rollout</td>
      <td>首次展示</td>
      <td>管理资源的发布</td>
   </tr>
   <tr>
      <td>scale</td>
      <td>规模</td>
      <td>扩（缩）容 pod 的数量</td>
   </tr>
   <tr>
      <td>autoscale</td>
      <td>自动调整</td>
      <td>自动调整 pod 的数量</td>
   </tr>
  <tr>
      <td rowspan="2">高级命令</td>
      <td>apply</td>
      <td>rc</td>
      <td>通过文件对资源进行配置</td>
   </tr>
   <tr>
      <td>label</td>
      <td>标签</td>
      <td>更新资源上的标签</td>
   </tr>
   <tr>
      <td rowspan="2">其他命令</td>
      <td>cluster-info</td>
      <td>集群信息</td>
      <td>显示集群信息</td>
   </tr>
   <tr>
      <td>version</td>
      <td>版本</td>
      <td>显示当前server和 client 的版本</td>
   </tr>
</table>



### 3.3.2 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作 kubernetes 资源

1）创建一个 nginxpod.yaml ，内容如下：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  
---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:1.17.1
```

2）执行 create 命令，创建资源：

```powershell
kubectl create -f nginxpod.yaml
```

3）执行 get 命令，查看资源：

```
kubectl get -f nginxpod.yaml
```

4）执行 delete 命令，删除资源：

```
kubectl delete -f nginxpod.yaml
```

```md
总结：
		命令式对象配置的方式操作，可以简单的认为： 命令 + yaml 配置文件
```



### 3.3.3 声明式对象配置

声明式对象配置跟命令式对象配置很相似，但是它只有一个命令 apply。

```powershell
# 首次执行一次， 创建了资源
kubectl apply -f nginxpod.yaml

# 再次执行 说资源没动过
namespace/dev unchanged
pod/nginxpod unchanged
```

```
总结：
		声明式对象配置就是使用 apply 描述一个资源最终的状态（ yaml 中定义状态）
		使用 apply 操作资源：
				如果资源不存在，就创建，相当于 kubectl create
				如果资源已存在，就更新，相当于 kubectl pathch
```

> 扩展： kubectl 可以在 node 节点上运行吗？
>
> kubectl 的运行需要配置，它的配置文件在 $HOME/.kube，如果想要在节点 node 运行此命令，需要将 master 的 .kube 文件复制到 node 节点上

```
scp -r HOME/.kube node1:HOME/
```

> 使用推荐：三种方式应该怎么用？

创建/更新资源   使用声明式对象配置 kubectl apply -f yaml

删除资源  使用命令式对象配置  kubectl delete -f yaml

查询资源 使用命令式对象管理 kubectl get/describe  name



# 第四章 实战入门

 ## 4.1 Namespace

​	Namespace 是 kubernetes 系统中的一种非常重要的资源，它的主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**

​	默认情况下，kubernetes 集群中的所有的 pod 都是可以相互访问的。但是在实际中，可能不想让两个 pod 之间进行互相的访问，那此时就可以将两个 pod 划分到不同的 namespace 下。kubernetes 通过将集群内部的资源分配到不同的 namespace 中，可以形成逻辑上的“组”，以方便不同的组的资源进行隔离使用和管理。

​	可以通过 kubernetes 的授权机制，将不同的 kubernetes 交给不同租户进行管理，这样就实现了多租户的资源隔离。此时还能结合 kubernetes 的资源配额机制，限定不同租户能占用的资源，例如 cpu 使用量、内存使用量等等，来实现租户可用资源的管理。

![chapter04-1](./images/chapter04/chapter04-1.jpeg)

Kubernetes 在集群启动之后，会默认创建几个 namespace

 ```powershell
 # kubectl get ns
 NAME                   STATUS   AGE
 default                Active   60d   # 所有未指定 namespace 的对象都会被分配在 default 命名空间
 kube-node-lease        Active   60d   # 集群节点之间的心跳维护， v1.13 开始引入
 kube-public            Active   60d   # 此命名空间下的资源可以被所有人访问（包括未认证用户）
 kube-system            Active   60d   # 所有由 kubernetes 系统创建的资源都处于这个命名空间
 ```

下面来看 namespace 资源的具体操作

**查看**

```powershell
# 1 查看所有 ns 
kubectl get ns
# 2 查看指定的 ns
kubectl get ns name
# 3 指定输出格式  kubetctl get ns ns名称 -o 格式参数
# kubenetes 支持的格式有很多，比如 wide、json、yaml
kubectl get ns name -o yaml  

# 4 查看 ns 详情 kubectl describe ns ns名称
kubectl describe ns default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active   ## Active 命名空间正在使用。Terminating 正在删除命名空间
```

**创建**

```powershell
# 创建 namespace
kubectl create ns dev
```

**删除**

```powershell
# 删除 namespace
kubectl delete ns dev
```

**配置方式**

首先准备一个 yaml 文件： ns-dev.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f ns-dev.yaml

​	删除：kubectl delete -f ns-dev.yaml



## 4.2 Pod

Pod 是 kuberbetes 集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于 pod 中，pod 可以认为是容器的封装，一个 Pod 可以存在一个或者多个容器。

// image

Kubernetes 在集群启动之后，集群中的各个组件也都是以 pod 方式运行。可通过下面命令查看：

```powershell
kubectl get pod -n kube-system
```

**创建并运行**

Kubenetes 没有提供单独运行 pod 的命令，都是通过 pod 控制器来实现的

```powershell
# 命令格式：kubectl run (pod 控制器名称) 【参数】
# --image 指定 pod 的镜像
# --port 指定端口
# --namespace 指定namespace
kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace=dev
```

**查看 pod 信息**

```powershell
# 查看 pod 基本信息
kubectl get pod -n dev
# 查看 pod 的详细信息
kubectl describe pod pod-name -n dev
```

**访问 pod**

```powershell
# 获取 pod IP
kubectl get pod -n dev -o wide
# 访问 pod
curl IP:port
```

**删除指定 pod**

```powershell
# 删除指定 pod
kubectl delete pod pod-name -n dev

# 此时，显示删除 pod 成功，但是再查询，发现又新产生一个

# 这是因为当前 pod 由 pod 控制器创建的，控制器会监控 pod 状况，一旦发现 pod 死亡，会立即重建
# 此时要想删除 pod，必须删除 pod 控制器

# 查看当前 namespace 下的 pod 控制器
kubectl get deploy -n dev

# 删除 pod 控制器
kubectl delete deploy nginx -n dev

# 查询 pod，发现 pod 被删除了
```

**配置操作**

创建一个 pod-nginx.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: nginx
	namespace: dev
spec:
	containers:
	-	image: nginx:1.17.1
		imagePullPolicy: IfNotPresent
		name: pod
		ports:
		-	name: nginx-port
			containerPort: 80
			protocol: TCP
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f pod-nginx.yaml

​	删除：kubectl delete -f pod-nginxyaml



## 4.3 Label

Label 是 kubernetes 系统中的一个重要概念。它的作用就是在资源上添加标识，用来对他们进行区分和选择。

Label 的特点：

* 一个 Label 会以 key/value 键值对的形式附加到各种对象上，如 Node、Pod、Service 等等
* 一个资源对象可以定义任意数量的 label ，同一个 Label 也可以被添加到任意数量的资源对象上去
* Label 通常在资源对象定义时确定，当然也可以在对象创建后动态添加或删除

可以通过 Label 实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作。

> 一些常用的 Label 示例如下：
>
> * 版本标签："version":"release", "version":"stable"...
> * 环境标签："environment":"dev", "environment":"test"
> * 架构标签："tier":"frontend", "tier":"backend"

标签定义完毕之后，还要考虑到标签的选择，这就要使用到 Label Selector，即：

​	Label 用于给某个资源对象定义标识

​	Label Selector 用于查询和筛选拥有某些标签的资源对象

当前有两种 Label Selector：

* 基于等式的 Label Selector

  name=salve:选择所有包含 Label 中 key="name" 且 value="salve" 的对象

  Env !=production:选择所有包括 Label 中 key="name" 且 value 不等于 "production" 的对象

* 基于集合的 Label Selector

  name in (master, salve):选择所有包含 Label 中 key="name" 且 value="salve" 或 “master” 的对象

  Name not in (frontend):  选择所有包含 Label 中的 key="name" 且 value 不等于 "frontend" 的对象

标签的选择可以使用多个，此时将多个 Label Selector 进行组合，使用逗号 "," 进行分隔即可。例如：

​	   name=salve, env != production

​		name not in (frontend), env != production



**命令方式**

```powershell
# 为 pod 资源打标签
kubectl label pod pod-name version=1.0 -n dev
# 为 pod 资源更新标签
kubectl label pod pod-name version=2.0 -n dev --overwrite
# 查看标签
kubectl get pod pod-name -n dev --show-labels
# 筛选标签
kubectl get pod pod-name -n dev -l version=2.0 --show-labels
#删除标签
kubectl label  pod pod-name -n dev tire-  // 标签 -
```

**配置方式**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    version: "3.0"
    env: "test"
spec:
  containers:
  - image: nginx:1.17.1
    imagePullPolicy: IfNotPresent
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```



## 4.4 Deployment

​	在 kubernetes 中，Pod 是最小的控制单元，但是 kubernetes 很少直接控制 pod ，一般都是通过 pod 控制器来完成的。Pod 控制器用于 Pod 的管理，确保 pod 资源符合预期的状态，当 pod 的资源出现故障时，会尝试进行重启或重建 pod 。

​	在 kubernetes 控制器的种类很多，这里介绍： deployment。

// todo  image

**命令操作**

```powershell
# 命令格式： kubectl run deployment名称 [参数]
# --image 指定 pod 的镜像
# --port 指定端口
# --replicas 指定创建 pod 数量
# --namespace 指定 namespace

kubectl run nginx --image=nginx:1.17.1 --port=80 --replicas=3 -n dev

# 查看创建的 pod 
kubectl get pod -n dev
# 查看deployment 的信息
kubectl get deploy deploy-name -n dev

# UP-TO-DATE：成功升级的副本数量
# AVAILABLE: 可用副本的数量
kubectl get deploy -n dev -o wide

# 查看 deployment 详细信息
kubectl describe deploy deploy-name -n dev
# 删除 deployment
kubectl delete deploy name -n dev
```

 **配置操作**

创建 deploy-nginx.yaml，内容如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        imagePullPolicy: IfNotPresent
        name: pod
        ports:
        - name: nginx-port
          containerPort: 80
          protocol: TCP
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f deploy-nginx.yaml

​	删除：kubectl delete -f deploy-nginx.yaml



## 4.5 Service

能利用 Deployment 来创建一组 pod 来提供具有高可用性的服务。虽然每个 pod 都会分配一个单独的 pod IP，然而却存在如下两问题：

* Pod IP 都随着 pod 的重建产生变化
* Pod IP 仅仅是集群内可见的虚拟 IP，外部无法访问

这样对于访问这个服务带来了难度。因此， kubernetes 设计了 Serveice 来解决这个问题。

Service 可以看作是一组同类 Pod **对外的访问接口**。借助 Service 应用可方便地实现服务发现和负载均衡。

//todo  image

**操作一：创建集群内部可访问的 service**

```powershell
# 暴露 Service
kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev

# 查看 Service
kubectl get svc svc-nginx -n dev -o wide

# 这里产生了一个 ClusterIP ，这就是 Service 的 IP，在 Service 的生命周期中，这个地址是不会变动的
# 可以通过这个 IP 访问当前 service 对应的 Pod
```



**操作二：创建集群外部也可访问的 Service**

```powershell
# 上面创建的 Service 的 type 类型为 ClusterIP，这个 IP 地址只能集群内部访问
# 如果需要创建外部也可以访问的 Service ，需要修改 type 为 NodePort
kubectl expose deploy nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev

# 此时查看，会发现出现了 NodePort 类型的 Service，而且有一对 Port 
kubectl get svc -n dev -o wide

# 就可以通过集群外的主机访问节点 IP:port 
```



**删除 Service**

```
kubectl delete svc svc-name -n dev
```



**配置方式**

创建一个 svc-nginx.yaml，内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  namespace: dev
spec:
  ClusterIP: 172.0.2.12
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP
```

然后就可以执行对应的创建和删除命令了：

​	创建：kubectl create -f svc-nginx.yaml

​	删除：kubectl delete -f svc-nginx.yaml



# 第五章 Pod 详解

## 5.1 Pod 介绍

### 5.1.1 Pod 结构

// todo image

每个 Pod 中都可以包含一个或者多个容器，这些容器可以分为两类：

* 用户程序所在的容器，数量可多可少

* Pause 容器，这是每个 Pod 都会有的一个**根容器**，它的作用有两个：

  * 可以以它为依据，平贵整个 Pod 的 健康状态

  * 可以在根容器上设置 IP 地址，其他容器都是此 IP，以实现 Pod 内部的网路通信

    ```
    这里是 Pod 内部通讯，Pod 的之间通讯采用虚拟二层网络技术实现
    ```

### 5.1.2 Pod 定义

下面是 Pod 的资源清单

```yaml
apiVerson: v1    #必选，版本号，例如 v1
kind: Pod  #必选，资源类型，例如 Pod
metadate:  # 必选，元数据
  name: string # 必选，Pod 名称
  namespace: # Pod 所属的命名空间，默认为 "default"
  labels:   # 自定义标签
    - name: string
spec:  # 必选，Pod 中容器的详细定义
  containers: # 必选， Pod 容器列表
  - name: # 必选，容器名称
    image: # 必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ] # 获取镜像的策略
    command: [string] # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string] # 容器的启动命令参数列表
    workingDir: string # 容器的工作目录
    volumeMounts: # 挂载到容器内部的存储卷位置
    - name: string # 引用 Pod 定义的共享存储卷的名称，需要 volume[] 部分定义的卷名
      mountPath: string # 存储卷在容器内 mount 的绝对路径，应少于 512  字符
      readOnly: boolean # 是否为只读模式
    port: # 需要暴露的端口库号列表
    - name: string # 端口的名称
      containerPort: int # 容器需要监听的端口号
      hostPort: int # 容器所在主机需要监听的端口号，默认与 Container 相同
      protocol: string # 端口协议，支持 TCP 和 UDP，默认 TCP
    env:  # 容器运行前需设置的环境变量列表
    - name: string # 环境变量名称
      value: string # 环境变量的值
    resources: # 资源限制和请求的设置
      limits: # 资源限制的值
  
```

```powershell
# 小提示：
# 可以通过一个命令来查看每种资源的可配置项
# kubectl explain 资源类型
# kubectl explain 资源类型.属性
kubectl explain pod
kubectl explain pod.metadata
```

在 kubernetes 中所有资源的一级属性都是一样的，主要包含 5 部分：

* apiVersion <string>  版本，由 kubenetes 内部定义，版本号必须可以用 kubetctl api-versions 查询到
* kind  <string>            类型，由 kubenetes 内部定义，版本号必须可以用 kubetctl api-resources 查询到
* metadata <Object>   元数据，主要是资源标识和说明，常用的有 name、namespace、labels 等
* spec <Object>            描述，这里配置中最重要的一部分，里面是对各种资源配置的详细描述
* status <Object>         状态信息，里面的内容不需要定义，由 kubernetes 自动生成

在上面的属性中，spec 是接下来研究的重点，看常见的子属性：

* containers <[]Object>  容器列表，用于定义容器的详细信息
* nodeName <string> 根据 nodeName 的值将 pod 调度到指定的 Node 节点上
* nodeSelector <map[]> 根据 nodeSelector 中定义的信息选择将该 pod 调度到这些 label 的 Node 上
* hostNetwork  <boolean> 是否使用主机网络模式，默认为 false，如果设置为 true，表示使用宿主机网络
* volumes <[]object> 存储卷，用于定义 pod 上面挂在的存储信息
* restartPolicy <string> 重启策略，表示 pod 在遇到故障的时候的处理策略

## 5.2 Pod 配置

