---
title: Android_NDK开发之build.gradle
date: 2018-01-18 11:28:17
tags:
---

Author JOY.
<!-- excerpt -->

``` java
android {
  sourceSets {
    main {
       //通过设置 jni 目录为空，我们可不使用 apk 插件的 jni 编译功能。在jni目录下编写Android.mk即可
      jniLibs.srcDir 'src/main/libs'
      jni.srcDirs = [] // 设置使用自己编写的Android.mk
    }
  }
}
tasks.withType(JavaCompile) {
    compileTask -> compileTask.dependsOn ndkBuild
}
String getNdkBuildPath() {
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    def ndkBuildingDir = properties.getProperty("ndk.dir")
    def ndkBuildPath = ndkBuildingDir
    if (Os.isFamily(Os.FAMILY_WINDOWS)) {
        ndkBuildPath = ndkBuildingDir + '/ndk-build.cmd'
    } else {
        ndkBuildPath = ndkBuildingDir + '/ndk-build'
    }
    return ndkBuildPath
}

task ndkBuild(type: Exec, description: 'Compile JNI source via NDK') {
    println('executing ndkBuild')
    def ndkBuildPath = getNdkBuildPath();
    commandLine ndkBuildPath, '-j8', '-C', file('src/main').absolutePath
}

task ndkClean(type: Exec, description: 'clean JNI libraries') {
    println('executing ndkBuild clean')
    def ndkBuildPath = getNdkBuildPath();
    commandLine ndkBuildPath, 'clean', '-C', file('src/main').absolutePath
}

clean.dependsOn 'ndkClean'

```
