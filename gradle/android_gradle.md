#android buildgradle 文件说明
android studio 采用gradle的脚本的形式来编译apk,这样使得apk的编译变得可编辑，可以手动的配置变量包括编译环境，版本，以及使用到的第三方库。这样做的好处是使编译变得更加直接，虽然看起来会麻烦一些，但是只要经过简单的学习就可以熟练运用，适合开发人员。
下面我将对最重要的android studio 中项目下的build.gradle文件做一下简单的说明。
android 项目可以通过这种方法引入插件。

- apply plugin ‘com.android.aaplication’ apply是引入的意思，plugin 是插件的意思。在项目中如果要引入插件的话就通过这种形式引进。
- android {} 这个标签的作用是对android项目做一些相关的配置。
- dependencies {} 添加依赖。其中有几个字段：compile testcompile 其作用是声明编译和测试编译引入的jar包，project，lirbraries.
- task{} 如其名是任务的意思。
- apply from ‘*.gradle’ 可以引入其他文件

gradle 文件中会看到许多标签，这些标签可以称为特性(property)，我们在引入插件时，其实就将这些特性引入进来了。我们在使用时只需要配置这些特性即可，在构建时会自动调用。
在android{} 中一些常见的标签有sourceSets，dexOptions, linkOptions, compileOptions, compileSdkVersion, buildTypes,defaultConfig…..
有些标签的含义比较直观，现在对这些标签做出解释。

- dexOptions android项目在生成dex时的做出的设置，其中对这个标签做出设置主要是解决了生成dex时方法不能超过64K的问题。
- sourceSets 如其英名含义:源的设置。android项目的文件对应的路径，可以在这个标签中设置src res对应的文件夹。
- buildTypes  其含义是建立不同的版本：开发版本，发布版本...
- signingConfigs 可以设置签名，当我们发布版本时不再需要去设置，当发布不同的版本时可通过脚本配置。
- productFlavors 可以设置不同的渠道发布。当发布的渠道多时，编译的时候会很长，现在有一种更好的方法是通过python打包发布，更快。
- defaultConfig 里可以设置包名，app适应的最低的SDK版本号，以及版本名，版本号。

对项目中经常用到的一些插件做出说明：

- me.tatarka.retrolambda //JDK 8中开始支持的一种函数式推导语言,能够大量减少匿名内部类那种冗余的代码
- findbugs //静态分析工具承诺无需开发人员费劲就能找出代码中已有的缺陷
- com.getkeepsafe.dexcount  //android程序中的方法个数不能超过64K，这个插件用来计算方法的个数。
- com.jakewharton.hugo // 用于打印函数信息及执行时间的工具，仅在debug模式生效
- mutidex

- DexOpt有一个问题，DexOpt会把每一个类的方法id检索起来，存在一个链表结构里面。但是这个链表的长度是用一个short类型来保存的，导致了方法id的数目不能够超过65536个。当一个项目足够大的时候，显然这个方法数的上限是不够的。尽管在新版本的Android系统中，DexOpt修复了这个问题，但是我们仍然需要对低版本的Android系统做兼容。
为了解决方法数超限的问题，需要将该dex文件拆成两个或多个，为此谷歌官方推出了multidex兼容包，配合AndroidStudio实现了一个APK包含多个dex的功能。
