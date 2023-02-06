Windows环境下hadoop安装和配置详细步骤
一、下载Hadoop
http://www.apache.org/dyn/closer.cgi/hadoop/common
（我下载的版本是hadoop-2.7.3.tar.gz，这里就以此版本为例）
下载完成后解压，把hadoop-2.7.3放到某个盘的根目录比如 D：\hadoop-2.7.3 （这样方便打开）。
原版的Hadoop不支持Windows系统，我们需要修改一些配置方便在Windows上运行，需要从网上搜索下载hadoop对应版本的windows运行包(网上自行下载对应版本的运行包)。
下载完之后复制解压开的bin文件和etc文件到hadoop-2.7.3文件中，并替换原有的bin和etc文件。

二、配置Hadoop的环境变量
1.配置Java环境变量（配置过的就可以跳过）
新建变量名：JAVA_HOME
输入路径：D:\Softwares\jdk1.8 （请根据自己的文件位置来设置）
在path中最前面加上：%JAVA_HOME%\bin;

2.配置Hadoop环境变量
新建变量名：HADOOP_HOME
输入路径：E:\hadoop-2.7.3（请根据自己的文件位置来设置）
在path中最前面加上：%HADOOP_HOME%\bin;

确认hadoop配置的jdk的路径
在hadoop-2.7.3\etc\hadoop找到hadoop-env.cmd，右键用一个文本编辑器打开找到 ：
set JAVA_HOME=C:\PROGRA~1\Java\jdk1.7.0_67

将C:\PROGRA~1\Java\jdk1.7.0_67 改为 D:\Softwares\jdk1.8（在环境变量设置中JAVA_HOME的值）(如果路径中有“Program Files”，则将Program Files改为 PROGRA~1）

配置好上面所有操作后，win+R 输入cmd打开命令提示符，然后输入hadoop version，按回车，如果出现如图所示结果，则说明安装成功


三、hadoop核心配置文件
在hadoop-2.7.3\etc\hadoop中找到以下几个文件用文本编辑器打开
1.打开 hadoop-2.7.3/etc/hadoop/core-site.xml, 复制下面内容粘贴到最后并保存
```
<configuration>
	<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>   
	</property>
</configuration>
```
2.打开 hadoop-2.7.3/etc/hadoop/mapred-site.xml, 复制下面内容粘贴到最后并保存

```
<configuration>   
	<property>       
	<name>mapreduce.framework.name</name>       
	<value>yarn</value>   
	</property>
</configuration>
```
3.打开 hadoop-2.7.3/etc/hadoop/hdfs-site.xml, 复制下面内容粘贴到最后并保存

先创建两个文件夹

```
E：/hadoop-2.7.3/namenode
E：/hadoop-2.7.3/datanode
```

```
<configuration>
	<property>       
	<name>dfs.replication</name>       
	<value>1</value>   
	</property>   
	<property>       
	<name>dfs.namenode.name.dir</name>       
	<value>/E:/hadoop-2.7.3/namenode</value>//路径为你的存放路径   
	</property>   
	<property>       
	<name>dfs.datanode.data.dir</name>     
	<value>/E:/hadoop-2.7.3/datanode</value>//路径为你的存放路径   
	</property>
</configuration>
```
4.打开 hadoop-2.7.3/etc/hadoop/yarn-site.xml,复制下面内容粘贴到最后并保存

```
<configuration>   
	<property>       
	<name>yarn.nodemanager.aux-services</name>       
	<value>mapreduce_shuffle</value>   
	</property>   
	<property>       
	<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>       
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>   
	</property>
</configuration>
```
（以上这些路径都是根据自己的实际路径！）

四、启动Hadoop服务
以管理员身份打开命令提示符
到Hadoop-2.7.3\bin下，输入hdfs namenode -format执行到如下图所示

格式化之后，namenode文件里会自动生成一个current文件，则格式化成功。

然后转到Hadoop-2.7.3\sbin下，输入start-all.cmd，启动hadoop服务，等待他启动完成。
完成之后，输入jps可以查看运行的所有服务 (前提是java路径设置正确)

显示的就是正在运行的，确保以上服务全部都启动了！

到这在Windows环境下的Hadoop就安装好了！

高版本缺少win插件 https://github.com/cdarlint/winutils



问题所在：hadoop启动报错，缺少hadoop-yarn-server-timelineservice jar包

解决办法：将hadoop-2.9.2\share\hadoop\yarn\timelineservice下 hadoop-yarn-server-timelineservice-2.9.2.jar 复制到hadoop-2.9.2\share\hadoop\yarn\lib目录下