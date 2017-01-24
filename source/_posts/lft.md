title: 新一代分布式调度系统--雷峰塔架构
tags: 
- akka
- scala
- 技术产品
- 技术架构
date: 2017-01-24 15:30:11
---
# 背景
  分布式任务调度系统工具很多，有Oozie ,Azkaban, airflow... ,大多数，都只能满足部分的需求。对于一个成熟的企业来说，只能当做工具，
  不能称之为产品。 对于一些特殊的定制化的需求，是无法满足的。 
  因此，我们决定自主开发一个任务调度系统，定义为新一代调度系统，吸收各类调度系统的优点，并加以细化，基于Hadoop，不局限于Hadoop，提供完整的工作模式和工作机制。

# 什么是雷峰塔
雷峰塔是新一代分布式任务调度系统。 可以通过扩展任务类型来丰富调度系统，同时也可以自定义WebUI的部分插件功能。
其中WebUI的部分功能设计是参考了 Azkaban 的设计风格。

# 对比 Azkaban ? 
  
  更灵活的插件框架，满足更深的定制化需求。
  多个工作节点在不同机器上协同调度，可以通过自定义负载均衡来实现任务的分发。
  增加执行用户概念，一个执行器上可以用不同用户执行 Hadoop/Spark任务。
  

# Overview
雷峰塔是一个分布式的任务调度系统。 
其特点是 :
 便于部署
 插件灵活，扩展性感
 Web UI 自定义
 
## 整体架构
雷峰塔架构是基于Master/Worker的架构模式
![](/img/lft/lft_cluster_architecture.png)

### 调度系统集群有三个角色
  1. Master
     Master作为任务协调节点，对外Rest API节点。整个集群只能存在一个。
  2. Worker
     Worker节点作为具体任务执行节点。
  3. Web 
     界面操作入口


### Java + Scala 混合服务

    整个系统混合了 Java + Scala的代码编程
    Java 主要处理对外接口逻辑层
    Scala 主要应用于服务底层
    其中，Akka 作为分布式的基础组件
    插件模块核心用scala编程，也可以用Java/Scala实现自定义插件扩展
    
    
### HDFS + Mysql 
 雷峰塔需要HDFS和Mysql (基本服务，如果有插件使用 Spark等还需要Yarn/Spark集群)的支持
1. Mysql 
作为整个调度系统的 工作流、任务、调度、权限等使用的数据库。 
2. HDFS
作为日志记录、调度资源共享的目录

## 工作流架构
以下是工作流提交的运行流程和机制
![](/img/lft/work_flow_architecture.png)

### Flow /Job / Scheduler /Dependence 说明
1. Flow (工作流)
Flow定义为工作流，由其中字段DAG来 描述一个工作的执行流程，由一组Job组成。
 
2. Job  (任务)
Job定义为任务，描述具体任务是什么。
 
3. Scheduler (调度)
调度定义，描述了调度的定义
 
4. Dependence (资源)
定义了一种资源的表达，某一个任务所需要的一些 jars,配置文件，脚本，都可以放在Dependence 中,
其和任务是"多对一"的关系， 一个Dependence可以被多个Job所使用
 
5. Execution (执行)
Flow/Job/Scheduler 是静态概念， 一旦生成可执行的内容，生成与之对应的 FlowExecution,JobExecution和 SchedulerExecution



## 任务插件框架
![](/img/lft/plugin_architecture.png)

核心部分是 Job Framework ， 包含了 任务的解析，产生，到分发。
Job FrameWork : 开发者可以根据自己的使用场景，定制自己的任务解析器，包括任务的执行规则，流程等。
 
 
## Job Framework
 部分组件说明 :
  1. Job Plugin Conf 和 Job Plugin Jars 
  Job Plugin打包的类，必须在所有的Master/Worker节点都部署相同的内容，如果更新，需要重启整个集群 。
 
  2. Job Serialized 
  Job 描述方式，以Json为明文的存储，JobDefine(Object)为序列化对象生成
  Master根据配置文件、数据库内容、参数等，构建了一个 Job Define，并经过序列化后生成ExecutionRequestContent发送到 Worker，由Worker解析、执行
 
  3. Job Generator 
  Job 生成器 ,在Master端执行，负责生成 Job Define
 
  4. Job Executor
  Job 执行器，在Worker端执行，负责解析 Job Define，并且执行具体的job
  
  
## Load Balance 
  任务分发策略，也支持自定义。
  目前支持的策略为: 
  1. round-robin :        轮训选择一个节点执行
  2. one-manually :       指定任务在某一个节点执行
  
  
 ---

作者：3h-william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---