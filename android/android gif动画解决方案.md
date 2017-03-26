## Android GIF动画解决方案

最近在做一个开机动画，因为动画太长，所以在实现的过程中从简单到复杂的实现有几种方案供大家参考。<br>

### AnimationDrawable 

定义Drawable list 资源：

	<?xml version="1.0" encoding="utf-8"?>
	<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
	                android:oneshot="false">
	    <item
	        android:drawable="@mipmap/start_00000"
	        android:duration="40"/>
	
	</animation-list>

	ImageView imageView;
	imageView.setBackgroundResource(R.drawable.anim);
	AnimationDrawable anim = (AnimationDrawable) imageView.getBackground;
	anim.start();
	
使用示例：
http://www.jb51.net/article/77481.htm<br>
这是最简单的GIf动画实现方案，但是只能实现简单的gif帧动画，如果帧图片资源大会造成OOM异常，此种方法只适用轻量的gif动画。<br>

### Handler setBackGround
这种方法实现看起来是最蠢的实现方法了，利用Handler发送消息，收到消息后加载帧图片，然后显示图片。<br>
这种实现方式我只能说太low了，因为从Drawable资源中取图片会耗时，set Drawable会耗时，整个gif动画显示很卡，GIf动画是3.5s，结果整个动画流程执行时间用了9.5s，后来采用双缓冲的技术来预加载图片,结果并没有减少多少执行时间。<br>
尤其在不同的手机上执行的动画的时间差异很大。体验太差，所以才有的之后的几种实现方法。

### SurfaceView 实现
SurfaceView可以在线程中绘图。在实现中自定义一个SurfaceView然后启动线程中去不断的取背景图片，在Canvas中绘图。这种不断的重绘背景图片，而造成的Gif动画效果的时间会超过GIf动画的时间，3.5s的动画大概播放了有6s左右。<br>
下面是绘制一帧的代码：<br>

    private void drawMovieFrame() {
      Timber.d("===tag==== drawMovieFrame");
      long now = SystemClock.uptimeMillis();
      long mStart = 0;
      Canvas canvas = holderWeak.get().lockCanvas();
      if (canvas == null) {
        return;
      }

      if (mStart == 0) {
        mStart = now;
      }
      if (mMovie != null) {
        int duration = mMovie.duration();
        if (duration == 0) {
          duration = 1000;
        }
        // 算出需要显示第几帧
        int relTime = (int) ((now - mStart) % duration);
	//      显示第几帧
	     mMovie.setTime(relTime);
	     mMovie.draw(canvas, 0, 0);
	      }
	      holderWeak.get().unlockCanvasAndPost(canvas);
	    }


### movie 实现

这是网上的一个示例：<br>
https://github.com/sbakhtiarov/gif-movie-view/blob/master/src/com/basv/gifmoviewview/widget/GifMovieView.java

在实现过程中会遇到一个lowMemory kill的问题，后来发现需要在Mainfest设置硬件不加速才降低内存的使用才不会出现lowMemory kill的现象。<br>


	<activity
		    android:name=".test.GifActivity"
		    android:hardwareAccelerated="false"
		    />
		    
这种方式在播放gif动画时需要适配屏幕的大小，在适配时可以在onDraw中计算出Canvas缩放的比例，然后对其进行缩放：<br>
	mMovie.setTime(relTime);
	
	int mViewWidth = getWidth();
	int mViewHeight = getHeight();
	mMovieWidth = mMovie.width();
	mMovieHeight = mMovie.height();
	
	float scaleY = (float) mViewHeight / (float) mMovieHeight;
	float scaleX = (float) mViewWidth / (float) mMovieWidth;
	//等比例缩放, 取小的比例值
	float sampleRate = scaleX < scaleY ? scaleX : scaleY;
	canvas.scale(sampleRate, sampleRate);
	mMovie.draw(canvas, 0, 0);
一开始的解决方案是这样的，后来发现网上已经有第三方库可以实现播放Gif动画，效果不错，所以最后采用第三方库android-gif-drawable实现。<br>


### Glide 库实现

Google 的Glide库Facebook的Fresco都可以实现对Gif图片的加载。下面是三者之间的对比。
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&ved=0ahUKEwj_8cXGuNPSAhXHw1QKHQ0FAPAQFggzMAI&url=http%3A%2F%2Fblog.qiji.tech%2Farchives%2F6344&usg=AFQjCNESjfSNv-rHx-CAkae6zEh33vhvZg&sig2=TrOZjxPhEW8VD1J7VSBN3Q

