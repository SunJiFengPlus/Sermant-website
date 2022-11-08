# 后端模块介绍

本文档主要介绍[**Sermant后端模块**](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend)，其主要职能就是将`核心功能`运行过程中产生的消息传递给Kafka，起到将短连接转化为长连接的作用。

[定位]: 

- Sermant数据处理后端模块和前端信息展示模块

[能力]: 
- 接受框架和插件的数据写入kafka
- 提供前端界面展示框架和插件的心跳信息

[使用方式]:
- backend运行环境依赖kafka, 默认kafka的配置如下，若kafka环境配置不同，需要修改[kafka环境配置](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend/src/main/resources/application.properties)
  ```properties
  spring.kafka.bootstrap-servers=127.0.0.1:9092
  ```
- 框架运行依赖backend，backend的默认配置如下，若宿主应用和backend未运行在统一机器，需要修改[backend配置](https://github.com/huaweicloud/Sermant/tree/develop/sermant-agentcore/sermant-agentcore-premain/src/main/resources/config/bootstrap.properties)
    ```properties
    backendIp=127.0.0.1
    backendPort=6888
    ```
- 运行backend模块之后，访问 `http://127.0.0.1:8900`查看框架和插件启动信息
  <MyImage src="/docs-img/backend_sermant_info.png"></MyImage>
- backend默认后端监听端口为8900，若出现端口被占用或需要设置为其他端口，请修改[backend端口](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend/src/main/resources/application.properties)
  ```properties
  server.port=8900
  ```
- backend前端采用vue框架开发，目前前端仅提供简单的展示框架运行状态信息，若前端不满足要求，请修改[backend前端](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend/src/main/webapp)
- backend要求框架和插件产生的数据存入不同的topic，若以下现有的topic配置不满足要求，则需要按照同样的格式添加topic配置，添加topic请修改[topic配置](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend/src/main/resources/application.properties)
  ```properties
  datatype.topic.mapping.0=topic-heartbeat
  datatype.topic.mapping.1=topic-log
  datatype.topic.mapping.2=topic-flowcontrol
  datatype.topic.mapping.3=topic-flowrecord
  datatype.topic.mapping.4=topic-server-monitor
  datatype.topic.mapping.5=topic-open-jdk-jvm-monitor
  datatype.topic.mapping.6=topic-ibm-jvm-monitor
  datatype.topic.mapping.7=topic-agent-registration
  datatype.topic.mapping.8=topic-agent-monitor
  datatype.topic.mapping.9=topic-agent-span-event
  datatype.topic.mapping.10=topic-druid-monitor
  datatype.topic.mapping.11=topic-flowcontrol-metric
  ```
- backend不强依赖kafka服务，配置中默认心跳数据写入缓存中，如需要将数据写入kafka，则更改[backend配置文件](https://github.com/huaweicloud/Sermant/tree/develop/sermant-backend/src/main/resources/application.properties)中以下配置为`false`即可。
  ```properties
  heartbeat.cache=true
  ```

## 相关文档

|文档名称|
|:-|
|[核心模块介绍](agentcore.md)|
|[入口模块介绍](entrance.md)|
