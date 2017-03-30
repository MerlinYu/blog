#ReactNative 工程
ReactNative 官方网址：http://facebook.github.io/react-native/releases/0.26/docs/getting-started.html<br>
ReactNative 中文官方网址：http://reactnative.cn/docs/0.28/getting-started.html<br>
一篇不错的讲解ReactNative Animated博客： http://blog.csdn.net/hello_hwc/article/details/51775696<br>
Flex布局篇 http://www.liuchungui.com/blog/2016/04/04/reactnativezhi-flexbu-ju-zong-jie/<br>
ES5和ES6的写法：http://www.cnblogs.com/Mrs-cc/p/4969755.html<br>

1.  ReactNative 开始实践时首先要搭建一个开发环境，新建一个ReactNative工程, 可以参考http://reactnative.cn/docs/0.28/getting-started.html
2.  正式开发的第一步，进入到工程目录下使用npm install安必要的开发组件。npm install 会默认安装当前发布的最新版本，如果要指定reacnative的开发版本可以在工程目录下的package.json中修改开发库版本，但是要注意react-native和react版本要匹配，不然在后续的开发中可能会出现一些奇怪的bug.
npm install如何版本不匹配的话会有warnning的警告，会提示版本的要求，
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/start_1.png)
在package.json添加依赖，然后执行npm install —save react@15.0.2安装开始库。<br>
```bash
"dependencies": {
     "react": "^15.0.2",
    "react-native": "^0.26.3",
      }
```
这是前期的安装开发库的准备。<br>

3. react-native run-android 将ape安装到手机中。初次调试apk使用此命令，如果ReactNative中android工程发生变化例如在drawable 中添加图片，需要重新执行此命令安装。之后可以使用nom start或react-native start 打开服务器开发。
4. 摇晃手机，打开调戏菜单，填写IP地址和端口号。
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/start_2.jpg)
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/start_3.jpg)<br>

经过这些步骤就可以正式开始RN的开发。
可能在上述步骤中会出现一些奇怪的bug，刚开始做RN开发就是这样的。网上有许多参考例子都是用ES5写的，个人感觉ES6的代码更好懂。
还需要注意的一个问题是明确自己的开发版本，不同的版本，开发库会有所差别，最好是选定一个稳定版本的去开发，你会少解决许多莫名的bug。
