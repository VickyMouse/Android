# 01 Configuring Gradle Builds
## Build Configuration Basics
```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:19.0.1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
1. AS项目顶层及每个模块各自有一个编译文件，文件名为build.gradle；
2. build.gradle都是普通的text文件，使用Groovy语法；
3. 大部分情况下，仅需要编辑模块级的编译文件;
4. 引入`android {...}`元素用于配置所有Android相关的编译选项：
   ```groovy
   apply plugin: 'com.android.application'
   ```

5. - `compileSdkVersion`指定编译目标（compilation target）；

   - `buildToolsVersion`指定使用哪个版本的build tools；
   > 注：总是使用主版本号等于或高于编译目标（compilation target）和目标SDK（target SDK）的build tools。  

   - `defaultConfig`用于动态配置AndroidManifest.xml文件中的主要设置项。`defaultConfig`中的值会覆盖manifest文件中的。`defaultConfig`中指定的配置会被应用到所有build variants上，除非build variant有配置覆盖的值。

   - `buildTypes`控制如何编译并打包app。编译系统默认定义两种build types：debug和release。

6. `dependencies`在`android`元素的外面，并跟在它后面。它定义了模块的依赖。

### Declare dependencies
编译系统会将所有的`compile`依赖添加到编译的classpath下，并将它们包含到最终的包里。
```groovy
dependencies {
    // Module dependency
    compile project(":lib")

    // Remote binary dependency
    compile 'com.android.support:appcompat-v7:19.0.1'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
1. Module依赖

	`compile project(":lib")`：`app`模块依赖于`lib`模块，编译系统编译`app`模块时，会组装并包含`lib`模块。

2. 远程binary依赖

	`compile 'com.android.support:appcompat-v7:19.0.1'`：通过指定库的Maven仓库地址来声明依赖。AS默认项目使用Maven的中心仓库（此属性在项目顶层的build.gradle文件中配置）。

3. 本地binary依赖

	如果模块对本地binary有依赖的话，将依赖的JAR文件复制到项目的`<moduleName>/libs`下。

	`compile fileTree(dir: 'libs', include: ['*.jar'])`：告诉编译系统添加`app/libs`下的所有JAR文件到编译的classpath下，并将它们包含到最终的包里。

### Run ProGuard
`getDefaultProguardFile('proguard-android.txt')`：从安装的Android SDK中获取默认的ProGuard配置。

AS将模块的规则文件`proguard-rules.pro`放置在模块的根目录，可以在其中自定义ProGuard规则。

### Application ID for package identification
1. build.gradle文件中`android`部分中的`applicationId`唯一确定了应用发布的包名。
   > 注：`applicationId`仅在build.gradle文件而不是AndroidManifest.xml文件中指定。

   使用build系统的build variants，可以为每个`product flavors`和`build types`定义不同的包名。`build type`的application ID是作为`product flavors`指定的值的后缀来指定的。

   ```groovy
   productFlavors {
        pro {
            applicationId = "com.example.my.pkg.pro"
        }
        free {
            applicationId = "com.example.my.pkg.free"
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }
	}		
   ```
2. `package name`必须在manifest文件中指定。它在代码中用于指向R类或决定任意相关activity/service的注册的。

   > 注：如果有多个manifests文件（如：`product flavor`和`build type`有各自的manifest文件），这些manifests文件可以不指定`package name`。如果指定了，则必须同`src/main/`目录下的manifest文件指定的`package name`相一致。

### Configure signing settings

`debug`和`release`版本的差别在于：应用是否可以在secure devices上debug，以及APK是如何签名的。编译系统用一个默认的Key给debug版签名，并使用已知身份的证书来避免编译时弹出密码输入对话框。除非显示定义签名配置，否则编译系统不会为release版签名。

---
## Work with build variants
此部分讨论编译系统如何帮助你用同一个项目来创建应用的不同版本，如：App的Demo版和付费版；Google Play上针对不同设备配置发布不同APK。

1. 编译系统使用`product flavors`来创建app的不同版本，每个版本可以有不同的功能和设备配置。
2. 编译系统也使用`build types`来设置不同版本的编译和打包参数。
3. 每一个`product flavor`和`build type`组合算作一个`build variant`。编译系统为每一个`build variant`创建一个APK。

### Build variants
#### Product flavors
要创建app的不同版本：
1. 在编译文件中定义`product flavors`；
2. 为每个flavor创建另外的source目录；
3. 为项目增加flavor相关的代码。

Example：`BuildSystemExample`应用有两个flavor，Demo版以及完整版。两个flavor共用MainActivity，上面有一个点击后会启动SecondActivity的按钮。不同flavor新打开的SecondActivity并不相同，即完整版中的SecondActivity比Demo版中的有更多功能。最后，每一个flavor会生成一个APK。

##### Define product flavors in the build file

```groovy
android {
    ...
    defaultConfig { ... }
    signingConfigs { ... }
    buildTypes { ... }
    productFlavors {
        demo {
            applicationId "com.buildsystemexample.app.demo"
            versionName "1.0-demo"
        }
        full {
            applicationId "com.buildsystemexample.app.full"
            versionName "1.0-full"
        }
    }
}
```
`product flavor`支持同defaultConfig`元素一样的属性。所有flavor的基本配置在`defaultConfig`中设置，不同的`flavor`覆写默认值。
例子中的编译文件为不同的flavor指定了不同的包名。

##### Add additional source directories for each flavor
接下来为每个flavor创建不同的目录，如为Demo flavor创建代码目录：
```
app/src/demo/java
app/src/demo/res
app/src/demo/res/layout
app/src/demo/res/values
```

##### Add a new activity to each flavor
为Demo flavor添加SecondActivity及相应文件：
- 添加`SecondActivity.java`到`app/src/demo/java`的`com.buildsystemexample.app`包名目录下；
- 添加`activity_second.xml`到`app/src/demo/res/layout`目录下；
- 添加`strings.xml`到`app/src/demo/res`目录下。

为完整版也添加相应的目录及文件。
> 注：自此即可以为不同的flavor独自开发相应SecondActivity，为完整版添加更多的功能。

##### Launch a flavor-specific activity from the main activity
由于不同flavor的SecondActivity有相同的包名和类名，所以可以从共用的MainActivity启动它。

#### Build types
`Build types`代表了每个app包的编译打包版本，默认提供`debug`和`release`两种。
```groovy
android {
    ...
    defaultConfig { ... }
    signingConfigs { ... }
    productFlavors {...}
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
         debug {
            debuggable true
        }
    }
}
```
> 注：尽管只有release`build type`出现在默认的build.gradle文件中，每次编译，release和debug的`build types`都有被应用。

此例中，`product flavors`和`build types`创建了如下`build variants`：
```
demoDebug
demoRelease
fullDebug
fullRelease
```
> 注：`Build > Make Project`编译整个项目中从上次编译后修改过的的所有代码文件；`Build > Rebuild Project`重新编译项目中所有代码文件。

会为每个`build variant`创建单独的输出目录。
