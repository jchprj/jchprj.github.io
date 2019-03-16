---
layout: post
title:  "Test grpc-java And Maven Modules In VSCode"
date:   2019-02-16 23:55:30 +0800
categories: java grpc maven vscode
---

Recently in the high frequency stress testing, two issues appeared. 

The server is distributed and connected through GRPC, using grpc-java library which is based on Netty.

First abnormal is about memory, shown below:

![JVM exporter](/assets/bugs/grpc-java-jvm-exporter.png)

In the top left graph, it is Memory area[heap]. Because the test is constantly high frequency, so the memory would recover in a certain interval. But every time when the memory decrease, the base memory is increasing, this results in unstable recycling while time goes on. 

Below is the Linux system information exported by node exporter:

![JVM exporter](/assets/bugs/grpc-java-linux-node-exporter.png)

The CPU usage suddenly disappeared due to all client connections lost. First suspicion was about memory leak caused server internal connection problem.

In order to find what occupy the memory, dumped memory from JVM and analysis the memory shown below:

![JVM dump](/assets/bugs/grpc-java-jvm-dump.png)

Large memory consumed by `io.netty.util.concurrent.ScheduledFutureTask`. To solve this, tried set more small keep alive time.

But there is no effect. Same result produced. After checking the log, there is a strange log:

```
io.grpc.StatusRuntimeException: RESOURCE_EXHAUSTED: Bandwidth exhausted
HTTP/2 error code: ENHANCE_YOUR_CALM
Received Goaway
too_many_pings
```

This is the cause of server internal connection lost. 

So I created a sample GRPC project to produce the log. 

[https://github.com/jchprj/test-grpc-java](https://github.com/jchprj/test-grpc-java)

After a lot of test and try, the problem was solved by upgrading the version of grpc-java. 

The test details is illustrated in the README of the test project. 

## Maven modules in VS Code

This is another small topic, just put it here.

I like using VS Code, now it supports Java well. But while creating the test project, there were some problems in Maven project.

VSCode will generate .classpath besides the pom.xml. But sometimes it won't do it when it is a sub module project and does not shown in the JAVA DEPENDENCIES.

After some test I submitted a sample project to test the Maven modules and finally find a way to solve it, just do `Java: Clean the Java language server workspace`.

The Maven module test project is here with illustrations: [https://github.com/jchprj/test-vscode-maven](https://github.com/jchprj/test-vscode-maven)

It seems like a bug in the VS Code extension.