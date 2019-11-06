---
title: jar、aar 的生成与导入
date: 2019-01-27 21:45:23
categories: "Android"
tags:
    - java
    - jar
    - aar
---

本文将介绍 jar、aar 包的生成与导入。

<!-- more -->

# jar 包的生成

1. 创建一个普通项目My Application。
2. 新建module：菜单栏 File -> New -> New Module
3. 选择 Android Librar，并确认。
4. 菜单栏 File -> Project Structure
5. 将新建的 module 导入 app 中。

![将module导入app](/images/将module导入app.png)
6. 修改 module 项目的 build.gradle 文件，添加如下代码： (注意：与 buildTypes 平级)
    方式一：打包的 jar 包括 .class 文件及资源文件等
    ```java
    def _BASENAME = "TestJar"
    def _VERSION = "_V1.0"
    def _DestinationPath = "build" //生成jar包的位置
    def zipFile = file('build/intermediates/packaged-classes/release/classes.jar') //待打包文件位置

    task deleteBuild(type:Delete){
        delete _DestinationPath + _BASENAME + _VERSION + ".jar"
    }

    task makeJar(type:Jar){
        from zipTree(zipFile)
        from fileTree(dir:'src/main',includes:['assets/**']) //将assets目录打入jar包
        baseName = _BASENAME + _VERSION
        destinationDir = file(_DestinationPath)
    }

    makeJar.dependsOn(deleteBuild, build)
    ```

    方式二：打包的 jar 只有源代码的.class 文件，不包含资源文件
    ```java
    task makeJar(type: Copy) {
        delete 'build/TestJar_V1.0.jar' //删除之前的旧jar包
        from('build/intermediates/packaged-classes/release/') //从这个目录下取出默认jar包
        into('build/') //将jar包输出到指定目录下
        include('classes.jar')
        rename('classes.jar', 'TestJar_V1.0.jar') //自定义jar包的名字
    }
    makeJar.dependsOn(build)
    ```
7. 找到 Gradle 中的 makeJar 命令：Gradle 一般在 android studio 右侧，点击，选择 Module 中的 other，找到 makeJar，双击执行。
8. 编译完成后，在对应位置找到生成的 jar 包即可。

![生成jar包位置](/images/生成jar包位置.png)

# jar 包的导入

1. 将 jar 包拷贝至 app/libs/ 目录下
2. 右键 jar 包，选择 Add As Library 确认即可。

# aar 包的生成

1. 同 jar 包的生成步骤中 1-5，新建 module 并导入 app 中。
2. 找到 Gradle 中的 assembleRelease 命令：Gradle 一般在 android studio 右侧，点击，选择 Module 中的 build，找到 assembleRelease，双击执行。

![assembleRelease](/images/assembleRelease.png)
3. 编译成功后，可以在 module 下的 build/outputs/aar/ 目录下看到生成的 aar 文件

![aar生成位置](/images/aar生成位置.png)

# aar 包的导入

1. 将 jar 包拷贝至 app/libs/ 目录下
2. 修改 app 下的 build.gradle 文件，添加如下代码：

```java
// 与 buildTypes 同级
repositories {
    flatDir {
        dirs 'libs'
    }
}

// 位于 dependencies 下
implementation(name:'mylibrary-release', ext:'aar')
```

3. 重新编译即可