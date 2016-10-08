#gradle的学习
这周在发布插件时接触到gradle写的一个脚本不得不重新去了解一下gradle的机制。<br>
学习的博客地址：<br>
https://gradle.org/getting-started-gradle-java/#toggle-id-1<br>
http://www.cnblogs.com/davenkin/p/gradle-learning-3.html<br>
gradle的内核是grovvy。gradle中主有有两个概念task和project。gradle最庞大的功能就是提供了许多插件，插件中自带task，属性。<br>
例如apply form ‘java’，这句代码的意思就是引入了java插件，引入插件之后就可以配置相关的属性，执行任务。<br>
在一个gradle配置的工程里一般会有如下相关的文件：<br>
build.grade settings.grade grade.properties gradle/ <br>
build.gradle 中设定任务，grade.properties中设置grade运行的相关属性，setings.gradle是sub project的说明。gradle/文件夹是默认下载的grade版本。<br>
以一个根build.gradle进行说明：<br>
```bash
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
* repositories 中一般添加的是依赖库名称，目前广为人知的只有两个库jcener和maven，几乎所有的插件都是发布到这两个库中的
* dependencies 中添加依赖
* allprojects 是对所有的包括根project在内的一个配置
* task clean 是定义的一个clean 的task

gradle 的语法以一种很接近自然语言的状态存在着，在引进插件之后这种感觉更加强烈，插件中的propert和task贴近语言习惯。
