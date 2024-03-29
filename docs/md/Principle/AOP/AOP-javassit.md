---
layout: post
title:  2023-03-25 XMind是怎么被破解的？
tagline: by 沉浮
categories: 
tags: 沉浮
---

哈喽，大家好，我是了不起。

有没有想过，XMind是如何被破解的？那么今天我们就来看看javaassit这项技术，其实在你接触的很多其他工具中这个工具早就被广泛使用了

<!--more-->
## javaassit

我们知道，java是一门面向对象的编程语言，更是一门面向切面的编程语言，正是这个特性，让Java更加地灵活。

可能你写过基于Spring AOP的代码，其原理都是基于JDK动态代理或者CGLIB来实现，其局限性在于我们只能以方法作为连接点，来实现基于方法执行过程的代理。

你可还知道更厉害的代理工具：AspectJ、javaassit，这些都是基于字节码，属于更底层，但是功能更强大的代理。


### 知识点

+ ASM

通过指令修改class字节码，主要基于ClassReader结合JVM指令集直接操作字节码，Cglib即是通过该技术实现。

+ JavaAssit

基于org.javassist:javassist类库提供的CtPool工具类对字节码进行修改

+ Instrumentation

JVM提供的一个可以修改已加载类的类库，通过编写java代码即可完成对字节码的修改

+ JavaAgent

JVM加载类之前与JVM运行时，基于JavaAssit、Instrumentation实现字节码修改并加载到JVM

###  应用场景

+ IDE的调试功能，例如 Eclipse、IntelliJ IDEA
+ 热部署功能，例如 JRebel、XRebel、spring-loaded
+ 线上诊断工具，例如 Btrace、Greys，还有阿里的 Arthas
+ 性能分析工具，例如 Visual VM、JConsole、TProfiler等
+ 全链路性能检测工具，例如 Skywalking、Pinpoint等

### 示例

下面我们基于javaagent以及运行时Attach的模式看下javaassit如何实现目标类的代理的：

> 基于javaagent

1. 编写代理类

方法签名固定，方法名为**premain**，参数分别对应args（不是数组）以及Instrumentation
```java
public class JavaAgent {

    private static final String TARGET_CLASS_NAME = "com.sucl.blog.javaassit.Target";

    public static void premain(String args, Instrumentation instrumentation){
        AgentHelper.create(TARGET_CLASS_NAME).proxy(args, instrumentation);
    }

}
```
2. 打包代理类

这里我们借助maven插件**maven-shade-plugin**，主要是为了打包时修改/META-INF/MANIFEST.MF文件，需要加上Premain-Class这项
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.3</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <manifestEntries>
                            <Premain-Class>com.sucl.blog.agent.JavaAgent</Premain-Class>
                            <Agent-Class>com.sucl.blog.agent.AttachAgent</Agent-Class>
                            <Can-Redefine-Classes>true</Can-Redefine-Classes>
                            <Can-Retransform-Classes>true</Can-Retransform-Classes>
                        </manifestEntries>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

3. 编写测试类

目的很简单，每隔3秒打印当前时间
```java
public class JavaAgentMain {
    public static void main(String[] args) throws InterruptedException {
        Target target = new Target();
        while (true) {
            target.print(new Date());
            TimeUnit.SECONDS.sleep(3);
        }
    }
}

@Slf4j
class Target {
    public void print(Object obj) {
        log.info("打印内容：{}", obj);
    }
}
```

4. 配置代理

如何让我们编写的代理生效，这里提供两种方法：

+ 当你使用IDEA启动时，可以在Config Configurations中通过配置VM OPTION，添加如下内容:

 -javaagent:/your_jar_path/agent.jar=param=value

+ 当你使用java命令启动时：

java -javaagent:/path/agent.jar=param=value -jar xxx.jar

5. 测试

执行测试类main方法，你可以看到，在打印时间前后，分别会打印“开始执行方法：print”，“结束执行方法：print”，这也是我们代理类实现的功能。
```
>>> 开始执行方法：print
14:46:09.457 [main] INFO com.sucl.blog.javaassit.Target - 打印内容：Fri Mar 10 14:46:09 CST 2023
>>> 结束执行方法：print
```

