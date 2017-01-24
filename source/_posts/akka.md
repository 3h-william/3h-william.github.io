title: 项目中，使用akka的一些经验和总结
tags: 
- akka
date: 2017-01-24 15:00:11
---
# 背景
Akka 是scala中，应该说不仅仅是scala，整个jvm语言领域，较为优秀的并发框架。 但是，由于国内使用的同僚少之又少，所以相关的资料较少。
大部分的资料和经验来自于 官网， [akka.io](http://akka.io/) ，还有一些奇奇乖乖的问题通过stackoverflow得意解决，最后，是在是解决不了的问题，需要多尝试，debug，等解决。
总之，最终不论是多少坑也好，多少弯路也好，算是把akka用了起来，构建了一个以akka为分布式集群核心的任务调度集群。每天的 job数量在 3000+，系统也是相对稳定，深感欣慰。 


# 一些关键点
  
  1.  Akka的Cluster，路由发现目前测试是有点问题的，路由的配置个人感觉也是有点繁琐，而且中间环节控制的不够好，所以不推荐使用。
  2.  在定义 Actor的时候，因为构建一个 Actor的时候，和初始化对象是不一样的。 所以如果再初始化 actor中发生异常，是无法在代码中捕获，只能通过parent的message来捕获
      可以参考   [question](http://stackoverflow.com/questions/18648390/how-should-an-akka-actor-be-created-that-might-throw-an-exception)
      比较好的方式是，尽量保证在 初始化 Actor的时候， 减少异常发生的可能性(对数据库的操作)等。 
  3.  Actor中对于 Future的使用，取决于并发和串行。 该并发的时候用 Future，不该用的时候，就串行。 因为和 Java的线程模型的抽象程度不一样，所有不要用Java中的线程思维理解Akka中Message和Actor
  4.  在机器负载很大的情况下， Akka的Message发送和接受延迟是回很大的，所以尽可能控制好机器的负载。
  5.  Akka的失败机制。在任何一个 Actor中发生异常，如何处理，默认情况是交给 Parent的supervisor，但是策略也有多种，根据具体使用场景，可以自定义。

 
 ---

作者：3h-william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---