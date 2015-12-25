# Build System
---

## Configuring Gradle Builds

### Build Configuration Basics

1. AS项目顶层及每个模块各自有一个编译文件，文件名为build.gradle；
2. build.gradle都是普通的text文件，使用Groovy语法；
3. 大部分情况下，仅需要编辑模块级的编译文件;
4. 引入`android {...}`元素及其编译选项：
```groovy
apply plugin: 'com.android.application'
```
	`android {...}`用于配置所有Android相关的编译选项:
	- `compileSdkVersion`指定编译目标（compilation target）；
	- `buildToolsVersion`指定使用哪个版本的build tools；
	> 注：总是使用主版本号等于或高于编译目标（compilation target）和目标SDK（target SDK）的build tools。

The defaultConfig element configures core settings and entries in the manifest file (AndroidManifest.xml) dynamically from the build system. The values in defaultConfig override those in the manifest file.
The configuration specified in the defaultConfig element applies to all build variants, unless the configuration for a build variant overrides some of these values.
The buildTypes element controls how to build and package your app. By default, the build system defines two build types: debug and release. The debug build type includes debugging symbols and is signed with the debug key. The release build type is not signed by default. In this example the build file configures the release version to use ProGuard.
The dependencies element is outside and after the android element. This element declares the dependencies for this module. Dependencies are covered in the following sections.

Note: When you make changes to the build files in your project, Android Studio requires a project sync to import the build configuration changes. Click Sync Now on the yellow notification bar that appears for Android Studio to import the changes.
