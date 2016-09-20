#RN Flex布局
Flex布局篇 http://www.liuchungui.com/blog/2016/04/04/reactnativezhi-flexbu-ju-zong-jie/<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/flex_1.jpg)
![](https://github.com/MerlinYu/blog/blob/master/blog_file/react/flex_2.jpg)<br>
通过博客我们知道了Flex布局的属性，用法，但是用起来可能没有那么得心应手，如图所示通过两个简单的例子来说明Flex布局的用法。通过这两个例子，以后可以完成大部分的布局。
第一个布局是以一张图片背景，图片和文字居中，如何来实现这样的效果呢？先看背景图片的style<br>
```java
  container: {
    flex: 1,
    flexDirection:'column',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#ffffff',
    width:null,
    height:null
    // resizeMode:'strectch/contain'
  },
```
关键的地方在width:null,height:null,至于注释掉的resizeMode可以设置图片的缩放格式。<br>
第二个布局是一个标题栏的布局，返回键居左，标题居中，看起来简单如果使用原生布局很轻松就可以达到，但是在刚开始使用Flex布局时，会感觉到困难，最关键的地方是如何使标题居中。
先看一下相关的布局:<br>
```bash
    <View style = {styles.container} >
        <View style = {styles.base_container}>
    // back
        <TouchableOpacity style = {styles.back_container} onPress={this.props.onBack}>
          <Text style = {styles.back}>
            back
          </Text>
        </TouchableOpacity>
// title
         <View style={styles.titl_container}>
        <Text style = {styles.title}>
        {this.props.title}
        </Text>
        </View>
// divide line
        </View>
        <View style = {styles.divide} />
      </View>
```bash
style如下 ：<br>
```bash
  container: {
    height:41,
    backgroundColor:'#757370',
    flexDirection:'column',
  },
  base_container: {
    height:40,
    backgroundColor:'#757370',
    flexDirection:'row',
  },

  back_container:{
    width:50,
    justifyContent:'center',
    alignItems:'center'
  },
  back: {
    fontSize:16,
  },
  divide: {
    height:1,
    backgroundColor:'#000000',
  },
  titl_container:{
    flex:1,
    justifyContent:'center',
    alignItems:'center',
    marginLeft: -30,
  },
  title : {
    fontSize:20,
  },
```
为什么要这样布局，首先看container中指定高度默认flex = 0,flex =0 代表实际大小，不进行扩展。back_container指定了宽度width，然后使back文字居中，剩下的就是title_container首先指定flex=1进行扩展，然后指定了居中，这只是在剩下的空间居中，如果要达到屏幕居中还需要设置marginLeft.
