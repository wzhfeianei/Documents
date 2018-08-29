# 配置Jmeter源码到IDEA

## 1. Fork源代码

分析源码之前最好还是Fork一份源代码到自己的Git仓库,方便注释与更改,Jmeter的源代码仓库为:[https://github.com/apache/jmeter.git](https://github.com/apache/jmeter.git),点击`Fork`,把源代码更新到自己的仓库.

## 2. 拉取源代码到本地

在电脑IDEA的常用工作区打开`gitbash`,使用`git pull 你的仓库地址`命令把刚Fork的仓库代码拉取到本地

## 3. 修改配置文件

进入源代码目录,输入命令

```bat
D:\workspace\jmeter>ren .\eclipse.classpath .classpath

D:\workspace\jmeter>ren .\eclipse.project .project
```

## 4. 导入源代码到IDEA

1. 新建并导入
  ![01-配置Jmeter源码到IDEA环境-2018829163834](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829163834.png)

2. 下一步导入方式选择eclipse模式导入
  ![01-配置Jmeter源码到IDEA环境-2018829164040](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164040.png)
3. 继续默认配置下一步到结束

## 5. 配置Ant

1. 在最右边的“边栏辅助工具”中，找到`ant build`，点开，再找到上方的“+号”，点击，会弹开如下图所示，选择`build.xml`
  ![01-配置Jmeter源码到IDEA环境-2018829164125](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164125.png)
2. 打开build.xml后，找到download_jar，双击，下载jmeter所需要的所有jar包，下载到%jmeter_src%/lib目录下，如下图所示：
  ![01-配置Jmeter源码到IDEA环境-2018829164145](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164145.png)

## 6. 设置项目属性

1. 右键项目后如下图选项,进入项目配置项
  ![01-配置Jmeter源码到IDEA环境-201882916421](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-201882916421.png)
2. 如下图，先选择protocol，再点一下上边的source按钮，之后，右边会出现src/protocol字样，最后，点一下apply按钮，如下图所示：
  ![01-配置Jmeter源码到IDEA环境-2018829164243](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164243.png)
3. 设置dependencies，就是导入jar包，先将所有出错的jar包删除，如下图：
  ![01-配置Jmeter源码到IDEA环境-201882916433](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-201882916433.png)
4. 删除之后,要点一下右下方的`apply`按钮,重新导入所有jmeter所需要的jar包，如下图，点击那个+号，选择`jars or derectories`：  
  ![01-配置Jmeter源码到IDEA环境-2018829164321](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164321.png)
5. 按下图所示，选择lib目录，确定，如下图所示：
  ![01-配置Jmeter源码到IDEA环境-201882916456](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-201882916456.png)
6. 必须重新ant install一下，如下图
  ![01-配置Jmeter源码到IDEA环境-2018829164519](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164519.png)

## 7. 运行与排错

1. 在`\src\core`目录找到NewDriver类，右键点击这个类，这个类是jmeter的main class，在build.xml中有配置，
  ![01-配置Jmeter源码到IDEA环境-2018829164555](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164555.png)
2. 第一次运行时会报错,如下,问题出现在，获取jmeter实例目录时，取的是parent()
  ![01-配置Jmeter源码到IDEA环境-201882916466](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-201882916466.png)
3. 设置一下jmeter.home系统变量了，如下图：
  ![01-配置Jmeter源码到IDEA环境-2018829164620](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164620.png)
4. 在vm options面板中输入如下：-Djmeter.home=D:\workspace\jmeter
  ![01-配置Jmeter源码到IDEA环境-2018829164755](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829164755.png)
5. 日志类的实现类重复报错,去指定目录删除一个实际类,这里删除第一个`log4j-slf4j-impl-2.10`。

   ```java
    SLF4J: Class path contains multiple SLF4J bindings.
    SLF4J: Found binding in [jar:file:/D:/workspace/jmeter/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/D:/workspace/jmeter/lib/opt/activemq-all-5.15.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
    SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
   ```

6. 再次运行,成功进入界面
  ![01-配置Jmeter源码到IDEA环境-2018829172516](http://owo8mviga.bkt.clouddn.com/01-配置Jmeter源码到IDEA环境-2018829172516.png)