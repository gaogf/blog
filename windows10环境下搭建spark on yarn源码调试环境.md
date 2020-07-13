# windows10环境下搭建spark on yarn源码调试环境

​	spark 3.0.0版本刚出炉，官方称相比2.4版本，性能提升了2倍。考虑到生产环境下 spark on yarn的模式居多，本文在Windows10环境下，搭建spark on yarn的源码阅读调试环境。使得最新功能能够以最快的速度反哺生产环境，进一步提升线上性能。spark3.0的最新功能可以参考官方博客内容：https://databricks.com/blog/2020/06/18/introducing-apache-spark-3-0-now-available-in-databricks-runtime-7-0.html

## 依赖工具

1.jdk 1.8

2.scala 2.12

3.maven 3.6

4.IDEA

5.winutils.exe (对应hadoop版本2.9.2)

6.hadoop (2.9.2)

7.spark3.0

## 步骤

​	winutils.exe是在Windows系统上需要的hadoop调试环境工具，里面包含一些在Windows系统下调试hadoop、spark所需要的基本的工具类，另外在使用eclipse调试hadoop程序时，也需要winutils.exe 。
 下载地址：https://github.com/steveloughran/winutils
 下载后的winutils.exe放到HADOOP_HOME/bin目录下。

### 设置hadoop环境变量

​	在系统变量path里增加%HADOOP_HOME%\bin

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200620222554922.png" alt="image-20200620222554922" style="zoom:70%;" />

### 下载spark源码

在spark官方网站下载spark源码：https://spark.apache.org/

下载后进行解压，进入源码根路径，因为想要调试在yarn下和kubernetes下的资源调度流程，设置yarn 和kubernetes选项：

```shell
./build/mvn -Pyarn -Dhadoop.version=2.9.2 -Phive -Phive-thriftserver -Pkubernetes -DskipTests clean package
```

![image-20200620223728410](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200620223728410.png)

耐心等待半个小时左右。编译结果如下

![image-20200620233854925](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200620233854925.png)

### 将编译后的代码导入IDEA

用`git bash`以管理员身份运行`build/spark-build-info` 用以生成`spark-version-info.properties`文件
 `build/spark-build-info D:\opensource\spark-3.0.0\core\target\extra-resources\ 3.0.0`
 将生成的`spark-version-info.properties`文件复制到`spark-core_2.12-3.0.0.jar`的根目录下。(复制之前先检查根目录下是否存在`spark-version-info.properties`，不存在再复制)在conf目录下复制`log4j.properties.template`，重命名为`log4j.properties`将`spark\assembly\target\scala-2.12\jars`目录下的所有jar包添加到`classpath`中。

![image-20200620235624737](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200620235624737.png) 运行`JavaLogQuery`示例代码：

![image-20200620235707614](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200620235707614.png)

本地运行的方式调试成功。接下来配置调试spark on yarn 的方式。

首先启动hadoop单点服务，启动namenode，datanode，yarn.

首先通过start-dfs.cmd和start-yarn.cmd启动hadoop环境.

![image-20200621091628573](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621091628573.png)

yarn

![image-20200621091826174](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621091826174.png)

我们以SparkSubmit的测试用例为入口，设置相关的启动参数。

![image-20200621101536332](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621101536332.png)

运行

![image-20200621101626121](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621101626121.png)

查看yarn的web界面

![image-20200621101334326](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621101334326.png)

之后我们可以从SparkSubmit.scala为入口，设置断点进行debug跟读了。