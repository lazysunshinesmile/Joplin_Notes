Notification使用

Notification使用

参考：https://www.jianshu.com/p/ec67ba83934a

## **一、创建通知**

在 **NotificationCompat.Builder** 对象中为通知指定 UI 信息和操作，然后通过调用 **NotificationCompat.Builder.build()** 创建通知，它将返回包含符合指定要求的 Notification 对象

要发出通知，通过调用 **NotificationManager.notify()** 将 Notification 对象传递给系统

## **二、必需的通知内容**

Notification 对象必须包含以下内容：

- 小图标，由 setSmallIcon() 设置
- 标题，由 setContentTitle() 设置
- 详细文本，由 setContentText() 设置

其他通知设置和内容都是可选的

## **三、通知操作**

尽管通知操作都是可选的，但是一般都需要向通知添加一个操作，例如让用户直接从通知跳转到应用中的 Activity，在其中查看一个更详细的信息

在 Notification 内部，操作本身由 **PendingIntent** 定义，后者包含在应用中启动 Activity 的 Intent。如果要在用户点击抽屉式通知栏中的通知文本时启动 Activity，则可通过调用 **setContentIntent()** 来添加 PendingIntent

## **四、通知优先级**

可以根据需要设置通知的优先级。优先级充当一个提示，提醒设备 UI 应该如何显示通知。 要设置通知的优先级，需调用 **NotificationCompat.Builder.setPriority()** 并传入一个 **NotificationCompat** 优先级常量。有五个优先级别，范围从 **PRIORITY_MIN (-2)** 到 **PRIORITY_MAX (2)**；如果未设置，则优先级默认为 **PRIORITY_DEFAULT (0)**

## **五、PendingIntent**

PendingIntent类封装了一个Intent和一个动作，当调用该类的send方法的时候，将会执行该动作。由于PendingIntent类是一个待处理的意图，这个动作通常是在将来的某个时刻要调用的一个操作，很可能是系统要调用的。例如，一个PendingIntent类可以用于构造一个Notification，以便当用户触碰该通知的时候，能够触发某些动作

PendingIntent类中的动作是Context类中的几个方法之一，例如**startActivity，startService，sendBroadcast**

使用PendingIntent来启动一个Activity

   PendingIntent pi = PendingIntent.getActivity(Context context, int requestCode,Intent intent, int flags);

静态方法getActivity是返回PendingIntent类的一个实例的几个方法之一，其他的方法还有**getActivities、getService和getBroadcast**

这些方法决定了最终PendingIntent所能执行的动作，可以用来启动Activity，启动Service，发送广播等

## **六、NotificationManager**

要发布一个通知，可以使用NotificationManager，这是Android系统中的内建服务之一，是一个已有的系统服务，可以通过如下代码来获取它

NotificationManager mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

然后，在该NotificationManager上调用notify方法来发布一个通知，需要传入唯一的ID和Notification对象
mNotificationManager.notify(int id, notification);
通知ID用于标识该通知，在想要取消特定Notification的时候，就需要使用到它
mNotificationManager.cancel(int id);
此外，除非发生以下情况之一，否则通知会一直可见：

- 用户单独或通过使用“全部清除”清除了该通知（如果通知可以清除）
- 用户点击通知，且在创建通知时调用了 **setAutoCancel(true)**
- 针对特定的通知 ID 调用了 **cancel(int id)**
- 调用了 **cancelAll()** 方法，该方法将删除之前发出的所有通知