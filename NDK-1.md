---
title: NDK开发
categories:
  - Android
  - NDK
tags:
  - NDK
date: 2020-05-15 00:20:25
---

NDK : Native Development Kit

**需要使用NDK的场景：**

- 进一步提升设备性能，以降低延迟，或运行计算密集型应用，如游戏或物理模拟。  
- 重复使用您自己或其他开发者的 C 或 C++ 库。  
- 在平台之间移植其应用。

Android Studio 编译原生库的默认构建工具是 [CMake](https://cmake.org/)。由于很多现有项目都使用 ndk-build 编译工具包，因此 Android Studio 也支持 [ndk-build](https://developer.android.google.cn/ndk/guides/ndk-build?hl=zh_cn)。不过，如果您要创建新的原生库，则应使用 CMake。

**要为您的应用编译和调试原生代码，您需要以下组件：**

- Android 原生开发套件 (NDK)：这套工具使您能在 Android 应用中使用 C 和 C++ 代码。  
- CMake：一款外部编译工具，可与 Gradle 搭配使用来编译原生库。如果您只计划使用 ndk-build，则不需要此组件。  
- LLDB：Android Studio 用于调试原生代码的调试程序。  
  如需了解如何安装这些组件，请参阅[安装及配置 NDK 和 CMake](https://developer.android.google.cn/studio/projects/install-ndk?hl=zh_cn)。  

**Tip:**
在Android studio 中使用记得在setting的Plugins中勾上Android SDK Support，否则不能直接new C++文件  

**构建NDK的主要组件：**

- 原生共享库：NDK 从 C/C++ 源代码编译这些库或 `.so` 文件。
- 原生静态库：NDK 也可编译静态库或 `.a` 文件，而您可将静态库关联到其他库。
- Java 原生接口 (JNI)：JNI 是 Java 和 C++ 组件用以互相通信的接口。本指南假设您具备 JNI 知识；如需了解相关信息，请查阅 [Java 原生接口规范](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)。
- 应用二进制接口 (ABI)：ABI 可以非常精确地定义应用的机器代码在运行时应该如何与系统交互。NDK 根据这些定义编译 `.so` 文件。不同的 ABI 对应不同的架构：NDK 为 32 位 ARM、AArch64、x86 及 x86-64 提供 ABI 支持。有关详情，请参阅 [ABI 管理](https://developer.android.google.cn/ndk/guides/abis?hl=zh_cn)。
- 清单：如果您编写的应用不包含 Java 组件，则必须在[清单](https://developer.android.google.cn/guide/topics/manifest/manifest-intro?hl=zh_cn)中声明 `NativeActivity` 类。[原生 Activity 和应用](https://developer.android.google.cn/ndk/guides/concepts?hl=zh_cn#naa)的“使用 `native_activity.h` 接口”部分进一步详细介绍了如何执行此操作。