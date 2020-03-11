# Btrace工具使用经验

## 简介
Btrace是JVM中的一个代码追踪工具，其原理就是通过JVM对外提供的字节码端口，进行JVM内部运行中的部分字节码修改，将需要执行的额外内容通过注入的形式替换到JVM运行时中，从而达到在指定代码被运行时能够执行额外代码的目的

## 使用

### 基础使用
Btrace的基本使用方法如下
+ 创建一个Btrace的脚本，即一个一般意义上面的JAVA类，并在内部按照Btrace的规则写上注入的方法
+ 使用```btrace <pid> <script.java>```其中pid指的时目标运行的JVM进程pid

其他的细节使用方法见阿里中间件团队的博文
http://jm.taobao.org/2010/11/11/509/

### 注意事项

#### 脚本限制
Btrace的功能十分的强大，但是在其强大功能之外还有不少现实的限制，在默认情况下，Btrace脚本中不允许随意的创建对象，并且能够执行的代码也只能使用默认工具包
```
<dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-client</artifactId>
            <version>1.2</version>
            <scope>provided</scope>
        </dependency>
```
内提供的各个对象和方法，这么做主要是为了确保所插入的字节码对于目标的JVM线程是安全的，这也是Btrace能被拿来进行线上环境DEBUG的原因

#### unsafe脚本操作
Btrace的安全操作往往不能完全满足我们的使用需求，这个时候需要对于Btrace开启非安全操作，包括以下两个部分
+ 在btarce启动脚本文件中加入对应的JVM参数
+ 在编写的插入脚本中声明使用Unsafe模式

找到btrace脚本中具体的java命令,在中间加入输入的参数
```
${JAVA_HOME}/bin/java -Dcom.sun.btrace.unsafe=true  -cp ${BTRACE_HOME}/build/btrace-client.jar:${TOOLS_JAR}:/usr/share/lib/java/dtrace.jar com.sun.btrace.client.Main $*
```

在脚本的类开头进行标签式的声明
```
@BTrace(unsafe = true)
public final class BtraceClusterDelay 
```
