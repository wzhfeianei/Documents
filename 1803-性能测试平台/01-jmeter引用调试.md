##引入core包

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/C:/JAVA/gradle-4.4/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-slf4j-impl/2.10.0/8e4e0a30736175e31c7f714d95032c1734cfbdea/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/D:/Repository/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Exception in thread "main" java.lang.StackOverflowError

```

## 不能找到NewDiver类，引用Jmeter包

## 各种访问错误 

```
java.lang.Throwable: Could not access D:\workspace\lib
	at org.apache.jmeter.NewDriver.<clinit>(NewDriver.java:102)
	at org.apache.jmeter.JMeter.initializeProperties(JMeter.java:714)
	at org.apache.jmeter.JMeter.start(JMeter.java:441)
	at com.example.jmdemo.JmeterStart.main(JmeterStart.java:16)
java.lang.Throwable: Could not access D:\workspace\lib\ext
	at org.apache.jmeter.NewDriver.<clinit>(NewDriver.java:102)
	at org.apache.jmeter.JMeter.initializeProperties(JMeter.java:714)
	at org.apache.jmeter.JMeter.start(JMeter.java:441)
	at com.example.jmdemo.JmeterStart.main(JmeterStart.java:16)
java.lang.Throwable: Could not access D:\workspace\lib\junit
	at org.apache.jmeter.NewDriver.<clinit>(NewDriver.java:102)
	at org.apache.jmeter.JMeter.initializeProperties(JMeter.java:714)
	at org.apache.jmeter.JMeter.start(JMeter.java:441)
	at com.example.jmdemo.JmeterStart.main(JmeterStart.java:16)
11:11:35.093 [main] ERROR org.apache.jmeter.JMeter - An error occurred: 
java.lang.RuntimeException: Could not read JMeter properties file:D:\workspace\bin\jmeter.properties
	at org.apache.jmeter.util.JMeterUtils.loadJMeterProperties(JMeterUtils.java:205)
	at org.apache.jmeter.JMeter.initializeProperties(JMeter.java:714)
	at org.apache.jmeter.JMeter.start(JMeter.java:441)
	at com.example.jmdemo.JmeterStart.main(JmeterStart.java:16)
An error occurred: Could not read JMeter properties file:D:\workspace\bin\jmeter.properties
```
临时将目录拷贝过去 

## 不能打开文件，参数写法问题

```
Could not open  D:/workspace/jmdemo/tpxy4.jmx
```
错误 

```
jMeter.start(new String[]{"-n","-t D:/workspace/jmdemo/tpxy4.jmx"});
```

正确
```
jMeter.start(new String[]{"-n","-tD:/workspace/jmdemo/tpxy4.jmx"});
```

##包的加载 


```
CannotResolveClassException: org.apache.jmeter.protocol.http.sampler.HTTPSamplerProxy
```
加载对应JAR包