---

> 基于Attach

1. 编写代理类

方法签名固定，方法名为**attachmain**，参数分别对应args（不是数组）以及Instrumentation;
和上面的相比唯一的不同是方法名称。
```java
public class AttachAgent {
    
    private static final String TARGET_CLASS_NAME = "com.sucl.blog.javaassit.Target";
    
    public static void agentmain(String args, Instrumentation instrumentation){
        System.out.println(String.format(">>> agentmain starting, args: %s",args));
        AgentHelper.create(TARGET_CLASS_NAME).proxy(args, instrumentation);
        System.out.println(String.format(">>> agentmain finished"));
    }

}
```

2. 打包代理类

同样借助插件**maven-shade-plugin**，主要是为了打包时修改/META-INF/MANIFEST.MF文件，需要加上Agent-Class这项
```xml
<!-- 省略 ...-->
<Agent-Class>com.sucl.blog.agent.AttachAgent</Agent-Class>
<!-- 省略 ...-->
```

注意，这里我们使用了ClassPool、CtClass、CtMethod相关的类，记得在pom.xml中引入对应的依赖
```xml
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
</dependency>
```

3. 编写测试类

测试类完全一样，由于启动代理织入的方式不一样，因此分为两个类
```java
public class AttachAgentMain {

    public static void main(String[] args) throws InterruptedException {
        Target target = new Target();
        while (true) {
            target.print(new Date());
            TimeUnit.SECONDS.sleep(3);
        }
    }

}
```

4. 执行代理

如何将编写的代码（AttachAgent）织入到目标类完成对目标类（Target）方法的代理？

这里我们需要用到jdk中的tool.jar，你可以在测试模块中添加下面的依赖：
```xml
<dependency>
    <groupId>com.sun</groupId>
    <artifactId>tools</artifactId>
    <version>1.8</version>
    <scope>system</scope>
    <systemPath>${java.home}/../lib/tools.jar</systemPath>
</dependency>
```

如何在运行时进行代理织入：
```java
public class AttachAgentTests {

    private static String JAR_PATH = AttachAgentTests.class.getClassLoader().getResource("").getPath().replace("test-classes/","")+"agent.jar";
    
    @Test
    public void attachAgent() throws Exception {
        String pid = findPid(KEY); // 通过jps命令找到AttachAgentMain执行的pid
        VirtualMachine virtualMachine = VirtualMachine.attach(pid);
        virtualMachine.loadAgent(JAR_PATH.substring(1));
        virtualMachine.detach();
    }
}
```

5. 测试

+ a. 先执行测试代码（AttachAgentMain.java），此时每间隔3秒会打印当前时间。
+ b. 执行代理织入方法（AttachAgentTests#attachAgent）
+ c. 观察测试代码输出结果，你会会发现此时每次打印时间前后都会有“开始执行方法：print”，“结束执行方法：print”

--- 

