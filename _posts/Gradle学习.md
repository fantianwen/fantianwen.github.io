title: Gradle学习
date: 2016-01-07 19:13:03
tags: [gradle,android]
---

## 一、Hack one:
### 将android studio中Library生成的aar文件上传到本地建立的（maven）仓库中，方便今后的库文件的管理和调用
### solution:
#### 1、将一个module成为一个android第三方库文件：
1>、在app这个module下面的build.gradle文件中：	

```xml
apply plugin: 'com.android.application'
```
修改为：

```xml
apply plugin: 'com.android.library'
```
2>、在

```xml
android {
			.....
    lintOptions{
        abortOnError false
    }
}
```
中这样添加（不加的话报错会提示）

3>在mac下的Terminal中，运行

```bash
sudo ./gradlew build
```
成功之后，在build/libs会有打包好的aar文件

<!-- more -->

#### 2、将aar文件打包至指定的本地maven仓库中
1>、在build.gradle文件中添加：

```xml
apply plugin: 'maven'
.....
.....
ext {
    PUBLISH_GROUP_ID = 'com.fantianwen'
    PUBLISH_ARTIFACT_ID = 'keyboard'
    PUBLISH_VERSION = android.defaultConfig.versionName
}

uploadArchives {
    repositories.mavenDeployer {
        def deployPath = file(getProperty('aar.deployPath'))
        repository(url: "file:///${deployPath.absolutePath}")
        pom.project {
            groupId project.PUBLISH_GROUP_ID
            artifactId project.PUBLISH_ARTIFACT_ID
            version project.PUBLISH_VERSION
        }
    }
}
```
其中：		
ext可以定义动态属性。		
uploadArchives定义上传aar文件的一个任务。

2>、使用命令：

```bash
sudo ./gradlew uploadArchives
```
这样在本地仓库就有了。

## 二、Hack two
### 将打包的apk文件放置在工程某文件夹下，或者放着在多个地方，或者上传至远程仓库
### Solution:
```xml
uploadArchives {
    repositories {
        flatDir {
            dirs 'file:///Users/RadAsm/gradle-study/0625'
        }
    }
}
```
将打包的文件放置在`/Users/RadAsm/gradle-study/0625`这个文件夹下面

## 三、Hack three
### 如何使用maven或者ivy的仓库
### solution：
```xml
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

```xml
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```
或者使用maven的本地仓库

```xml
repositories {
    maven {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```


## 四、Hack four
### 使用gradle打包`android library`并上传至`jcenter`全球maven库

### solution
在project根目录下面的build.gradle的`dependencies`节点下：

```gradle
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.0'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
```

library的build.gradle文件中：

```gradle
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

version = "1.3"

def siteUrl = 'https://github.com/fantianwen/CommonDHJT'    // project homepage
def gitUrl = 'https://github.com/fantianwen/CommonDHJT.git' // project git

group = "com.chinaums.dhjt"

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ["libs"]
        }

    }
    lintOptions{
        abortOnError false
    }


}



install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                name 'dhjt-sdk-release'
                url siteUrl
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'fantianwen'
                        name 'tianwen.fan'
                        email 'twfan_09@hotmail.com'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives javadocJar
    archives sourcesJar
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.0.1'
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = "dhjt-sdk-release"                // project name in jcenter
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

同时你需要在工程根目录下面的`local.properties`中书写：

```gradle
bintray.user=fantianwen
bintray.apikey=1bd280897a1734ec2b6f2963bca71cf0f1df25c0
```

然后依次执行`gradle`命令

```bash

bash gradlew javadocJar

bash gradlew sourcesJar

bash gradlew install

bash gradlew bintrayUpload

```

这样library就上传到了jcenter中，但是一个**新的**jcenter中的文件目录（library）需要进行提交审核。然后提交审核一下就OK了，一般从提交到审核完毕需要8小时左右。

>**Note**：注意你的library的名称，就是在jcenter中的文件名称了，这个会区分文件目录，新的文件目录需要提交审核！

#### 如何在build.gradle中进行library的引用

上传完毕之后，会在`>>library根目录>>build>>poms`下面生成`pom-default`文件，该文件中有诸如下面的说明：

```xml
  <groupId>com.chinaums.dhjt</groupId>
  <artifactId>dhjt-sdk-release</artifactId>
  <version>1.3</version>
  <packaging>aar</packaging>
  <name>dhjt-sdk-release</name>
```

然后，引用的时候，按照格式：

```gradle
compile 'groupId:artifactId:version'
```

进行引用

#### 注意点：
只有不同的library的名称才需要重新提交，已经审核通过的，但是不同版本（经过迭代，提高版本）只要upload上去，不再需要重新审核了。


## 五、Hack 5

### 多渠道打包

1、多个module同时需要使用一样的变量配置（共享gradle文件）

在rootProject的build.gradle中声明该文件即可

```gradle
apply from: 'buildsystem/dependencies.gradle'
```

其中，`denpendencies.gradle`中如此定义变量

```gradle
ext {
    ANDROID_BUILD_MIN_SDK_VERSION = '8'
    ANDROID_COMPILE_SDK_VERSION = '19'
    ANDROID_TARGET_SDK_VERSION = '15'
    ANDROID_BUILD_TOOLS_VERSION = '23.0.2'
    VERSION_NAME = 'v2.0.2'
    VERSION_CODE = '180'
}
```

然后，在各个模块的gradle文件中，使用

```java
def globalConfiguration = rootProject.extensions.getByName('ext')
```

就行获取到全局变量

2、android{...}

>defaultConfig

所有定义在defaultConfig中的变量，如果在AndroidManifest.xml中有定义，将会对AndroidManifest.xml文件中的变量进行覆写

>productFlavors{...}

定义各种不用的“渠道”

比方说定义了如下的渠道：

```java
pro{
}
air{
}
```

那么在`Build>Select Build Variant`中将会看见不同的打包的参数

这个打包的参数，在执行`./gradlew assemble`命令的时候会全部执行，从而在build下面生成不同版本的apk

>variantFilter{...}

对上面的buildVariant进行过滤，如果设置

```java
 variant.setIgnore(false)
```

则表示，在assemble中，会进行相应打包命令的忽略，会在Build Variant中看见相应的打包参数发生了变化

>各种不用的打包命令中对进行的处理

```java
applicationVariants.all{variant->...}
```

2、如果在不用的渠道打包过程中需要compile不同的文件

```java
dependencies {
    
    for (def i = 0; i < android.productFlavors.size(); i++) {
        if (android.productFlavors[i].name == 'dhjt_debug') {
            dhjt_debugCompile files('libs/mpos-plugin-sdk-3.5.0-debug.jar')
        }
        if (android.productFlavors[i].name == 'dhjt_release') {
            dhjt_releaseCompile files('libs/mpos-plugin-sdk-3.5.0.jar')
        }
    }

}
```
















