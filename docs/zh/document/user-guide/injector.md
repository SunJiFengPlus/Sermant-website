# Sermant-injector使用手册

sermant-injector是基于Kubernetes准入控制器（Admission Controllers）特性开发而来。准入控制器位于k8s API Server中，能够拦截对API Server的请求，完成身份验证、授权、变更等操作。本文介绍在k8s环境下，如何通过sermant-injector组件来实现宿主应用自动挂载sermant-agent包的快速部署。

sermant-injector属于变更准入控制器(MutatingAdmissionWebhook), 能够在创建容器资源前对请求进行拦截和修改。sermant-injector部署在k8s后，只需在宿主应用部署的YAML文件中`spec > template > metadata> labels`层级加入`sermant-injection: enabled`即可实现自动挂载sermant-agent。

## 参数配置

**公共环境变量配置：**

sermant-injector支持为宿主应用所在pod配置自定义的环境变量，方法为在`sermant-injector/deployment/release/injector/values.yaml`中修改env的内容，修改方式如下(kv形式)：

```yaml
env:
  TEST_ENV1: abc
  TEST_ENV2: 123456
```

例如，在Sermant使用过程中，某些配置为当前k8s集群下各pod共享的公共配置，例如**Backend**后端的ip和端口等。则可在此处配置：

```yaml
env:
  backend.nettyIp: 127.0.0.1
  backend.nettyPort: 8900
```

即可使所有pod挂载的Sermant都与该**Backend**后端连接。

sermant-injector部署时的相关参数修改请参考启动和结果验证一节内容。

## 支持版本

sermant-injector当前支持在Kubernetes 1.15及以上版本进行部署，通过Helm v3版本来进行Kubernetes包管理。

- [Kubernetes 1.15+](https://kubernetes.io/)

- [Helm v3](https://helm.sh/)

## 启动和结果验证

在部署sermant-injector前需要先构建sermant-agent镜像以及sermant-injector镜像。

### 构建sermant-agent镜像

#### 准备sermant-agent包

点击 [here](https://github.com/huaweicloud/Sermant/releases)下载release包，也可以在项目中自行打包。

#### 制作镜像

修改文件夹 `sermant-injector/images/sermant-agent`下`build-sermant-image.sh` 脚本中`sermantVersion`,`imageName`和`imageVerison`的值：

> 1. `sermantVersion`为release包的版本
>
> 2. `imageName`为构建的sermant-agent镜像名称
>
> 3. `imageVerison`为构建的sermant-agent镜像版本

在k8s节点下，将`build-sermant-image.sh`和`Sermant.Dockerfile`置于release包`sermant-agent-xxx.tar.gz`同一目录下，执行`build-sermant-image.sh`脚本，完成sermant-agent镜像制作。

```shell
sh build-sermant-image.sh
```

### 构建sermant-injector镜像

#### 准备sermant-injector包

在sermant-injector项目下执行`mvn clean package`命令，在项目目录下生成`sermant-injector.jar`文件

#### 制作镜像

修改文件夹 `sermant-injector/images/injector`下`build-injector-image.sh` 脚本中`imageName`和`imageVerison`的值：

> 1. `imageName`为构建的sermant-injector镜像名称
> 2. `imageVerison`为构建的sermant-injector镜像版本

在k8s节点下，将`build-injector-image.sh`、`start.sh`和`Injector.Dockerfile`置于sermant-injector包`sermant-injector.jar`同一目录下，执行`build-injector-image.sh`脚本，完成sermant-injector镜像制作。

```shell
sh build-injector-image.sh
```

### 部署sermant-injector实例

在宿主应用容器化部署前，需要先部署sermant-injector实例。本项目采用Helm进行Kubernetes包管理。

使用`sermant-injector/deployment/release`下的`injector`Chart模版。

按实际环境修改`values.yaml`中的模版变量：

> `agent.image.addr`和`injector.image.addr`变量与构建镜像时的镜像地址保持一致

上述配置修改完成后，执行`helm install`命令在k8s中部署sermant-injector实例:

```shell
helm install sermant-injector ../injector
```

检查sermant-injector部署pod状态为running。

至此，宿主应用部署前的环境配置工作完成。

### 部署宿主应用

在完成上述sermant-injector部署后，用户根据实际应用编写yaml部署K8s Deployment资源，只需在`spec > template > metadata> labels`层级加入`sermant-injection: enabled`即可实现自动挂载sermant-agent。(如后续不希望挂载，删除后重新启动应用即可)

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: demo-test
  labels:
    app: demo-test
spec:
  replicas: 1
  selector:
    app: demo-test
    matchLabels:
      app: demo-test
  template:
    metadata:
      labels:
        app: demo-test
        sermant-injection: enabled
    spec:
      containers:
      - name: image
        # 请替换成您的应用镜像
        image: image:1.0.0
        ports: 
        - containerPort: 8080
  ports:
    - port: 443
      targetPort: 8443
```

若pod无法创建，请检查sermant-injector是否正确部署以及sermant-agent镜像是否正确构建。

### 验证

pod创建成功后，执行如下命令，其中`${pod_name}`为宿主应用的pod名称

```shell
kubectl get po/${pod_name} -o yaml
```

1. 查看上述命令输出内容`spec > containers > - env`下是否包含环境变量：name为`JAVA_TOOL_OPTIONS`，value为 `-javaagent:/home/sermant-agent/agent/sermant-agent.jar=appName=default`。

2. 查看上述命令输出内容`spec > containers > initContainers > image` 的值是否为构建sermant-agent镜像时的镜像地址。

执行如下命令，其中`${pod_name}`为用户应用的pod名称，`${namespace}`为用户部署应用的namespace名称

```shell
kubectl logs ${pod_name} -n ${namespace}
```

3. 查看上述命令输出内容pod日志开头部分是否包含：

```
[INFO] Loading sermant agent...
```

如果上述信息无误，则表明sermant-agent已成功挂载至用户应用中。