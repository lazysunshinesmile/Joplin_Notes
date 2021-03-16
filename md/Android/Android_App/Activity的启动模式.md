Activity的启动模式

Activity的启动模式

singleTask

singleTop

singleInstanse

standard

参考：https://blog.csdn.net/mynameishuangshuai/article/details/51491074

参考：https://developer.android.com/guide/topics/manifest/activity-element?hl=zh-cn#lmode（官方）
## standard

## singleTop模式分3种情况

1、当前栈中已有该Activity的实例并且该实例位于栈顶时，不会新建实例，而是复用栈顶的实例，并且会将Intent对象传入，回调onNewIntent方法。  onNewIntent()---->onResart()------>onStart()----->onResume().

2、当前栈中已有该Activity的实例但是该实例不在栈顶时，其行为和standard启动模式一样，依然会创建一个新的实例
3、 当前栈中不存在该Activity的实例时，其行为同standard启动模式

standard和singleTop启动模式都是在原任务栈中新建Activity实例，不会启动新的Task，即使你指定了taskAffinity属性。那么什么是taskAffinity属性呢，可以简单的理解为任务相关性。

（1）这个参数标识了一个Activity所需任务栈的名字，默认情况下，所有Activity所需的任务栈的名字为应用的包名
（2）我们可以单独指定每一个Activity的taskAffinity属性覆盖默认值
（3）一个任务的affinity决定于这个任务的根activity（root activity）的taskAffinity
（4）在概念上，具有相同的affinity的activity（即设置了相同taskAffinity属性的activity）属于同一个任务
（5）为一个activity的taskAffinity设置一个空字符串，表明这个activity不属于任何task

很重要的一点taskAffinity属性不对standard和singleTop模式没有任何影响，即时你指定了该属性为其他不同的值，这两种启动模式下不会创建新的task（如果不指定即默认值，即包名）

## singleTask-栈内复用模式

这个模式十分复杂，有各式各样的组合。在这个模式下，如果栈中存在这个Activity的实例就会复用这个Activity，不管它是否位于栈顶，复用时，会将它上面的Activity全部出栈，并且会回调该实例的onNewIntent方法。其实这个过程还存在一个任务栈的匹配，因为这个模式启动时，会在自己需要的任务栈中寻找实例，这个任务栈就是通过taskAffinity属性指定。如果这个任务栈不存在，则会创建这个任务栈。

singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈
        （1）如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去
如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例
        （2）如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法
        （3）如果不存在该实例，则新建Activity，并入栈

即使是两个不同的应用通过taskAffinity指定了两个Activity为相同的taskID，依然会把他们压入同一个栈内。

## singleInstance-全局唯一模式

该模式具备singleTask模式的所有特性外，与它的区别就是，这种模式下的Activity会单独占用一个Task栈，具有全局唯一性，即整个系统中就这么一个实例，由于栈内复用的特性，后续的请求均不会创建新的Activity实例，除非这个特殊的任务栈被销毁了。以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。