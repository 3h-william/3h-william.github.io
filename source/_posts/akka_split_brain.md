title: Akka集群对于脑裂的策略
tags: 
- akka
- 分布式系统
date: 2017-02-25 17:35:00
---
# 背景
一直以为，Akka的文档，是开源产品中写得较为优秀的，不但全面，而且有一定的深度。 
在使用Akka Cluster的过程中，有几次关于脑裂的问题的发生，下面，就简单的介绍这篇文档:
[Split Brain Resolver](http://doc.akka.io/docs/akka/rp-16s01p01/scala/split-brain-resolver.html)

# 什么是脑裂

```
A fundamental problem in distributed systems is that network partitions (split brain scenarios) and machine crashes are indistinguishable for the observer, 
i.e. a node can observe that there is a problem with another node, but it cannot tell if it has crashed and will never be available again or 
if there is a network issue that might or might not heal again after a while. Temporary and permanent failures are indistinguishable because decisions must be made in finite time, 
and there always exists a temporary failure that lasts longer than the time limit for the decision.
```
  简单的解释一下  
  脑裂常常发身在网络故障的时候，把一个分布式的系统，分成了至少2个partitions。 并且这两个partitions不能识别出到底谁才是正常的，因为集群需要再一个有限的短时间内做出一个判断，而网络的故障的时间是不定。
  最终造成了整个集群处于一种不可用的状态。
  
 # 策略
  所有策略的前提是，集群处于一个稳定的(stable)状态。 也就是说，在这个时间点，已经发生了脑裂，但是，不会有反复 (back and force )节点的 up/down。
  基于这个前提，akka提供了一些策略，来应付脑裂的情况
  
 ## 1 Static Quorum
  配置文件中，固定一个数值(static quorum) 当集群发生脑裂的时候，任何一个partition中的node >= 过这个值，则认定是有效的。 
  数值和集群有一关系  总节点数，必须小于 quorum-size * 2 - 1 ， 当然，也必须大于 quorum。
 
  这个策略的有点是， 在任何脑裂发生的时候，当产生两个 partition的时候，效果是非常显著的， 因为必然会推举出一个 partition 有效。 另一个无效。
  他的缺点是 :
  a)  quorum的数值是静态配置的，如果集群动态增加node，这个值必须随之而调整。
  b） 如果partitions的数量超过2。 比如 10 Nodes，static quorum最小的值是6 ，如果发生脑裂时，将集群划分为3个partition，分别是  (3,3,4) ，那么任何partition都无法运作。
  
  总的来说， Static Quorum相对简单粗放一些，可以应对大部分的异常，也有不少分布式系统使用的是这个策略， 比如 Elasticsearch。
  
  ## 2 Keep Majority
  基本规则是保留整个 partition中，拥有majority nodes的partition。
  他和 static quorum相比的好处是， 如果整个 cluster 的nodes数量是动态调整的，那么majority也是一个动态的值，这样会避免 static quorum的问题。
  但此策略依旧不能避免的是，如果两个 partitions所拥有的node数量是相同的，那么，整个 cluster 将会被终止。
  
  此外，akka提供了基于 keep majority，额外的一个配置项
  ```
   akka.cluster.split-brain-resolver.keep-majority {
    # if the 'role' is defined the decision is based only on members with that 'role'
    role = ""
  }
  ```
  可以对于某些node定义他的价值，如果在多个partition中，拥有这些 valuable的节点数量较多的话，就会保留这个partition。
  这个选项对于一个多角色的集群，某些node是无状态的(stateless)的工作节点，他们相对而言价值较低，而某些节点可能是数据持久化节点，或者是Master节点。
  这样就可以区分出来多个Partition的价值，而保留价值高的部分。
  
  ## 3 Keep Oldest
  顾名思义，整个集群只保留一个拥有最老节点的partition
  当然，有一个特例是，如果某一个partition，只有这一个oldest node，则这个partition会关闭它自己。
  这个策略适用于一些 Singleton 的场景，即某些特殊的服务只存在于 oldest的节点上。
  
  ## 4 Keep Referee
  这个策略和 Keepr oldest类似，只是把 oldest节点替换成了 referee节点，referee节点可以是cluster中的任意一台。
  
  
  # Stable
  最后，回顾一个前提，所有的策略都必须基于集群稳定的情况下，才做出决定。 
  在Cluster Singleton 和  Cluster Sharding 的过程中，新的Instance必须等到老的instance关闭。 
  为了减少集群同时存在两个instance的情况， 有一个 duration 来保持集群(脑裂)状态的稳定。
  以下是akka的文档给出的建议:
  
  ![](/img/akka_split_brain/p1.png)
  

---

作者：3h_william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---