#drawable canvas
在android中如果要实现自定义view，canvas和drawable是两个必不可少的类。canvas故名思义是一块画布，可以在这块画布上画出你想要的东西。而其本身也提供了一些API可以画点，线，圆，路径。<br>
```java
Canvas canvas = new Canvas(bitmap);
```
通过上述代码创建了一张属于bitmap的画布，通过对画布的操作修改Bitmap。Noti: 可以.9drawable。<br>
```java
canvas.drawColor(cardColorSetting.backgroundColor);
canvas.drawText(motto, cardSizeSetting.paddingLeft, marginTop, paint);
canvas.drawBitmap(transformToCircle(avatar),
    (width-cardSizeSetting.avatarSize)/2,
    marginTop,
    null);
```
如果要画多行文本，这里可以使用另一个控件：<br>
```java
if (!TextUtils.isEmpty(userCard.getDescribe())) {
  buffer.append("\t" + userCard.getDescribe());
  TextPaint tPaint = new TextPaint();
  tPaint.setTextSize(cardSizeSetting.fontNormalSize);
  tPaint.setAntiAlias(true);
  staticLayout = new StaticLayout(buffer.toString(),
      tPaint,
      width - cardSizeSetting.paddingLeft * 2,
      Layout.Alignment.ALIGN_NORMAL,
      1.0f,
      0.5f,
      false);
  descHeight = staticLayout.getHeight();
}

if (staticLayout != null) {
  paint.setTextSize(cardSizeSetting.fontNormalSize);
  canvas.save();
  canvas.translate(cardSizeSetting.paddingLeft, marginTop);
  staticLayout.draw(canvas);
  marginTop = marginTop + descHeight;
  canvas.restore();
}
```
代码简单，也不用多说什么，一看就会。<br>
另外提一下如何自定义view生成圆角图片，关键代码如下：如果要自定义view在OnDraw使用。<br>
同样可以用下边的代码生成圆形的图片。<br>
```java
// 圆角图片
private Bitmap transformToRoundCircle(Bitmap bitmap,int radius) {
  Bitmap circleBitmap = Bitmap.createBitmap(bitmap.getWidth(),bitmap.getHeight(), Bitmap.Config.ARGB_8888);
  Canvas canvas = new Canvas(circleBitmap);
  Paint paint = new Paint();
  Rect rect = new Rect(0,0,bitmap.getWidth(),bitmap.getHeight());
  paint.setAntiAlias(true);
  canvas.drawARGB(0,0,0,0);
  canvas.drawRoundRect(new RectF(rect), radius, radius,paint);
  paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
  canvas.drawBitmap(bitmap,rect,rect,paint);
  bitmap.recycle();
  return circleBitmap;
 }
 ```
