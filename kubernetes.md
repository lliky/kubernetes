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

<table><tr><td>资源分类</td><td>资源名称</td><td>缩写</td><td>资源作用</td></tr><tr><td>集群级别资源</td><td>nodes</td><td>no</td><td>集群组成部分</td></tr><tr><td></td><td>namespaces</td><td>ns</td><td>隔离 pod</td></tr><tr><td>pod 资源</td><td>pods</td><td>po</td><td>装载容器</td></tr><tr><td>pod 资源控制器</td><td>replicationcontrollers</td><td>rc</td><td>控制 pod 资源</td></tr><tr><td></td><td>replicasets</td><td>rs</td><td>控制 pod 资源</td></tr><tr><td></td><td>deployments</td><td>deploy</td><td>控制 pod 资源</td></tr><tr><td></td><td>daemonsets</td><td>ds</td><td>控制 pod 资源</td></tr><tr><td></td><td>jobs</td><td></td><td>控制 pod 资源</td></tr><tr><td></td><td>cronjobs</td><td>cj</td><td>控制 pod 资源</td></tr><tr><td></td><td>horizontalpodautoscalers</td><td>hpa</td><td>控制 pod 资源</td></tr><tr><td></td><td>statefulsets</td><td>sts</td><td>控制 pod 资源</td></tr></table>