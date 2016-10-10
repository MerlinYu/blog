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
5. EditText 内存泄漏
原因：EditText光标闪烁导到内存泄漏。https://github.com/square/leakcanary/issues/297》<br>
leaks截图：
![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/edit_leaks.jpg)<br>
android.Widget.Editor中有一段代码是这样的：
```java
    private class Blink extends Handler implements Runnable {
        private boolean mCancelled;
        public void run() {
            if (mCancelled) {
                return;
            }
            removeCallbacks(Blink.this);

            if (shouldBlink()) {
                if (mTextView.getLayout() != null) {
                    mTextView.invalidateCursorPath();
                }

                postAtTime(this, SystemClock.uptimeMillis() + BLINK);
            }
        }

        void cancel() {
            if (!mCancelled) {
                removeCallbacks(Blink.this);
                mCancelled = true;
            }
        }

        void uncancel() {
            mCancelled = false;
        }
    }

```
可以看到Runnable闪烁时会调用：mTextView.invalidateCursorPath();与截图中的g提示一致。
解决办法：EditText.setCusrsorVisible(false).

