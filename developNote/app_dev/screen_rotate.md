# 屏幕旋转
## 禁止屏幕旋转，固定屏幕方向
- 需要在AndroidManifest.xml中将对应的Activity的配置改为如下：
```xml
<!-- 竖屏-->
 android:screenOrientation="portrait"
 <!-- 横屏-->
 android:screenOrientation="landscape"
``` 
## 屏幕旋转的时候
- 当不修改Activity在AndroidManifest.xml中的配置，当前activty发生屏幕旋转的时候，activity的生命周期调用如下：
```java
	RotateActivity: onPause
	RotateActivity: onSaveInstanceState
	RotateActivity: onStop
	RotateActivity: onDestroy
	RotateActivity: onCreate
	RotateActivity: onStart
	RotateActivity: onRestoreInstanceState
	RotateActivity: onResume
```
- 请求屏幕旋转
```java
 private void switchScreenOriention(){
    boolean isPortrait = getResources().getConfiguration().orientation ==
        Configuration.ORIENTATION_PORTRAIT;
    if(isPortrait){
        RotateActivity.this.setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
    }else{
        RotateActivity.this.setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
    }
}
```
## 不希望Activity的生命周期重新走onCreate
- 需要在AndroidManifest.xml中将对应的Activity的配置改为如下：
```xml
android:configChanges="orientation|keyboardHidden|screenSize"
```
此时屏幕旋转仅会触发　onConfigurationChanged　回调.

- 当设置了此项的时候，同时用户开启了自动旋转开关，activity无法根据手机的方向自动个旋转屏幕，
	此时需要监听系统开关做对应的设置．

- 当用户开启屏幕旋转开关的时候，需要将activity设置为跟随用户的手机方向，自动旋转

```java
 setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_USER);
```

## 系统屏幕旋转开关监听

```java
public class RotateObserver extends ContentObserver {
    private ContentResolver mResolver;
    private Context mContext;
    private RotateSwitchListener mListener;
    public interface RotateSwitchListener{
        void onChange();
    }

    public RotateObserver(Context context, Handler handler, RotateSwitchListener listener) {
        super(handler);
        this.mContext = context;
        mResolver = context.getContentResolver();
        mListener = listener;
    }

    @Override
    public boolean deliverSelfNotifications() {
        return super.deliverSelfNotifications();
    }

    @Override
    public void onChange(boolean selfChange) {
        super.onChange(selfChange);
    }
	
    /*
    * 变化通知
    */
    @Override
    public void onChange(boolean selfChange, Uri uri) {
        super.onChange(selfChange, uri);
        if(mListener != null){
            mListener.onChange();
        }

    }
    
	/*
    *注册监听者
    */
    public void registerRotateObserver() {
        mResolver.registerContentObserver(Settings.System.getUriFor(
        	Settings.System.ACCELEROMETER_ROTATION), false,this);
    }

    public void unRegisterRotateObserver() {
        mResolver.unregisterContentObserver(this);
        this.mContext = null;

    }

    /*
    * 得到屏幕旋转的状态
    * 0 　关闭
    * １　开启
    */
    
    public int getRotationStatus() {
        int status = 0;
        try {
            if(mContext != null){
                status = android.provider.Settings.System.getInt(mContext.getContentResolver(),
                        android.provider.Settings.System.ACCELEROMETER_ROTATION);
            }
        } catch (Settings.SettingNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return status;
    }
}
```

## 屏幕旋转角度监听
- 屏幕角度判断
	调用this.getWindowManager().getDefaultDisplay().getRotation();

	该函数的返回值，有如下四种：
		Surface.ROTATION_0，Surface.ROTATION_90，Surface.ROTATION_180，Surface.ROTATION_270
		其中，Surface.ROTATION_0 表示的是手机竖屏方向向上，后面几个以此为基准依次以顺时针90度递增。

- 如果要实时判断屏幕的旋转角度，可以继承一个　OrientationEventListener，用来监听屏幕的旋转角度的变化
```java
class ScreenOrientionListener extends OrientationEventListener{
        public ScreenOrientionListener(Context context, int rate){
            super(context, rate);
        }

        /**
         * 开启监听
         */
        @Override
        public void enable() {
            super.enable();
        }

        /**
         * 关闭监听
         */
        @Override
        public void disable() {
            super.disable();
        }

        /**
         *
         * @param orientation 为屏幕的旋转角度
         */
        @Override
        public void onOrientationChanged(int orientation) {
        		// 角度未知
 				if (orientation == OrientationEventListener.ORIENTATION_UNKNOWN) {
                	return;
                // 自定义监听事件处理
            }
        }
    }
```

## 注意：
	如果activity在屏幕旋转开关开启状态下，自动旋转到横屏，此时关闭屏幕旋转开关，activity会自动旋转为竖屏．