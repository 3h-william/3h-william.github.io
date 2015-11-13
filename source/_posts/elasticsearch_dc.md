title: 为ElasticSearch 添加支持跨DC的特性
tags: 
- elasticsearch
- search
date: 2015-08-10 09:50:22
---
# 背景
无意间发现，去年年初提到社区的改动，在去年8月得到回复，虽然没有被采纳…
https://github.com/elastic/elasticsearch/pull/4651#issuecomment-51578376  
对方提出两点:
1. 不赞成ElasticSearch跨越数据中心构建。   
2. 认为同步和延迟会造成比较大的问题。    

### **不过，目前为止，我认为这个特性有它的价值，所以在此分享一些当初的设计思想和改造方法。同时，透露一下，这个改造在公司内部也正常稳定运行了近一年多**  

# 改进点  
Elastic Search的节点设计思想是，每一组索引分片，由一个primary做write操作，而前天的replica做读操作。 
那么，如果对于一个跨数据中心的elastic Search设计，将会因为以下的场景，带来一些问题:
1. 如果所有的写操作是由一个数据中心完成（事实上，大多数情况都是如此），由一个主的数据中心处理数据，然后分发到其他的数据中心(跨location)
2. 而ES的Primary的规则，并不支持将Primary都分配在同属于一个location的Node，而是会根据其自身的集中策略来分配
3. 根据以上两点，最终的结果是: 如果由两个数据中心，Primary可能分配到任何的数据中心，造成write操作会分散在不同的location

### **而我们要做的是，提供一种策略，使得primary分配到同一个location下的index分片**

# 改进效果 

![分布](/img/elasticsearch_dc/1.png)

### 这种策略可以支持任何的情况:
1. primary所在的location全部down掉，只剩下replica的location
2. 任意顺序启动Node.
3. 任意一种HA的可能情况。  


### 代码部分，可参考我已经提交的版本  
https://github.com/3h-william/elasticsearch-dc-version0.0.1


---

作者：3h-william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---
 