Glide和Picasso很相似，它比Picasso在缓存、gif和长图片处理上有优势,Glide在缓存时会将一张图片的所加载过的尺寸都缓存下来而Picasso只缓存一张全尺寸图片。而Picasso的大小大约是118KB，而Glide大约有430KB。<br>

Glide 加载 Gif图片：

      Glide.with(this)
          .load(R.mipmap.launch_2)
          .asGif()
          .diskCacheStrategy(DiskCacheStrategy.SOURCE)
          .into(glide_gif);

Glide 在加载 Gif图片时需要设置缓存的来源（diskCacheStrategy）。
但是经过测试了现此种方法在加载Gif资源时特别耗内存，加载Gif资源前后内存分别是16M ， 加载之后却达到了40多M， 在大型的项目中此种方案不行。而且在项目中已经采用了Picasso库加载图片，不值得为了加载 gif而再引入Glide库。<br>

### android-gif-drawable
这是网上的一个gif库，其地址是：https://github.com/koral--/android-gif-drawable
其实现原理：定义Runnable取图片然后通过Handler发送消息。刷新UI。但是其底层解码使用C实现，极大的提高了解码效率，同时很大程度上避免了OOM现象出现。

	    GifImageView gifImageView = new GifImageView(this);
		 GifDrawable gifDrawable = new GifDrawable(getResources(), R.mipmap.launch);
		 gifImageView.setImageDrawable(gifDrawable);
android-gif-drawable原理是利用线程池去取图片，handler刷新，其优势在于底层使用c去解码。<br>
```java
// git drawable draw drawable
@Override
	public void draw(@NonNull Canvas canvas) {
		final boolean clearColorFilter;
		if (mTintFilter != null && mPaint.getColorFilter() == null) {
			mPaint.setColorFilter(mTintFilter);
			clearColorFilter = true;
		} else {
			clearColorFilter = false;
		}
		if (mTransform == null) {
			canvas.drawBitmap(mBuffer, mSrcRect, mDstRect, mPaint);
		} else {
			mTransform.onDraw(canvas, mPaint, mBuffer);
		}
		if (clearColorFilter) {
			mPaint.setColorFilter(null);
		}

		if (mIsRenderingTriggeredOnDraw && mIsRunning && mNextFrameRenderTime != Long.MIN_VALUE) {
			final long renderDelay = Math.max(0, mNextFrameRenderTime - SystemClock.uptimeMillis());
			mNextFrameRenderTime = Long.MIN_VALUE;
			mExecutor.remove(mRenderTask);
			mRenderTaskSchedule = mExecutor.schedule(mRenderTask, renderDelay, TimeUnit.MILLISECONDS);
		}
	}
```
RenderTask去取图片通过 Handler发送消息
```java
class RenderTask extends SafeRunnable {

	RenderTask(GifDrawable gifDrawable) {
		super(gifDrawable);
	}

	@Override
	public void doWork() {
		final long invalidationDelay = mGifDrawable.mNativeInfoHandle.renderFrame(mGifDrawable.mBuffer);
		if (invalidationDelay >= 0) {
			mGifDrawable.mNextFrameRenderTime = SystemClock.uptimeMillis() + invalidationDelay;
			if (mGifDrawable.isVisible() && mGifDrawable.mIsRunning && !mGifDrawable.mIsRenderingTriggeredOnDraw) {
				mGifDrawable.mExecutor.remove(this);
				mGifDrawable.mRenderTaskSchedule = mGifDrawable.mExecutor.schedule(this, invalidationDelay, TimeUnit.MILLISECONDS);
			}
			if (!mGifDrawable.mListeners.isEmpty() && mGifDrawable.getCurrentFrameIndex() == mGifDrawable.mNativeInfoHandle.getNumberOfFrames() - 1) {
				mGifDrawable.mInvalidationHandler.sendEmptyMessageAtTime(mGifDrawable.getCurrentLoop(), mGifDrawable.mNextFrameRenderTime);
			}
		} else {
			mGifDrawable.mNextFrameRenderTime = Long.MIN_VALUE;
			mGifDrawable.mIsRunning = false;
		}
		if (mGifDrawable.isVisible() && !mGifDrawable.mInvalidationHandler.hasMessages(MSG_TYPE_INVALIDATION)) {
			mGifDrawable.mInvalidationHandler.sendEmptyMessageAtTime(MSG_TYPE_INVALIDATION, 0);
		}
	}
}
```
其中通过private static native long renderFrame(long gifFileInPtr, Bitmap frameBuffer);方法去解析图片。
