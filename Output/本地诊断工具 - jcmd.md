---
type: permanent
aliases:
  - jcmd
banner: Assets/Banner/pexels-eberhardgross-1624496.jpg
---
# 🌐 核心观点

jcmd 是 JDK 8 推出的一款本地诊断工具, 只支持连接本机上同一个用户空间的 JVM 进程.

---

# 🔖 详细解释

jcmd 是 JDK 8 推出的一款**本地**诊断工具, **只支持**连接本机上**同一个用户空间**的 JVM 进程.

## 1	查看帮助

```bash
# jcmd -help
Usage: jcmd <pid | main class> <command ...|PerfCounter.print|-f file>
   or: jcmd -l                                                   
   or: jcmd -h                                                   

  command 必须是指定 JVM 可用的有效 jcmd 命令。     
  可以使用 "help" 命令查看该 JVM 支持哪些命令。  
  如果指定 pid 部分的值为 0，则会将 commands 发送给所有可见的 Java 进程。  
  指定 main class 则用来匹配启动类。可以部分匹配。（适用同一个类启动多实例）。                       
  If no options are given, lists Java processes (same as -p).    

  PerfCounter.print 命令可以展示该进程暴露的各种计数器
  -f  从文件读取可执行命令                 
  -l  列出（list）本机上可见的 JVM 进程                    
  -h  this help                          
```

## 2	查看进程信息

```bash
jcmd
jcmd -l
jps -lm

11155 org.jetbrains.idea.maven.server.RemoteMavenServer
```

上面的几个命令的结果类似, 查看本机的进程信息.

## 3	查看 JVM 支持的命令

```bash
jcmd 11155 help
jcmd RemoteMavenServer help

# ====== 输出信息 ======
11155:
The following commands are available:
VM.native_memory
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help
```



---

# 📚 参考内容

