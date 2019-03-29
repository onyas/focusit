---
title: 添加一行日志还要重启服务器？
date: 2017-02-17T19:12:20+08:00
categories:
  - Architecture
tags: 
  - java
  - btrace 
---

在工作中我们经常会遇到以下问题：

* 有个方法的入参没有打印，不知道用户输入了什么
* 有个错误日志没有打印堆栈，只是打印了error.getMessage()，不知道上下文
* 某个接口响应时间很慢，不知道哪一步有问题
* 不知道是在哪一步创建了很大的ArrayList,或都HashMap

等等这一系列的问题，如果你没有遇到过这样的问题，停下来想一想要如何解决呢？
如果你遇到过这样的现象，那么你又是如何解决的呢？加日志然后重启服务器吗？

本文就是要介绍一个神器让你不用重启服务器就可以解决以上问题，它就是[Btrace](https://github.com/btraceio/btrace),以下是一个简单的demo，我们一起来看下Btrace的强大功能。

## 应用程序

假设我们有一个应用程序正在生产环境运行，如下代码所示:


```

import java.util.concurrent.TimeUnit;

public class MyService {
    public static void main(String[] args) {
        MyService myService = new MyService();
        while (true) {
            myService.errors();
        }
    }
    public void errors() {
        try {
            TimeUnit.SECONDS.sleep(2);
            int a = 1 / 0;
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}

```

当然以上代码没有什么意义，这里我们想要表达的意思是我们在打印日志的时候只打印了message，没有打印堆栈上下文；如何打印出上下文呢，看下一步


## Btrace 脚本

```

import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.println;

@BTrace
public class MyServiceBtrace {
    @OnMethod(clazz = "MyService", method = "errors",
            location = @Location(value = Kind.CATCH))
    public static void printErrors(@ProbeClassName String cn, @ProbeMethodName String methodName,
                                   @TargetInstance Exception e) {
        println(e);
    }

    @OnMethod(clazz = "MyService", method = "errors",
            location = @Location(value = Kind.ENTRY))
    public static void entry(@ProbeClassName String cn, @ProbeMethodName String methodName) {
        println("entry");
    }
}

```

以上就是Btrace的用法，我们只需要依赖它的一个jar包btrace-boot就可以了，它提供了很多注解帮我们执行想要的功能。比如这里

* @BTrace 声明这是一个Btrace脚本
* @OnMethod 用来指定所拦截的方法
* @Location 用来指定拦截时机，比如第一个是拦截异常被捕获的时候，第二个是拦截刚进入方法的时候

所以上面代码的含义就是声明了两个方法 printErrors和entry，这两个方法都拦截MyService.errors方法，当进入这个方法时entry会触发执行，当捕获异常时printErrors方法会触发。

如果还不懂的话，可以参考[官方文档](https://github.com/btraceio/btrace/wiki)


## 运行脚本

```
// 执行我们的应用程序
javac MyService 
java MyService

//拿到MyService的PID然后执行脚本
btrace -v PID MyServiceBtrace.java
```

以下是程序运行的一些截图，可以看到entry还有printErrors里面的代码都打印出来了

![java-service](/btrace-introduce/java-service.jpg)
![btrace-output](/btrace-introduce/btrace-output.jpg)


## 可能会遇到的问题

```
! ERROR
java.lang.IllegalArgumentException
at com.sun.btrace.org.objectweb.asm.ClassVisitor.(Unknown Source)
at com.sun.btrace.org.objectweb.asm.ClassVisitor.(Unknown Source)
at com.sun.btrace.org.objectweb.asm.tree.ClassNode.(Unknown Source)
at com.sun.btrace.runtime.BTraceProbe.(BTraceProbe.java:73)
at com.sun.btrace.runtime.BTraceProbe.(BTraceProbe.java:84)
at com.sun.btrace.runtime.BTraceProbeFactory.createProbe(BTraceProbeFactory.java:50)
at com.sun.btrace.agent.Client.load(Client.java:399)
at com.sun.btrace.agent.Client.loadClass(Client.java:262)
at com.sun.btrace.agent.RemoteClient.(RemoteClient.java:64)
at com.sun.btrace.agent.Main.startServer(Main.java:580)
at com.sun.btrace.agent.Main.access$000(Main.java:60)
at com.sun.btrace.agent.Main$2.run(Main.java:123)
at java.lang.Thread.run(Thread.java:745)
```

我在第一次运行btrace的时候遇到以上问题，经过排查发现2020端口被其他应用程序占用了，所以把2020端口释放出来重新执行就解决了，当然在执行的时候加 -p 指定执行的端口也可以解决

另外在执行btrace的时候最好加-v看下详细信息，我在第一次执行的时候没有加-v，导致排查了很久才解决。所以这个参数对于我们排查问题还是很重要的。


## 后话
这只是btrace的冰山一角，更多内容和想象可以参考[官网](https://github.com/btraceio/btrace)