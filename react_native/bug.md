#RN 开发目前遇到的坑
学习ReactNative恐怕是最让我恼火的一门技术吧，在开发RN的过程中总是会遇到各种坑，想像不到的坑。中间有许多坑我都已经忘了是怎么解决的。<br>
官方教程android 和react-native 混合开发https://facebook.github.io/react-native/docs/embedded-app-android.html<br>
按照这个教程开发100%会死掉。<br>
注意在npm init 时如果不知道参数的含义就默认设置<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_0.png)<br>
前期准备：<br>
1）. 在AndroidManifest.xml中添加设置activity的权限<br>
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity"/><br>
2）. 在app build.gradle中修改support的依赖版本：<br>
compile 'com.android.support:appcompat-v7:23.0.1'<br>
3）.npm start 开启服务器<br>
4）. 手机摇动，设置调试的IP地址和端口 IP:8081。<br>
做完这些工作之后下面就是解决坑的过程。<br><br>
问题1：<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_1.jpg)<br>
在网上找了大量的资料都说是要打开服务器，设置调试地址，然并卵。
最后查看package.json文件发现自动生成的json文件中并没有对react的依赖。
```bash
"dependencies": {
    "react-native": "^0.26.3",
      }
```
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_1_0.png)<br>
当时我没有留意到这个警告，结果在后期一直解决上述问题。
添加依赖:
```bash
"dependencies": {
     "react": "^15.0.2",
    "react-native": "^0.26.3",
      }
```
然后使用npm install —save react@15.0.2。这个问题才得到解决。
问题二：
解决了上一个问题之后又出现了如下的问题，在网上找到一个相关的解决办法：<br>
https://github.com/jhabdas/react-native-webpack-starter-kit/issues/101<br>
https://github.com/facebook/react-native/issues/7409<br>
说实话我也不知道这是什么问题…解决办法是降低react-native的开发版本。<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_2.jpg)
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_3.png)<br>

这些问题解决之后的package.json内容是：
```bash
{
  "name": "apprn",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^0.14.8",
    "react-native": "^0.25.1"
  }
}
```

问题三：<br>
这个问题的原因是将React-native更新到0.26版本之后 UIManager中的一个函数访问错误，解决办法如下：<br>
https://github.com/facebook/react-native/issues/7409<br>
解决办法是将node_modules react-native中的UiManager重写。坑比的0.26<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_4.jpg)<br>

问题四：<br>
这个问题的原因好像是在<view></view>中有注释。<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/bug_5.jpg)<br>
