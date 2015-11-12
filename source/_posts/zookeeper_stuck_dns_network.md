title: 使用Zookeeper过程中遇到客户端stuck的问题和解决建议
tags: 
- zookeeper
date: 2015-11-10 9:18:11
---
# 背景
最近在查询一个和zookeeper相关的case。追查结果是一个称之为”Reverse DNS lookup”的问题。
可以称之为:反向DNS查找。 [https://en.wikipedia.org/wiki/Reverse_DNS_lookup](https://en.wikipedia.org/wiki/Reverse_DNS_lookup)

# 隐患 
zookeeper 3.5以下的版本，会在客户端并发链接的时候，有一定几率发生stuck，随着并发连接的增多，发生概率越大。尤其是第一次创建连接的时候。  
(因为zookeeper的client版本和server的版本兼容性，可以先升级客户端，客户端版本>服务端版本，是支持的)

原因: 
-> 1)   归根结底，是因为DNS解析的问题。 可参考:
[issue 1666](https://issues.apache.org/jira/browse/ZOOKEEPER-1666)   

-> 2)   1666 issue在3.4.6已经fix，但是和它相关的另一个issue，只有在3.5才fix 
[issue 1891](https://issues.apache.org/jira/browse/ZOOKEEPER-1891)  


# 案例与解释  

简单的来说，问题出在 “通过IP找HostName”，上， 例如: 通过”10.x.x.x”找到”hadoopXXX” 这个方法上。  
在计算机网络中，我们可以用ip也可以用HostName来表明一台唯一的机器，我们也知道有一个叫做DNS的服务来帮助我们将IP转化为hostName. **但是，这一个转换是有开销的**。  
是需要通过上层的DNS服务器或者IP机器本身的网卡来达到映射。  
至于细节，可以参考wiki  

接下来，我们可以本地来模拟这个过程:  
我们用Java可以启动以下代码，其中，注释的两行是两种构建方式。  


```
        for(int i=0;i<100;i++){
            new Thread(new Runnable() {
                @Override
                public void run() {
//   1                 InetSocketAddress ia = new InetSocketAddress( "10.x.x.x",2181);;
//   2                 InetSocketAddress ia = new InetSocketAddress( "hadoopXXX/10.x.x.x",2181);
                    System.out.println(ia.getHostName());
                }
            }).start();
        }

```

- 当使用注释第一行的时候，线程是有很大的几率发生stuck.  
- 当使用注释第二行的时候，是不会有这个问题。  
而不凑巧对的是，zookeeper3.5以下的版本中，是存在第一行注释的代码的。因此，问题就产生了。  


---

作者：3h-william  
联系: https://github.com/3h-william  
出处：http://3h-william.github.io  

---
 