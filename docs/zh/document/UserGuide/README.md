# Sermant 开发和使用介绍

Sermant 是基于Java Agent的字节码增强技术，通过 Java Agent 对宿主应用进行非侵入式增强，以解决Java应用的微服务治理问题。Sermant的初衷是建立一个面向微服务治理的对开发态无侵入的解决方案生态，降低服务治理开发和使用的难度，通过抽象接口、功能整合、插件隔离等手段，达到简化开发、功能即插即用的效果。

本文档对如何开发和使用 Sermant 做详细介绍。

## 运行环境

- [HuaweiJDK 1.8](https://gitee.com/openeuler/bishengjdk-8) / [OpenJDK 1.8](https://github.com/openjdk/jdk) / [OracleJDK 1.8](https://www.oracle.com/java/technologies/downloads/)
- [Apache Maven 3](https://maven.apache.org/download.cgi)

## 模块说明

**Sermant**包含以下模块：

- [sermant-agentcore](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore): *Java Agent*相关内容
    - [sermant-agentcore-core](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore/sermant-agentcore-core): 核心框架模块
    - [sermant-agentcore-premain](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore/sermant-agentcore-premain): *Java Agent*入口模块
    - [sermant-agentcore-implement](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore/sermant-agentcore-implement): 核心功能实现模块
    - [sermant-agentcore-config](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore/sermant-agentcore-config): 配置模块
- [sermant-backend](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend): 消息发送模块服务端
- [sermant-package](https://github.com/huaweicloud/Sermant/tree/develop/sermant-package): 打包模块
- [sermant-plugins](https://github.com/huaweicloud/Sermant/tree/develop/sermant-plugins): 插件根模块，内含各种功能的插件及相关附加件
- [sermant-injector](https://github.com/huaweicloud/Sermant/tree/develop/sermant-injector): sermant-agent容器化部署Admission Webhook组件

## 打包

**Sermant**项目中包含以下几种profile，对应不同使用场景：

- *agent*: 编译、打包核心功能和发布的稳定版本插件。
- *package*: 将打包结果归档为产品包
- *release*: 发布构建产物到中央仓库
- *test*: 编译打包所有项目模块

执行以下*maven*命令，对**Sermant**工程使用*agent*进行默认打包：

```shell
mvn clean package -Dmaven.test.skip
```

命令执行完毕后，工程目录下将生成一个形如`sermant-agent-x.x.x`的文件夹和形如`sermant-agent-x.x.x.tar.gz`的压缩文件，后者为**Sermant**的产品包，前者则为产品包解压后的内容。

## 产品目录说明

- *agent*: Java Agent相关内容
    - *config*: 配置文件目录
        - *bootstrap.properties*: 启动配置
        - *config.properties*: 核心功能配置
        - *plugins.yaml*: 插件配置，配置着需要被加载的插件功能
    - *core/sermant-agentcore-core-x.x.x.jar*: **Sermant**的核心框架包
    - implement/sermant-agentcore-implement-x.x.x.jar: **Sermant**核心功能实现包
    - *pluginPackage*: 插件包目录，插件按功能名称分类
        - *xxx*: 任意插件功能
            - *config/config.yaml*: 插件配置文件
            - *plugin*: 插件包目录
            - *service*: 插件服务包目录
    - *sermant-agent.jar*: JavaAgent入口包
- *server*: 服务器目录，含**Sermant**的服务端，插件的服务端和客户端

## 容器化部署说明
k8s环境下，Sermant支持通过sermant-injector组件实现宿主应用自动挂载sermant-agent包的快速部署方式。如何部署sermant-injector与宿主应用可以参考[容器化部署指导手册](https://github.com/huaweicloud/Sermant/tree/develop/docs/user-guide/injector-zh.md)

## 插件开发

如何新增一个插件可以参考[插件模块开发手册](https://github.com/huaweicloud/Sermant/tree/develop/docs/dev-guide/dev_plugin_module-zh.md)，其中涉及添加插件、插件服务及附加件的详细流程。

如何编写一个插件的内容可以参考[插件代码开发手册](https://github.com/huaweicloud/Sermant/tree/develop/docs/dev-guide/dev_plugin_code-zh.md)，其中涉及大部分开发插件过程中可能遇到场景。