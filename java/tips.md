#java 小知识点
###1. string 长度
Java String 在内存中的引用的长度是4*8个字节如果这样算的话理论上String的长度可以达到4G.<br>
但是在class文件的规范中， CONSTANT_Utf8_info 表中使用一个16 位的无符号整数来记录字符串的长度的，最多能表示 65536 个字节即64K，<br>
而java class 文件是使用一种变体UTF-8格式来存放字符的，null 值使用两个字节来表示，因此只剩下 65536－ 2 ＝ 65534个字节。<br>
也正是变体UTF-8 的原因，如果字符串中含有中文等非ASCII 字符，那么双引号中字符的数量会更少（一个中文字符占用三个字节）。<br>
如果超出这个数量，在编译的时候编译器会报错。
###2. char 2个字节
