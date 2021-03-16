深入理解Android 卷2

### 使app具有SystemUID
1. 在AndroidManifest.xml中加入`sharedUserId`
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.gs.apidemo"
    android:sharedUserId="android.uid.system">
	...
	...
</manifest>
```

2. 修改Android.mk
``` 
LOCAL_CERTIFICATE := platform
```