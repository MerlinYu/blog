#tips
1. var let const
在JavaScript中有三种声明变量的方式：var、let、const。

- var：声明全局变量，换句话理解就是，声明在for循环中的变量，跳出for循环同样可以使用。
```bash
for(var i=0;i<=1000;i++){
var sum=0;
sum+=i;
}
alert(sum);
```
声明在for循环内部的sum，跳出for循环一样可以使用，不会报错正常弹出结果<br>
- let：声明块级变量，即局部变量。
在上面的例子中，跳出for循环，再使用sum变量就会报错 <br>
注意：必须声明'use strict';后才能使用let声明变量否则浏览并不能显示结果<br>
- const：用于声明常量，也具有块级作用域
const PI=3.14;

2. background view set: http://reactcafe.com/how-to-set-a-full-screen-background-image-in-react-native
