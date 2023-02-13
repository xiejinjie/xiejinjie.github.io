---
layout: post
title: "Java打包可执行Jar"
date: 2023-02-10 10:10:00 +0800
categories: tech
---

## 简介
当Java项目开发完成后，要在实际环境运行，就需要将代码打包为可执行Jar。对于不同类型的Java项目，需要做不同的配置和打包操作。这边介绍三种类型项目配置打包可执行Jar的方式：普通Java项目、普通Maven项目、Spring项目。

## 基础知识
- Jar文件
  Jar文件，即Java归档文件，是使用Zip压缩格式进行压缩的压缩包，所以Jar文件可以直接使用压缩工具打开的。
可以使用Jar命令制作Jar文件, 常见命令格式 
```jar cvf JARFileName File1 File2...```

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/07acf299108271c33f21af49c10a3fa.png)

- 可执行Jar文件
  可以在jar命令参数中添加e选项，指定程序的入口点， 即程序主类，创建可执行Jar。
  ```jar cvfe MyProgram.jar com.mycompany.mypkg.MainAppClass files to add```
  
  可执行Jar可以通过下面命令直接启动程序。
  ```java -jar MyProgram.jar```

- 清单文件
  清单文件 MANIFEST.MF，用于描述Jar文件特征，位于JAR文件META-INF子目录中。创建的可执行Jar文件实际就是在原始Jar的清单文件中添加了Main-Classd的配置。


http://docs.orade.com/javase/8/docs/technotes/guides/jar

## 普通Java项目
IDEA配置
1. 在项目配置中选择 Artifacts -> JAR -> From modules with dependencies
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/edb214499998e716002aff9a8100ddf.png)

2. 选择项目模块，程序主类、依赖引入方式、清单文件位置
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/d2e477e9838e9ab7558548f52a3cdb6.png)

3. 确认Jar名称和Jar输出目录
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/93a2068ecc4dd98f77b9b149fa04bb5.png)

4. 通过 Build -> Build Artifact -> Build 打包Jar文件
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/f5eef04cd6be040d24568194bb64478.png)

## 普通Maven项目


## Spring项目


## 参考资料
- Java核心技术·卷一·十三章 部署Java应用程序 