**AgentHelper**
```java
public class AgentHelper {

    private String targetClassName;

    private AgentHelper(String targetClassName) {
        this.targetClassName = targetClassName;
    }

    public static AgentHelper create(String targetClassName){
        AgentHelper agentHelper = new AgentHelper(targetClassName);
        return agentHelper;
    }

    public void proxy(String args, Instrumentation instrumentation){
        Class targetClass = obtainTargetClass(instrumentation);

        try {
            instrumentation.addTransformer(new SimpleTransformer(targetClassName), true);
            instrumentation.retransformClasses(targetClass); //
        } catch (Exception e) {
            System.out.println(String.format(">>> agentmain failure, error: %s: %s", e.getClass().getName(),e.getLocalizedMessage()));
            e.printStackTrace();
        }
    }

    private Class obtainTargetClass(Instrumentation instrumentation) {
        Class targetClass = null;
        for (Class loadedClass : instrumentation.getAllLoadedClasses()) {
            if(targetClassName.equals(loadedClass.getName())){
                targetClass = loadedClass;
            }
        }

        if(targetClass == null){
            try {
                // 无法加载
                targetClass = Class.forName(targetClassName);
            } catch (ClassNotFoundException e) {
                System.out.println(String.format(">>> Class [%s] not found", targetClassName));
            }
        }
        return targetClass;
    }

    public static class SimpleTransformer implements ClassFileTransformer {

        private String targetClassName;

        public SimpleTransformer(String targetClassName) {
            this.targetClassName = targetClassName;
        }

        @Override
        public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                                ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
            if(!className.equals(targetClassName.replaceAll("\\.","/"))){
                return null;
            }

            ClassPool classPool = ClassPool.getDefault();
            System.out.println(String.format("+++++ 代理类名：%s", className));
            try {
                CtClass ctClass = classPool.get(className.replace("/","."));
                CtMethod[] ctMethods = ctClass.getDeclaredMethods();
                for (CtMethod ctMethod : ctMethods) { // 所有类方法
                    ctMethod.insertBefore(String.format("{System.out.println(\">>> 开始执行方法：%s\");}",ctMethod.getName()));
                    ctMethod.insertAfter(String.format("{System.out.println(\">>> 结束执行方法：%s\");}",ctMethod.getName()));
                }
                return ctClass.toBytecode();
            } catch (NotFoundException | CannotCompileException | IOException e) {
                System.out.println(String.format("+++++ 代理出错：%s",e.getMessage()));
                e.printStackTrace();
            }
            return classfileBuffer;
        }
    }
}
```
---

> 通过上面的例子可以看到，两种方式的比对如下：

| 对比                    | JavaAgent       | AttachAgent   |
|-----------------------|-----------------|---------------|
| /META-INF/MANIFEST.MF | Premain-Class   | Agent-Class   |
| 代理类方法名称               | premain         | attachmain    |
| 代理入口                  | VM配置：-javaagent | JVM attach进程ID |
| 代理时机                  | JVM加载字节码时       | 程序运行时         |
| 作用                    | Java桌面程序        | Web应用         |

### 原理

代理可以发送在编译时，类加载时或者是运行时。

这里你要清楚，*java程序的入口是main方法*，不管是普通程序（比如桌面应用、可执行jar）或是Web应用（在Web容器中运行的基于Servlet的应用）

以javaagent为例，是在执行main方法前对已经加载到JVM的类进行修改，从而实现对目标类的代理，这里的修改是在字节码层面的，当然我们可以基于ASM工具库来实现，但是门槛太高。

基于Instrumentation可以与编写java代码一样，实现修改字节码来

ClassPool：保存CtClass的池子，通过classPool.get(类全路径名)来获取CtClass
CtClass：编译时类信息，它是一个class文件在代码中的抽象表现形式
CtMethod：对应类中的方法
CtField：对应类中的属性、变量

## XMind

*还记得XMind8的破解之法吗？*

是不是需要在XMind.ini文件中插入这样一段：-javaagent:.../XMindCrack.jar
要是你打开这个jar，你会看到这样的内容：
![xmind](/assets/images/Principle/AOP/xmind.png)

首先你需要知道其原理，是通过/plugins/net.xmind.verify.jar中提供的方法LicenseVerifier#doCheckLicenseKeyBlacklisted来进行身份校验

我们是不是只用修改License的校验方法*doCheckLicenseKeyBlacklisted*，忽略其校验过程并直接返回true就完事了？当然截图中就是这样做的，如果你想看懂那几行代码，可能你先要去学习ASM相关的知识。
```
InsnList insnList = methodNode.instructions;
insnList.clear();
insnList.add((AbstractInsnNode)new InsnNode(4));
insnList.add((AbstractInsnNode)new InsnNode(172));
methodNode.exceptions.clear();
methodNode.visitEnd();
```
以上代码其实就是讲方法体清除，并写入“return true”

### 结束语

通过示例了解javaassit如何实现代对目标类的代理。是不是觉得java应用程序都能被修改，那不是太不安全了？所以，你觉得呢...
