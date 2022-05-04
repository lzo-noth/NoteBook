---
tags: java, quarkus, graalvm
---

## 什么是Quarkus
Quarkus 是一个为 Java 虚拟机（JVM）和原生编译而设计的全堆栈 Kubernetes 原生 Java 框架，用于专门针对容器优化 Java，并使其成为无服务器、云和 Kubernetes 环境的高效平台。

Quarkus 可与常用 Java 标准、框架和库协同工作，例如 Eclipse MicroProfile、Spring（作为 [2020 年红帽全球峰会追踪](https://tracks.redhat.com/c/red-hat-summit-virtu-11?x=QBJ6Wl&sc_cid=7013a000002DgCjAAK)的一个环节一起演示）、Apache Kafka、RESTEasy (JAX-RS)、Hibernate ORM (JPA)、Spring、Infinispan、Camel 等。

Quarkus 的依赖注入解决方案基于 CDI（上下文和依赖注入），且包含一个扩展框架来扩展功能并将其配置、引导并集成到您的应用中。添加[扩展](https://quarkus.io/extensions/)就像添加依赖项一样容易；或者，您可以使用 Quarkus 工具。

此外，它还向 GraalVM（一种通用虚拟机，用于运行以多种语言（包括 Java 和 JavaScript）编写的应用）提供正确信息，以便对应用进行原生编译
[官网](https://quarkus.io/guides/)
## 为什么使用Quarkus
## Quarkus环境搭建
#### 1. [graalvm](https://github.com/graalvm)安装
	- 下载 https://github.com/graalvm/graalvm-ce-builds/releases
	- 解压
	- 环境变量
		- GRAALVM_HOME=D:/graalvm/graalvm-ce-java11-22.1.0
		- JAVA_HOME=%GRAALVM_HOME%
		- Path 添加 %JAVA_HOME%/bin
	- 验证 
		- cmd.exe
			- java -version
			- openjdk version "11.0.15" 2022-04-19
			OpenJDK Runtime Environment GraalVM CE 22.1.0 (build 11.0.15+10-jvmci-22.1-b06)
			OpenJDK 64-Bit Server VM GraalVM CE 22.1.0 (build 11.0.15+10-jvmci-22.1-b06, mixed mode, sharing)

```
gu install native-image
```

#### 2. 安装C语言运行环境
- 安装# Visual Studio2019，语言包记得选英文
- 环境变量设置
	- INCLUDE=C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\shared;D:\MVS2019\VC\Tools\MSVC\14.29.30133\include;
	- LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\ucrt\x64;D:\MVS2019\VC\Tools\MSVC\14.29.30133\lib\x64;
	- Path=D:\MVS2019\VC\Tools\MSVC\14.29.30133\bin\Hostx64\x64
	- 根据安装目录调整

#### Maven
- 要求Apache Maven 3.8.1+

## get startted
[官方项目](https://github.com/quarkusio/quarkus-quickstarts.git)
clone到本地
- 构建本地可执行文件
```
mvn package -Pnative
```
- run application
```
mvn quarkus:dev
```

