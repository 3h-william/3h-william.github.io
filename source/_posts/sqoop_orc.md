title: 使用sqoop生成ORC格式的文件，并给Hive加载 
tags: 
- sqoop
- hive
date: 2015-09-11 15:00:11
---
# 背景
在当前版本: sqoop-1.4.5(CDH540)中的sqoop并不支持导出格式为orcfile的格式。因此，如果我们想将数据从DB=>HDFS，并生成ORC格式，只有两个办法。

-> a) 使用HCatalog服务，目前没有测试过，看文档是可以生成ORC的格式。
-> b) 原始办法，通过Hive的insert命令将一张原本不是ORC格式的数据，导入到另一个ORC格式中，这样就有了ORC格式的文件。 但该方法会造成时间、资源消耗翻倍。

# DIY  

既然如此，自己动手，丰衣足食。介于近年来对于sqoop的使用经验，很方便的定位到了几个地方，并加入了一些类和方法，最终达到了目的。当然，因为我们的需求目前仅仅是从DB = > HDFS，即只是增加了import方法中的参数，并没有在export => DB中增加。

具体代码可以参考我已经提供的版本 [link](https://github.com/3h-william/sqoop-orc-import) 
使用ant编译后，替换原来的sqoop jar即可。


# 关键点 

-> 1) 使用hive的提供的TypeInfoUtils 构建ORC file的Schema.并传递到每一个Map中
-> 2) 使用 org.apache.hadoop.hive.ql.io.orc.OrcNewOutputFormat 作为输出格式
 
 ---

作者：3h_william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---