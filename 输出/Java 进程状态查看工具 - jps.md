---
type: permanent
aliases:
  - jps
---
# 🌐 核心观点

Java 提供的, 类似 ps (Process Status) 的命令, 用于展示 Java 进程信息.

---

# 🔖 详细解释

Java 提供的, 类似 ps (Process Status) 的命令, 用于展示 Java **进程信息**.

需要注意, jps 展示的是**当前用户可见**的 Java 进程, 如果看不见某些进程, 可能需要使用 sudo, su 之类的命令来**切换权限**.

```bash
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]
Definitions:
    <hostid>:      <hostname>[:<port>]
```

参数分成了**多个组**, 同一个组的参数可以共用一个 `-`, 最常用的参数是 `-v`, 用于显示传递给 JVM 的**启动参数**, 其他的参数的作用分别是:

- `-q`: 只显示进程号.
	
- `-m`: 显示传给 main 方法的参数信息
	
- `-l`: 显示启动 class 的完整类名, 或者启动 jar 的完整路径
	
- `<hostid>`: 远程主机的标识符, 需要远程主机启动 jstatd 服务器支持.

```bash
$ jps -v

15883 Jps -Dapplication.home=/usr/local/jdk1.8.0_74 -Xms8m
6446 Jstatd -Dapplication.home=/usr/local/jdk1.8.0_74 -Xms8m
        -Djava.security.policy=/etc/java/jstatd.all.policy
32383 Bootstrap -Xmx4096m -XX:+UseG1GC -verbose:gc
        -XX:+PrintGCDateStamps -XX:+PrintGCDetails
        -Xloggc:/xxx-tomcat/logs/gc.log
        -Dcatalina.base=/xxx-tomcat -Dcatalina.home=/data/tomcat
```

知道了 Java 进程的 pid 后, 就可以使用其他工具来进行诊断了.

---

# 📚 参考内容

