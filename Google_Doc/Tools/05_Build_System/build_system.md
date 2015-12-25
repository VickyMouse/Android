# Build System
## Configuring Gradle Builds
### Build Configuration Basics
1. AS项目顶层及每个模块各自有一个编译文件，文件名为build.gradle；
2. build.gradle都是普通的text文件，使用Groovy语法；
3. 大部分情况下，仅需要编辑模块级的编译文件;
4. 引入`android {...}`元素用于配置所有Android相关的编译选项：
   ```groovy
   apply plugin: 'com.android.application'
   ```

   - `compileSdkVersion`指定编译目标（compilation target）；

   - `buildToolsVersion`指定使用哪个版本的build tools；
   > 注：总是使用主版本号等于或高于编译目标（compilation target）和目标SDK（target SDK）的build tools。  

   - `defaultConfig`用于动态配置AndroidManifest.xml文件中的主要设置项。`defaultConfig`中的值会覆盖manifest文件中的。`defaultConfig`中指定的配置会被应用到所有build variants上，除非build variant有配置覆盖的值。

   - `buildTypes`控制如何编译并打包app。编译系统默认定义两种build types：debug和release。

5. `dependencies`在`android`元素的外面，并跟在它后面。它定义了模块的依赖。

---
### Declare dependencies
