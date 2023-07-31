# Pycharm链接集群环境配置（Mac）

准备工作：

1、使用专业版的Pycharm（本文中是）

2、现有集群以及对应的权限

3、Pycharm配置SSL



#### 一、专业版的Pycharm

###### 	参考https://www.yuque.com/ningmengna-6ulrv/whpu0w/sxevlf1wd6amur51

#### 二、配置ssh链接

###### 使用⌘+，或者pycharm--->preferences进入配置界面

![image-20230731104923085](/Users/violy/Library/Application Support/typora-user-images/image-20230731104923085.png)

###### 具体操作如下图所示

![image-20230731105133206](/Users/violy/Library/Application Support/typora-user-images/image-20230731105133206.png)

###### 在弹出的小框中继续配置如下

![image-20230731105317805](/Users/violy/Library/Application Support/typora-user-images/image-20230731105317805.png)

![image-20230731105414594](/Users/violy/Library/Application Support/typora-user-images/image-20230731105414594.png)

![image-20230731105433466](/Users/violy/Library/Application Support/typora-user-images/image-20230731105433466.png)



###### 一般使用system interpreter联机远程开发，interpreter会自动识别集群Python解释器路径，如要另外配置可以点击旁边的按钮修改

###### 配置本地路径和线上路径：

本地路径：Mac代码保存的位置

线上路径：集群中刚刚ssh链接填写的host机器下的路径，用来同步上传本地路径保存的文件

![image-20230731105656807](/Users/violy/Library/Application Support/typora-user-images/image-20230731105656807.png)

配置完成以后会到配置页，选择你刚刚配置的Python interpreter（这里配置了三次，所以有三个）

![image-20230731110606683](/Users/violy/Library/Application Support/typora-user-images/image-20230731110606683.png)

在配置页的deployment中可以对刚刚建立的服务重命名或者修改配置路径

![image-20230731111003149](/Users/violy/Library/Application Support/typora-user-images/image-20230731111003149.png)

![image-20230731111123960](/Users/violy/Library/Application Support/typora-user-images/image-20230731111123960.png)

测试联通集群并运行代码通过

```
from pyspark.sql import SparkSession
spark = SparkSession.builder \
    .appName("test") \
    .config("spark.sql.catalogImplementation", "hive") \
    .enableHiveSupport() \
    .getOrCreate()

if __name__ == '__main__':
    spark.sql("use data_prod")
    spark.sql("show tables").show()
    print(spark.conf.get("spark.sql.warehouse.dir"))
```

![image-20230731111339654](/Users/violy/Library/Application Support/typora-user-images/image-20230731111339654.png)



#### 四、链接报错

###### 1、No such file or directory

如果运行代码出现[Errno 2] No such file or directory，检查自己的mapping是否正确（Tools--->Deployment--->Configuration--->Mappings--->Deployment path）

或者从刚刚配置页的deployment进入也可以，将Deployment path改为项目在集群上的位置，具体操作步骤如下图所示：

![image-20230731112543084](/Users/violy/Library/Application Support/typora-user-images/image-20230731112543084.png)

![image-20230731112910798](/Users/violy/Library/Application Support/typora-user-images/image-20230731112910798.png)

###### 2、pythonpath错误

如果报pythonpath错误如下，本地和线上环境不一致，需要配置pycharm环境变量PYTHONPATH

```
pythonpath = os.environ.get("PYTHONPATH") print("PYTHONPATH:", pythonpath) 输出： PYTHONPATH: /opt/apps/datax/job/pyjob/:/home/meifutedb/.pycharm_helpers/pycharm_matplotlib_backend:/home/meifutedb/.pycharm_helpers/pycharm_display
```

查看集群的PYTHONPATH

```
echo $PYTHONPATH
```

如果打印为空表示集群没有配置PYTHONPATH环境变量，

py脚本可以在集群上运行成功，但是pycharm远程失败的情况下在集群先配好PYTHONPATH

集群终端进入python3 执行

```
import os 
pythonpath = os.environ.get("PYTHONPATH") 
print("PYTHONPATH:", pythonpath)
PYTHONPATH: :/opt/apps/spark-hadoop/python/lib/pyspark.zip:/opt/apps/spark-hadoop/python/lib/py4j-0.10.9.5-src.zip
```

![image-20230731114935282](/Users/violy/Library/Application Support/typora-user-images/image-20230731114935282.png)

然后在集群配置文件中配置全局变量

```
export PYTHONPATH=$PYTHONPATH:/opt/apps/spark-hadoop/python/lib/pyspark.zip:/opt/apps/spark-hadoop/python/lib/py4j-0.10.9.5-src.zip
```

###### 3、import _ssl错误

如果报import _ssl错误如下，本地和线上环境不一致，需要配置pycharm环境变量LD_LIBRARY_PATH

```
  File "/usr/local/python3/lib/python3.10/site-packages/pyhive/hive.py", line 15, in <module>
    from ssl import CERT_NONE, CERT_OPTIONAL, CERT_REQUIRED, create_default_context
  File "/usr/local/python3/lib/python3.10/ssl.py", line 99, in <module>
    import _ssl             # if we can't import it, let the error propagate
ImportError: libssl.so.1.1: cannot open shared object file: No such file or directory 
```

![image-20230731113352922](/Users/violy/Library/Application Support/typora-user-images/image-20230731113352922.png)

LD_LIBRARY_PATH和PYTHONPATH的值均为集群里面的环境配置

查看自己集群的环境

echo $LD_LIBRARY_PATH

echo $PYTHONPATH

![image-20230731113609899](/Users/violy/Library/Application Support/typora-user-images/image-20230731113609899.png)