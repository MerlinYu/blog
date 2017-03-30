#Fragment
Android是在Android 3.0 (API level 11)开始引入Fragment的。<br>
可以把Fragment想成Activity中的模块，这个模块有自己的布局，有自己的生命周期，单独处理自己的输入，在Activity运行的时候可以加载
或者移除Fragment模块。<br>
可以把Fragment设计成可以在多个Activity中复用的模块。<br>
当开发的应用程序同时适用于平板电脑和手机时，可以利用Fragment实现灵活的布局，改善用户体验。Fragment 替代TabActivity做导航，性能更好。<br>
为了支持所有的API，现在一般开发中常用的fragment是android.v4.support，同样要获取FragmentManager也使用getsupportFragmentManager方法去获得。
Activity中如果想要使用fragment必须继承FragmentActivity。其中fragment 的生命周期如下：<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/fragment.png)<br>
在app当中常见到的一个问题是：重新打开app时fragment重叠。发生这种现象的原因是activity非正常退出（home键或跳转到其他应用）系统发生回收activity，
在activity中的onsaveinstancestate中保存fragment的状态在oncreate中恢复fragment的状态。将上次已经加载过的fragment都show出来，造成重叠。
同时我们又会重新new fragment又会造成重叠。<br>
解决的思路是：将上次保存的fragment赋值给当前的fragment。<br>
解决的步骤是：<br>

1. 重写onsaveinstancestate
```java
@Override
protected void onSaveInstanceState(Bundle outState) {
  if(null != outState) {
    Timber.d("save tab index " + getActiveIndex());
    outState.putInt(TAB_ACTIVE_INDEX, getActiveIndex());
  }
  super.onSaveInstanceState(outState);
}
```

2. 在oncreate中恢复状态。
```java
protected void onCreate(Bundle savedInstanceState) {
  if(null != savedInstanceState) {
    mIsSaveState = true;
    mSaveTabIndex = (int)savedInstanceState.get(TAB_ACTIVE_INDEX);
    Timber.d(" on create savedInstanceState saveTabIndex = " + mSaveTabIndex);
  }
  super.onCreate(savedInstanceState);
  if(mIsSaveState && mSaveTabIndex != -1) {
    FragmentManager fm  = getSupportFragmentManager();
    tabManager.restoreSaveState((ArrayList<Fragment>)fm.getFragments(),mSaveTabIndex);
    mIsSaveState = false;
  }
}
```
在tabManager中将上次的fragment恢复：
```java
for (Fragment fragment : fragmentArrayList) {
  if (fragment instanceof  HomeTabFragment) {
    homeFragment = (HomeTabFragment) fragment;
  } else if (fragment instanceof  SearchHomeFragment) {
    mSearchFragment = (SearchHomeFragment) fragment;
  } else if (fragment instanceof CartFragment) {
    cartFragment = (CartFragment) fragment;
  } else if (fragment instanceof ItemShowHomeFragment) {
    itemshowFragment = (ItemShowHomeFragment) fragment;
  } else if (fragment instanceof ProfileFragment) {
    profileFragment = (ProfileFragment) fragment;
  }
}
tabDisplay.hideContent(fragmentArrayList);
```
