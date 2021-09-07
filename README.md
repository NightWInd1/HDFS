一个正在学习大数据开发的普通学生
========================
HDFS简介（Hadoop中三大件之一，主要是起到存储功能）
------------
>1.概述（了解）
----------
>>1.1 HDFS出现的背景
---
>>1.2 HDFS定义
---
HDFS是一个分布式文件系统，用来存储文件，通过目录树结构来定位文件
HDFS使用场景：适合一次写入，多次读出，不支持文件修改，适合用来做数据分析，不适合用来做网盘
>>1.3 优缺点
---
>>>1.3.1 优点：
---
>>>>1）高容错性
>>>>2）适合处理大数据
>>>>3）可以部署在廉价机器上，通过多副本机制，提高可靠性
-----
>>>1.3.2 缺点
---
>>>>1）不适合低延时数据访问
>>>>2）无法高效对大量小文件进行存储（因为小文件太多会占用太多的NameNode，从而导致内存不够）
>>>>3）不不支持并发写入，文件随机修改（仅支持追加append）
----
>>1.4 HDFS组成架构（重要） 
------
![Snipaste_2021-09-07_14-02-17](https://user-images.githubusercontent.com/32889586/132292300-1039e166-7eb2-42b7-96e9-df1c17bae963.jpg)
![Snipaste_2021-09-07_14-12-11](https://user-images.githubusercontent.com/32889586/132293239-3653a1a1-8505-4c05-ae4a-047e6babcdc6.jpg)<br>	
>>>>其实可以理解为NameNode是主管，管理不同工人workers即DataNode,一天客户（Client）来谈生意打算存储自己的东西（上传），首先和主管交流，然后主管叫来工人，带领客户去仓库（相当于DateNode中Block操作）存放东西（Write），并记录好东西的信息，又因为考虑到安全，所以在不同仓库（2个到3个）又分别存储一份，存放完毕后，当客户想来取东西的时候（下载）（DownLoad），又要先和主管交流，主管在去叫工人带他去存放数据的仓库取出东西（Read）
注意：Block分块操作一般是是128M（与磁盘存储性能相关），这个不是实际磁盘空间，只是一个参数，如果存放的东西超过128M，就进行切块(当然切块并不是真的指将磁盘切开，而是又设置一个块储存剩下的数据，当进行读写的操作时依次读写块)，分别储存，如果不足128M，就实际占用多少就分多少
![关于开辟128M空间的理解](https://user-images.githubusercontent.com/32889586/132327846-dd9ab92f-e6c8-4a85-811e-5feea0ee09a1.jpg)
----
>2.HDFS中的Shell操作（重点）
-----------------
>>hdfs dfs -命令 与 hadoop fs -命令 效果是一摸一样的，一般偏向于使用后者
--------------------
>>2.1 上传
>>>hadoop fs -moveFromLocal(从本地仓库上传到集群，并删除源文件) 文件本地路径 /集群中的路径
>>>haddop fs -copyFromLocal(从本地仓库上传到集群) 文件本地路径 /集群中的路径（在HDFS系统中的路径）
>>>hadoop fs -put(效果和copyFromLocal一摸一样) 文件本地路径 /集群中的路径
----------------------
>>2.2 下载
>>>hadoop fs -copyToLocal(从集群中下载到本地仓库) 集群中的路径 /本地路径
>>>hadoop fs -get(效果和上相同)
>>>hadoop fs -getmerge:合并下载多个文件，比如HDFS的目录 /user/atguigu/test下有多个文件:log.1, log.2,log.3,...
>>>>实例代码：[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -getmerge /user/atguigu/test/* ./zaiyiqi.txt
------------
>>2.3 其他常用命令
>>>1）-ls: 显示目录信息
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -ls /
>>>2）-mkdir：在HDFS上创建目录
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir -p /sanguo/shuguo
>>>3）-cat：显示文件内容
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cat /sanguo/shuguo/kongming.txt
>>>4）-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo/kongming.txt
>>>5）-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cp /sanguo/shuguo/kongming.txt /zhuge.txt
>>>6）-mv：在HDFS目录中移动文件
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /zhuge.txt /sanguo/shuguo/
>>>7）-tail：显示一个文件的末尾
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -tail /sanguo/shuguo/kongming.txt
>>>8）-rm：删除文件或文件夹
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm /user/atguigu/test/jinlian2.txt
>>>9）-rmdir：删除空目录
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir /test
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rmdir /test
>>>10）-du统计文件夹的大小信息
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du -s -h /user/atguigu/test
2.7 K  /user/atguigu/test
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du  -h /user/atguigu/test
1.3 K  /user/atguigu/test/README.txt
15     /user/atguigu/test/jinlian.txt
1.4 K  /user/atguigu/test/zaiyiqi.txt
>>>11）-setrep：设置HDFS中文件的副本数量
>>>>[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -setrep 10 /sanguo/shuguo/kongming.txt
>>>>这里设置的副本数只是记录在NameNode的元数据中，是否真的会有这么多副本，还得看DataNode的数量。因为目前只有3台设备，最多也就3个副本，只有节点数的增加到10台时，副本数才能达到10。
---
>3.HDFS客户端API操作（较为重要）
---------
>4.HDFS中的读写过程、步骤(面试重点)
--------
>>4.1 HDFS数据流
------
>>4.1 HDFS写数据流程
-----
>>4.2 HDFS读数据流程
-----
>>4.2 NameNode和SecondaryNameNode
-----
>>4.3 DateNode
------
 
