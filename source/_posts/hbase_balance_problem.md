title: HBase Coprocessor问题引起异常Balance  
tags: hbase
date: 2015-09-08 14:08:11
---
# 问题背景
最近发现HBase集群的文件compaction次数明显增加。经常到时M/R job扫描snapshot文件的时候发生文件找不到，为此trace了一下日志，大致发现了问题。  
![balance统计](/img/hbase_balance_problems/balance_metrics.png)

# 问题大致流程Trace  

- 1)  全量索引M/R  job 失败   
- 2)  HFile文件不存在  
- 3)  HFile发生了compaction，文件被代替  
- 4)  HFile文件数量达到默认数量(3个) 触发Minor compaction  
- 5)  Memstore被flush，生成了HFile  
- 6)  HRegion 被close之前会触发flush (包括snapshot，倒到Memory的阀值也会触发)，但close是主要原因。  
- 7)  Cluster的Balance操作需要 Close & Move HRegion  
- 8)  StochasticLoadBalancer HBase的Balance策略机制类，触发条件为:  
	- a)  达到Balance触发条件 Max Region > AVG(Region数量) +1  OR  Min Region <AVG(Region数量) -1  
	- b)  调整后的Cost Function值比原来的小（策略更为优秀）  
附:  CF的值:
new ReadRequestCostFunction(conf),
new WriteRequestCostFunction(conf),
new MemstoreSizeCostFunction(conf),
new StoreFileCostFunction(conf)

- 9)  新增的Region数量并不多。不会经常触发balance
- 10)  Balance的大部分Region都是旧的Region，并且 无法调整到最优状态，导致不断调整，死循环。如下某一个时刻的Region分布:
![region分布](/img/hbase_balance_problems/regions_info.png)

只要Region数小于73或者大于76，就会满足条件a。
并且，如果机会调整之后的cost比较小，就会满足b。

- 11)  调整不到平衡点，可能是因为某些调整失败。
- 12)  没有加载Phoenix coprocessor的机器如果balance 一些有该coprocessor的Region就会失败。
- 13)  大规模调整从9/3 18:00开始第一个balance周期。 
- 14)  9/3 17:57 分执行了truncate的操作。 

# Balance Trace 
因为日志涉及公司信息，不再给出，大体步骤如下:   
需要调整的Region号:  XXXXX
- Step 1: 将该Region从server12迁移到server24 (这台机器已dead)
- Step 2: 因为迁移server24失败，所以选择另一台机器进行迁移，选择了server34
- Step 3:  server34加载该Region失败，因为coprocessor的原因。
- Step 4 : 选择server32迁移，也失败了，最终选择了server19成功

Balance的总结:    
- a) 我们可以看到，Balance的策略是失败的，因为最初是想将这个Region迁移到server34，而最终却是迁移到了server19。  
- b) 这样的balance，不仅仅是失败，而且会对server19增加了负担。
- c) 从HBase代码我没有看到对这种情况的处理方式。但理论上一次失败是没有关系的，只要第二次Balance成功即可。
- d) 但是，某些机器注定无法收容某些Region，所以，注定永远无法调整到一个平衡的balance掉。


遗留问题:  
从日志中有一个问题: server24这台机器在9/3 15:29已经dead，为何balance策略仍然会选择这台机器。是不是又bug…

---

作者：3h_william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---
 