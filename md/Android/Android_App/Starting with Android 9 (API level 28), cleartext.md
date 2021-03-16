Starting with Android 9 (API level 28), cleartext 

Starting with Android 9 (API level 28), cleartext support is disabled by default.

[参考](https://stackoverflow.com/questions/45940861/android-8-cleartext-http-traffic-not-permitted)

[官方说明：](https://developer.android.com/training/articles/security-config#CleartextTrafficPermitted)

## 一种方案:

1. 创建文件：

res/xml/network_security_config.xml
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">Your URL(ex: www.baidu.com)</domain>
    </domain-config>
</network-security-config>
```
**2、更改AndroidManifest.xml**
```
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
    <uses-permission android:name="android.permission.INTERNET" />
    <application
        ...
       android:networkSecurityConfig="@xml/network_security_config"
        ...>
        ...
    </application>
</manifest>
```
## 第二种方案:

2. 修改AndrdoidManifest.xml
```
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
    <uses-permission android:name="android.permission.INTERNET" />
    <application
        ...
        android:usesCleartextTraffic="true"
        ...>
        ...
    </application>
</manifest>
```