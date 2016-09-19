#android bug总结二
##逻辑bug
1. String.formact警告<br>
原因：String.formact()在转换时需要指定日期的形式，eg：String.formact(Local.getDefault,...)
2. null异常<br>
原因：在许多情况下类都需要进行null判断，重复的判断，有时会导致逻辑的冗余,可考虑将其写成一个函数eg：checkNotNull。
3. 三星内存泄漏
原因：系统Rom问题(ClipboardUImanager) http://blog.csdn.net/xingchenxiao/article/details/48549215
4. 360手机内存泄漏
原因：系统Rom问题（LargeBackground）

