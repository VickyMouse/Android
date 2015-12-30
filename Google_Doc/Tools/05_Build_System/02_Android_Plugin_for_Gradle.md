# 02 Android Plugin for Gradle
> Android编译系统由Gradle的Android插件构成，它独立于AS，因此也可以使用命令行来编译，输出结果都是相同的。

> Gradle是一个高级的编译工具，用于管理依赖、自定义编译逻辑。它使用Domain Specific Language(DSL)，Groovy语法。

## Build Configuration
项目的编译配置定义在`build.gradle`文件中，涵盖了编译的如下几方面：
`Build variants`，`Dependencies`，`Manifest entries`，`Signing`，`ProGuard`，`Testing`，

---

## Build Convention
AS的编译系统有默认的`sensible defaults`，如果项目使用这些惯例，则Gradle的编译文件会很简洁；如果惯例不适用于你的项目，也可以配置编译过程的几乎每一方面。

---

## Project and Module Settings
一个AS项目包含项目文件及一个或多个模块（module）。
> `module`：app中可以独立编译、测试或调试的组件。`module`包含app的`source code`及`resources`。

AS项目可以有如下几种模块：
- `Android application modules`：包含应用代码，可能依赖于`library modules`。编译系统为它创建APK包。
- `Android library modules`：包含可复用的Android代码和资源。编译系统为它生成AAR（Android ARchive）包。
- `App Engine modules`：包含App Engine集成用的代码和资源。
- `Java library modules`：包含可复用的代码。编译系统为Java库模块生成JAR包。

### Project Build File
项目级的Gradle文件默认使用`buildscript`来定义Gradle仓库和依赖，从而允许不同的项目使用不同的Gradle版本。可用的仓库包括：`JCenter`，`Maven Central`，`Ivy`。下面的例子声明使用了`JCenter`仓库，以及对适用于Gradle1.0.1版本的Android插件的`classpath`依赖。

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.0.1'

        // NOTE: Do not place your application dependencies here: they belong
        // in the individual module build.gradle files
    }
}

allprojects {
   repositories {
       jcenter()
   }
}
```

> 注：AS项目的SDK目录通过`sdk.dir`下的`local.properties`文件来定义，或通过环境变量`ANDROID_HOME`配置。

### Module Build File
- `android settings`  
  - `compileSdkVersion`
  - `buildToolsVersion`


- `defaultConfig` & `productFlavors`
  - 覆盖manifest属性：`applicationId`，`minSdkVersion`，`targetSdkVersion`和测试信息。


- `buildTypes`
  - 编译属性：debuggable，ProGuard，debug signing，版本名字后缀和测试信息。


- `dependencies`

> 注：可以定义函数`function`，并通过调用函数来注入自定义的值到属性的编译逻辑中。如：

```groovy
def computeVersionName() {
  ...
}

android {
    defaultConfig {
        versionName computeVersionName()
        ...
    }
}
```

---

## Dependencies
- `Module Dependencies`：module
- `Local Dependencies`：本地binary archives，如：JAR文件。
- `Remote Dependencies`：远程仓库。AS支持`Maven`等远程仓库，以及`Ivy`等依赖管理。编译系统中使用的Maven坐标格式是：`group:name:version`

---

## Build Tasks
AS的编译系统定义了一级级往下的编译任务：顶层任务调起依赖的任务，并生成共同的编译输出。
顶层编译任务有：
- `assemble`：编译项目输出。

- `check`：运行`checks`和`tests`。Android插件为：连接的、模拟的或远程的设备的`checks`提供了`connectedCheck`和`deviceCheck`任务。

- `build`：运行`assemble`和`check`。

- `clean`：执行`clean`。

点击右边的“Gradle”tab即可查看Gradle任务。

---

## Gradle Wrapper
AS项目包含有如下组成的`Gradle wrapper`：
- 一个JAR文件
- 一个properties文件
- 一个Windows平台使用的shell脚本文件
- 一个Mac和Linux平台使用的shell脚本文件

> 注：应该提交上述所有的文件到代码控制系统上。

`Gradle wrapper`确保了总是使用`local.properties`文件中定义的Gradle版本，而不是本地安装的版本。既可以同时处理使用不同Gradle版本的多个项目了。

> 注：AS不使用脚本文件，在使用IDE编译时，脚本文件中的任何改变都不会起效，如要自定义编译逻辑，应该使用Gradle编译文件。

---

## Build Variants
多Android应用代码资源会被合成，他们的优先级由低到高是：`libraries`/`dependencies` -> `main src` -> `productFlavor` -> `buildType`。

有些项目会有更复杂的、超过一维的功能组合，如：除了有demo版和完整版，有些游戏会为特殊的CPU/ABI指定binaries，此时项目有2种`build types`（`debug`和`release`）以及2个维度的`product flavors`，一个是app type（`demo`或`full`），另一个是CPU/ABI（`x86`, `ARM`, or `MIPS`）。

### Source directories
编译各个版本的app时，编译系统从如下目录整合代码和资源：

- `src/main/`
- `src/<buildType>/`
- `src/<productFlavor>/`
> 注：`build type`和`product flavor`代码目录可以不存在，AS也不会自动创建它们。它们不存在的话，编译系统就不会使用它们。

对于定义了`product flavors`的项目，编译系统合并`build type`，`product flavor`及`main source`目录。

对于有多维`product flavors`的项目，编译系统每一维取一个flavor代码目录进行合并。如：对`arm-demo-release`的`build variant`，编译系统合并：
- `src/main/` (default configuration)
- `src/release/` (build type)
- `src/demo/` (flavor - app type维度)
- `src/arm/` (flavor - ABI维度)

只要不是用于同一个`build variant`，可以在不同的目录下有相同的类名。

编译系统也会将所有的manifests合并成一个manifest，每个`build variant`可以在最后的manifest文件中定义不同的组件和权限。manifest合并优先级从低到高是：`libraries`/`dependencies` -> `main src` -> `productFlavor` -> `buildType`。

编译系统合并所有代码目录下的所有资源`resources`。如果同一个`build variant`的不同目录下有相同名字的资源，优先级由高到低是：`build type` > `product flavor` > `main source` > `libraries`。
