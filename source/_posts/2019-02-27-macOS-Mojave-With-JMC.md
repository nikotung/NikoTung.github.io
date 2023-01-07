---
title: "macOS Mojave 无法使用JMC(Java Mission Controller)"
description: ""
categories: "2019-02"
tags: [Java,JVM,JMC]
---

前两天想用JMC(Java Mission Controller)看看应用的内存使用和GC 情况，就在终端下敲下`jmc`,没想到启动后就只剩左上角的三个按钮，小到不注意看都没法发现。一开始以为是样式的问题，就尝试按下`option` 然后去点击全屏的➕按钮，竟然一点反应也没有，反应过来后，想必这又是一个坑了。

去网上一搜发现很多这样的问题。只能跟着看看有没啥解决办法，在[SO](https://stackoverflow.com/questions/48400346/java-mission-control-from-jdk-1-8-0-161-frozen-upon-startup-on-mac-os-x)上发现一个解决方案，就开始尝试去跟着改改。

1. 在[这里](https://search.maven.org/search?q=g:org.eclipse.platform%20AND%20a:org.eclipse.swt.cocoa.macosx.x86_64&core=gav)去下载一个jar包。
2. 找到JMC 所在的路径。比如`/Library/Java/JavaVirtualMachines/jdk1.8.0_192.jdk/Contents/Home/lib/missioncontrol`
3. 备份当前存在的jar 包。`sudo mv
plugins/org.eclipse.swt.cocoa.macosx.x86_64_3.103.1.v20140903-1947.jar
../`
4. 把下载好的jar 包放到plugins 目录下。`sudo cp ~/Downloads/org.eclipse.swt.cocoa.macosx.x86_64-3.105.2.jar plugins/org.eclipse.swt.cocoa.macosx.x86_64_3.103.1.v20140903-1947.jar`
5. 重启jmc

注意在第一步下载jar 包时，由于没有找到跟系统一样版本的，所以就下载了一个`org.eclipse.swt.cocoa.macosx.x86_64-3.109.0.jar` 3.109 版本的，但是没有用。最后下载了一个`3.105.2`版本的才能使用。

 #### 连接到本地应用

 在启动应用时，需要配置如下几个jvm 参数：

    -Dcom.sun.management.jmxremote.rmi.port=8192 
    -Dcom.sun.management.jmxremote=true 
    -Dcom.sun.management.jmxremote.port=8192 
    -Dcom.sun.management.jmxremote.ssl=false 
    -Dcom.sun.management.jmxremote.authenticate=false  
    -Djava.rmi.server.hostname=localhost

其中`java.rmi.server.hostname` 为你应用所在机器的ip地址，如果是本地直接`localhost`就可以。

![](/assets/2019-02-27-jmc.png)

#### Reference
[https://stackoverflow.com/questions/48400346/java-mission-control-from-jdk-1-8-0-161-frozen-upon-startup-on-mac-os-x](https://stackoverflow.com/questions/48400346/java-mission-control-from-jdk-1-8-0-161-frozen-upon-startup-on-mac-os-x)

[https://blog.overops.com/oracle-java-mission-control-the-ultimate-guide](https://blog.overops.com/oracle-java-mission-control-the-ultimate-guide/)

[https://www.youtube.com/watch?v=WMEpRUgp9Y4](https://www.youtube.com/watch?v=WMEpRUgp9Y4)