---
title: Android之gradle
date: 2018-02-01 17:54:00
tags:
---

Author JOY.
<!-- excerpt -->

构建规则语言：Gradle 选择了 Groovy，Groovy 基于 Java 并拓展了 Java。 Java 程序员可以无缝切换到使用 Groovy 开发程序。Groovy 说白了就是把写 Java 程序变得像写脚本一样简单。写完就可以执行，Groovy 内部会将其编译成 Java class 然后启动虚拟机来执行。

### Gradle基础知识：
* Groovy，由于它基于 Java，所以我们仅介绍 Java 之外的东西。了解 Groovy 语言是掌握 Gradle 的基础。
* Gradle 作为一个工具，它的行话和它“为人处事”的原则。

Gradle中，每一个待编译的工程都叫一个Project。每一个 Project 在构建的时候都包含一系列的Task。比如一个 Android APK 的编译可能包含：Java 源码编译Task、资源编译Task、JNI编译Task、lint检查Task、打包生成APK的Task、签名 Task 等。
