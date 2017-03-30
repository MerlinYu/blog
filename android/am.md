#am 命令启动程序
am命令可以直接在命令端输入命令来控制手机。<br>
几个启动指定程序activity的例子<br>
Music 和 Video（音乐和视频）的启动方法为：<br>
```bash
# am start -n com.android.music/com.android.music.MusicBrowserActivity
# am start -n com.android.music/com.android.music.VideoBrowserActivity
# am start -n com.android.music/com.android.music.MediaPlaybackActivity
```

Camera（照相机）的启动方法为：
```bash
# am start -n com.android.camera/com.android.camera.Camera
```
Browser（浏览器）的启动方法为：
```bash
# am start -n com.android.browser/com.android.browser.BrowserActivity
```
启动浏览器 :
```bash
am start -a android.intent.action.VIEW -d  http://www.google.cn/
```
拨打电话 :
```bash
am start -a android.intent.action.CALL -d tel:10086
```

启动 google map 直接定位到北京 :
```bash
am start -a android.intent.action.VIEW geo:0,0?q=beijing
```
