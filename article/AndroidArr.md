(1）将编译好的 aar 文件放到 app/libs 下，类似 jar 的做法。譬如，libs/HDDecodeLibrary_1.1.03_a.aar。

(2）在工程的当前 module 的 gradle（app/build.gradle）的 dependencies 节点前添加：

```groovy
repositories {    flatDir {dirs'libs'}}
```

并且在 dependencies 中添加：

```groovy
compile(name: ‘HDDecodeLibrary_1.1.03_a’, ext: ‘aar’)
```

一个完整的 build.gradle 例子如下：

```groovy
apply plugin: 'com.android.application'
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "sim.com.scannertest"
        minSdkVersion 24
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabledfalse
            proguardFiles getDefault
            ProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets {
        main {
            res.srcDirs = ['src/main/res', 'src/main/res/raw']
        }
    }
}
repositories {
    flatDir { dirs 'libs' }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2',
            {
                exclude group: 'com.android.support', module: 'support-annotations'
            })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
    compile(name: 'HDDecodeLibrary_1.1.03_a', ext: 'aar')
}
```

本文来自 洌冰 的 CSDN 博客 ，全文地址请点击：https://blog.csdn.net/u011109881/article/details/78544147?utm_source=copy
