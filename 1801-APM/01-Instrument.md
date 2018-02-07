# 重温Instrument机制
## Instrument介绍

 当互联网服务越来越碎片化，服务的细化与微服务，加上越来越多的用分布式方式部署，服务与服务之间相互调用，可能有数以百计的调用链，一旦中间某个环节发生问题，那么对问题的追踪和定位则比单体应用的定位方式复杂无数倍。于是[Dapper](http://bigbully.github.io/Dapper-translation/)应运而生，谷歌发布相关论文后，各自现代化的调用链追踪的APM监控软件如雨后春笋般的纷纷出现，并且在过去两年都获得了资本的不少的亲睐，连国内的OneAPM，听云等相关APM厂商均获取到大量融资。各家大厂也均建立自己的APM产品，而大部分APM相关产品均基于Dapper和Instruments技术，随着Apache的Opentracing的标准化组织的建立，未来APM相关标准和规范会越来越统一化，建立在APM的数据收集和性能监控等会给应用带来更多的效益。Instrument技术是JAVA5加入的特性，之前看曾经看过，时间久了也忘记个差不多了，近期看了一个开源的Skywalking项目，再重温一下相关原理。

## 创建原始demo应用

### 一个简单的Pig类
```java
/**
 * 描述:
 * pig
 *
 * @outhor weizhanfei
 * @create 2018-02-05
 */
public class Pig {
    public String sayHello() {

        return "heng,heng,heng";
    }
}

```
### 一个Person类

其中person类与Pig类均有一个sayHello方法，假定Person类听到Pig叫时做出回应，简写制定为Persono在的sayHello方法调用Pig的sayHello方法
```java
/**
 * 描述:
 * person
 *
 * @outhor weizhanfei
 * @create 2018-02-05
 */
public class Person {
    public String sayHello() {
        String pigHello=new Pig().sayHello();
        System.out.println(pigHello);
        return "Hello Pig";
    }
}

```
### 创建Main入口
生成一个Person类循环执行数干分钟
```java
public class Main {

    public static void main(String[] args) {
        int i = 0;
        while (i < 60) {
            System.out.println(new Person().sayHello());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            i++;
        }

    }

}

```

### 打JAR并执行
打成JAR包随便放个位置并执行对应JAR包，其效果为
```shell
C:\Users\Administrator\Desktop>java -jar demo.jar
heng,heng,heng
Hello Pig
heng,heng,heng
Hello Pig
heng,heng,heng
Hello Pig
heng,heng,heng
Hello Pig
```

## 创建基于Instrument的Agent类

### 创建transformer实现类
另建一个JAVA工程，在工程实现一个Instrument，实现对相关目标类的编辑、功能增强等相关功能,在这里实现的如果目标应用在调用方法时自动打印出是哪个类调用的哪个方法

```java
import jdk.internal.org.objectweb.asm.ClassReader;
import jdk.internal.org.objectweb.asm.ClassWriter;
import jdk.internal.org.objectweb.asm.Opcodes;
import jdk.internal.org.objectweb.asm.tree.*;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

/**
 * 描述:
 * transformer
 *
 * @outhor weizhanfei
 * @create 2018-02-05
 */
public class DemoTransformer implements ClassFileTransformer {
    //ASM字节码编程，比较晦涩难懂，可以用其它的字节码框架实现
  
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {

        ClassReader cr = new ClassReader(classfileBuffer);
        ClassNode cn = new ClassNode();
        cr.accept(cn, 0);
        for (Object obj : cn.methods) {
            MethodNode md = (MethodNode) obj;
            if (!"sayHello".endsWith(md.name) || "<init>".endsWith(md.name) || "<clinit>".equals(md.name)) {
                continue;
            }
            InsnList insns = md.instructions;
            InsnList il = new InsnList();
            il.add(new FieldInsnNode(Opcodes.GETSTATIC, "java/lang/System",
                    "out", "Ljava/io/PrintStream;"));
            il.add(new LdcInsnNode("Calling method-> " + cn.name + "." + md.name));
            il.add(new MethodInsnNode(Opcodes.INVOKEVIRTUAL,
                    "java/io/PrintStream", "println", "(Ljava/lang/String;)V"));
            insns.insert(il);
            md.maxStack += 3;

        }
        ClassWriter cw = new ClassWriter(0);
        cn.accept(cw);
        return cw.toByteArray();
    }

//    @Override
//    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
//        return null;
//    }

}


```

### permain方法实现 
代理 JAR 文件的清单必须包含 Premain-Class 属性。此属性的值是代理类 的名称。代理类必须实现公共静态premain 方法，该方法的原理与 main 应用程序入口点类似。在 Java 虚拟机 (JVM) 初始化后，每个 premain 方法将按照指定代理的顺序调用，然后将调用实际的应用程序 main 方法。每个 premain 方法必须按照依次进行的启动顺序返回。

premain 方法有两种可能的签名。JVM 首先尝试在代理类上调用以下方法：

```
public static void premain(String agentArgs, Instrumentation inst);
```

如果代理类没有实现此方法，那么 JVM 将尝试调用：
```
public static void premain(String agentArgs);
```
我们已经建立好对应的Instrumentation，则直接使用第一种方法

```java
import java.lang.instrument.Instrumentation;

/**
 * 描述:
 * Agent
 *
 * @outhor weizhanfei
 * @create 2018-02-05
 */
public class DemoAgent {
    public static void premain(String agentArgs, Instrumentation inst) {
        if (null != agentArgs) {
            System.out.println("I've been called with agentArgs: " + agentArgs);
        } else {
            System.out.println("I've been called with no agentArgs.");
        }

        inst.addTransformer(new DemoTransformer());

    }
}

```
###打JAR

打包的时候应该注意一下上面提到过代理类的文件清单必须包括`Premain-Class`属性，在`MANIFEST.MF`文件内修改为以下内容
```
Manifest-Version: 1.0
Premain-Class: DemoAgent
Can-Redefine-Classes: true

```
修改之后打成对应的JAR包，放在固定位置，方便期间可以和demo放在一个目录下。

### 以--javaagent方式运行demo.jar

在命令栏里输入`java -javaagent:instrument.jar -jar demo.jar`命令
可以看到对应的显示结果为:
```shell
C:\Users\Administrator\Desktop>java -javaagent:instrument.jar -jar demo.jar
I've been called with no agentArgs.
Calling method-> Person.sayHello
Calling method-> Pig.sayHello
heng,heng,heng
Hello Pig
Calling method-> Person.sayHello
Calling method-> Pig.sayHello
heng,heng,heng
Hello Pig
Calling method-> Person.sayHello
Calling method-> Pig.sayHello
heng,heng,heng
Hello Pig
```
##限制与补充

使用permain方式的代理必须在启动JAR时使用--javaagent方法调用，在加载类的时候预先对目标类进行，如果想在加载后再使用代理的方法，那就必须要使用`agentmain `

实现可以提供一种机制在 VM 启动之后某一时刻启动代理。如何初启的细节是特定于实现的，但通常应用程序已经启动并且其 main 方法已经调用。如果实现支持在 VM 启动之后启动代理，则以下内容适用：

1. 代理 JAR 的清单必须包含属性 Agent-Class。此属性的值是代理类 的名称。

2. 代理类必须实现公共静态 agentmain 方法。

3. 系统类加载器（ClassLoader.getSystemClassLoader）必须支持将代理 JAR 文件添加到系统类路径的机制。


代理 JAR 将被添加到系统类路径。系统类路径是通常加载包含应用程序 main 方法的类的类路径。代理类将被加载，JVM 尝试调用 agentmain 方法。JVM 首先尝试对代理类调用以下方法：
```
public static void agentmain(String agentArgs, Instrumentation inst);
```
如果代理类没有实现此方法，那么 JVM 将尝试调用：
```
public static void agentmain(String agentArgs);

```

## 后记
可以看出最后输入结果显示什么类调用了什么方法，当然只是纯粹的显示调用了哪里，没有上下游的调用追踪技术，配置Dapper的原理在指定方法和服务内标记好traceId并存储到数据库，通过分析就可以分析出哪个方法调用了哪个方法，使用的什么参数，期间消耗的时间是多少，用户的业务访问热点是哪些，从而配合后期开发及业务做对应的数据分析。