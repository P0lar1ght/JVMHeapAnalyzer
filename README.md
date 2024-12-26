<h1 align="center">JVMHeapAnalyzer</h1>

`JVMHeapAnalyzer`简介：

`JVMHeapAnalyzer`是一款自动化的 Java 堆转储分析工具，支持多种操作系统和 Java 版本。旨在通过简单的Shell终端，自动生成堆转储文件并分析其中的敏感信息，包括但不限于`JDK`信息，`Env`信息，`ShiroKey`，存储`Cookie`，`authorization`，`oss`，数据库连接账号密码等等，辅助企业安全人员或合法授权的红队⚔️进行安全测试🔍️，进一步的信息搜集以获取更好的成果。

## ⚠️声明

此工具仅用于企业安全人员自查验证自身企业资产的安全风险，或有合法授权的安全测试，请勿用于其他用途，如有，后果自负。

## 🔧设计流程

```
获取所有java的pid（SystemInfo获取或者jps -q获取） -> 通过对应的pid生成HeapDump（agent模式和jmp，jcmd模式）-> 将对应生成的HeapDump文件丢给Analyzer分析对应的敏感信息 -> 输出分析结果。
```

## ⌨️ 命令行传参设计

```bash
usage: java -jar JVMHeapAnalyzer.jar
 -a,--agent <arg>   agent生成HeapDump的模式, 需要指定agent文件路径
 -c,--cmd <arg>     jmap或jcmd的JDK安装路径($JAVA_HOME/bin),
                    默认为空字符串(即使用当前环境变量中的jmap或jcmd)
 -f,--file <arg>    直接分析HeapDump文件, 需要指定HeapDump文件路径
 -h,--help          Show help
```

## 🤏使用

`cmd`模式仅适用于JDK环境

```shell
#使用默认环境变量jdk环境
java -jar JVMHeapAnalyzer.jar -c
java -jar JVMHeapAnalyzer.jar -cmd
## -c 后可指定jdk环境
java -jar JVMHeapAnalyzer.jar -c /usr/local/openjdk-8/bin
java -jar JVMHeapAnalyzer.jar -cmd /usr/local/openjdk-8/bin
```

`agent`模式适配`JRE`和`JDK`环境

```shell
#java8
java -jar JVMHeapAnalyzer.jar -a heapdump-agent.jar
java -jar JVMHeapAnalyzer.jar -agent heapdump-agent.jar
#java8以上
java --add-modules jdk.attach -jar JVMHeapAnalyzer.jar -a heapdump-agent.jar
java --add-modules jdk.attach -jar JVMHeapAnalyzer.jar -agent heapdump-agent.jar
```

`file`模式直接分析`HeapDump`文件

```shell
java -jar JVMHeapAnalyzer.jar -f heapdump
java -jar JVMHeapAnalyzer.jar -file heapdump
```

## 💡其他

### 🧐 HeapDumpAgent

`HeapDumpAgent.java`

```java
import java.lang.instrument.Instrumentation;
import java.lang.management.ManagementFactory;
import java.net.URLDecoder;
import com.sun.management.HotSpotDiagnosticMXBean;

public class HeapDumpAgent {

    private static final String HOTSPOT_BEAN_NAME = "com.sun.management:type=HotSpotDiagnostic";

    public static void premain(String agentArgs, Instrumentation inst) {
        dumpHeap(agentArgs);
    }

    public static void agentmain(String agentArgs, Instrumentation inst) {
        dumpHeap(agentArgs);
    }

    private static void dumpHeap(String fileName) {
        if (fileName == null || fileName.isEmpty()) {
            String pid = ManagementFactory.getRuntimeMXBean().getName().split("@")[0];
            fileName = "heapDump_" + pid + ".hprof";
        }
        try {
            HotSpotDiagnosticMXBean hotspotBean = ManagementFactory.newPlatformMXBeanProxy(
                    ManagementFactory.getPlatformMBeanServer(),
                    HOTSPOT_BEAN_NAME,
                    HotSpotDiagnosticMXBean.class
            );
            fileName = URLDecoder.decode(fileName, "UTF-8");
            //System.out.println("Dumping heap to " + fileName);
            hotspotBean.dumpHeap(fileName, true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

`MANIFEST.MF`

```java
Manifest-Version: 1.0
Agent-Class: HeapDumpAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

## 🤗参考和致谢

**⚠️仅支持个人研究学习，切勿用于非法犯罪活动。**

本项目的开发者、提供者和维护者不对使用者使用工具的行为和后果负责，工具的使用者应自行承担风险。

参考致谢：

   - https://github.com/whwlsfb/JDumpSpider 
