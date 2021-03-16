AlarmManager 的使用

AlarmManager 的使用

# 获取AlarmManager
```
AlarmManager alarmManager = 
(AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
```

# 设置一次性闹钟
```
Intent nIntent = 
    new Intent("com.android.contacts.IMMEDIATE_DOWNLOAD_USING_NVRAM");
PendingIntent pIntent = PendingIntent.getBroadcast(context, 
                            0, 
                            nIntent,
                            PendingIntent.FLAG_UPDATE_CURRENT);
// interval间隔
long firstTime = System.currentTimeMillis() + interval;
alarmManager.setExact(AlarmManager.RTC_WAKEUP,
                       firstTime, pIntent);
```

# 设置重复闹钟
1. 准确间隔时间
```
// interval间隔
long firstTime = System.currentTimeMillis() + interval;
alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, 
                            firstTime, interval, pIntent);
```

2. 大概间隔时间
```
setInexactRepeating（int type，long startTime，long intervalTime，PendingIntent pi）；
```
