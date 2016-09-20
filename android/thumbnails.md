#Android 缩略图
android 手机 上可以看到的图片都会有一张缩略图，在加载相册显示图片时，可以先显示这张缩略图。<br>
提取图片和视频的缩略图可以直接访问:<br>
1. android.provider.MediaStore.Images.Thumbnails<br>
2. android.provider.MediaStore.Video.Thumbnails<br>
原图存放在：<br>
1. MediaStore.Images<br>
2. MediaStore.Video<br>
我们关心的一个最关键的问题是缩略图和原图是怎么关联的。<br>
表Thumbnails的image_id 和images 的_id 是一样的通过这两个值就可以找出两表的映射关系，进而找出图片的位置。<br>
下面是通过原图的地址找出缩略图的地址：<br>

```java
private static final String[] PHOTO_PROJECTIONS = {
    MediaStore.Images.Media._ID,
    MediaStore.Images.Media.DATA,
    MediaStore.Images.Media.DATE_ADDED,
    MediaStore.Images.Media.DISPLAY_NAME,
    MediaStore.Images.Media.BUCKET_ID,
    MediaStore.Images.Media.BUCKET_DISPLAY_NAME
};

private static final Uri PHOTO_URI = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;

private static String getThumbnailsId(@NonNull final Context context,
    @NonNull final String path) {
  final ContentResolver cr = context.getContentResolver();
  final Cursor cursor = cr.query(PHOTO_URI, ALBUM_PROJECTIONS,
      MediaStore.Images.Media.DATA + "=?", new String[]{path}, null);
  final String _id;
  if (null != cursor && cursor.moveToFirst()) {
    _id = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media._ID));
  } else {
    _id = null;
  }
  if (null != cursor && !cursor.isClosed()) {
    cursor.close();
  }
  return _id;
}

/**
 * 得到相册图片中的缩略图 uri
 */
public static String getThumbnailsPath(@NonNull final Context context,
    @NonNull final String path) {
  final String _id = getThumbnailsId(context, path);
  if (TextUtils.isEmpty(_id)) {
    return null;
  }
  final Uri pathUri = Uri.withAppendedPath(MediaStore.Images.Thumbnails.EXTERNAL_CONTENT_URI,
      String.valueOf(getThumbnailsId(context, path)));
  return pathUri.toString();
}
```
