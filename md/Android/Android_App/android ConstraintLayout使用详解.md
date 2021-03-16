android ConstraintLayout使用详解

ConstraintLayout是2016年Google I/O推出并重点宣传的一个组件，在Android Studio 2.2之后的版本支持了ConstraintLayout的开发，并加入了蓝图模式(Blueprint)方便开发者开发。[英文原文](https://codelabs.developers.google.com/codelabs/constraint-layout/index.html)。

ConstraintLayout按照字面的意思是约束布局，子View通过start(left)t、top、end(right)、bottom四个方向的约束来决定自己的位置(本文只讨论start和end属性，left和right属性忽略，如果你的minSdkVersion小与17的话只需要把start、end属性替换为left、right属性即可)。这个功能跟传统的RelativeLayout布局很相似，但是ConstraintLayout采用了和iOS布局AutoLayout相同的算法：火鸡算法(Cassowary algorithm，[参考1](http://ethanhua.cn/archives/155)，[参考2](http://cassowary.readthedocs.io/en/latest/topics/theory.html))，性能要比LinearLayout和RelativeLayout高出不少。而且ConstraintLayout还提供了很多有趣有用的属性API和动画API，深入了解ConstraintLayout，应该能得到意外的惊喜。

## API讲解

##### 1，常用API：

基本使用ConstraintLayout的功能非常非常简单，首先在build.gradle中添加最新的库：`implementation 'com.android.support.constraint:constraint-layout:1.1.0-beta4'`，然后在xml文件中调用相关的属性API即可。因为一个View有四个方向的约束条件，因此只需要4个基本的属性API就能上手ConstraintLayout了：

```
app:layout_constraintBottom_toBottomOf
app:layout_constraintEnd_toEndOf
app:layout_constraintStart_toStartOf
app:layout_constraintTop_toTopOf

```

你再举一反三一下，每个方向的约束包含两种，比如top约束包含：1约束在某个View的top(意思是和当前View的top这个View的top对齐)或者某个View的bottom(意思是和当前View的top这个View的bottom对齐)，因此四个方向的约束条件相加总共是有8个约束条件。这个8个属性的值可以是parent或者某个View的id。怎么样，上手是不是很简单？其实还有一个特殊的第9个约束条件，用的比较少，叫做“基准线约束”：  
`app:layout_constraintBaseline_toBaselineOf//默认parent//基准线约束`  
什么叫做基准线？我们上初中刚开始学习英语的时候拼写英语字母下面的线条就是基准线，用来使某一行文字对齐。基准线在TextView的设计中是个很重要的概念。这个属性对包含文本的View比如Button和TextView才起作用，主要用来帮助对齐两个控件的文本区域，与控件尺寸无关，想使用两个不同大小的控件同时又想保持其中文字对齐的时候很有帮助。不设置这个属性的话它的默认值parent，而parent的基准线位于Y坐标为0的位置即屏幕最上方。

只要掌握上面8个属性API就能在项目中使用ConstraintLayout做一些基本的UI开发了，当然，google提供的远远不止这些，下面介绍2个有用的API：

```
app:layout_constraintHorizontal_bias
app:layout_constraintVertical_bias

```

解释它俩之前先看张gif动态图：

![](https://upload-images.jianshu.io/upload_images/1323444-7b1698fe4ad28ea3.gif?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

bias.gif

`layout_constraintHorizontal_bias`必须在同时设置了start和end两个方向的约束才起作用，取值范围是0-1，默认值是0.5即start和end的比例平分都是0.5，通过动态图应该能了解不同的`layout_constraintHorizontal_bias`值带来的UI效果；`layout_constraintVertical_bias`同理必须在同时设置了top和bottom两个方向的约束才起作用。在上面的gif图片中你可以看到那个Button的高度是40dp，宽度是wrap_content，你再看下调整`layout_constraintHorizontal_bias`的拖动条的区域的表示Button的正方形那里，上下是线段表示高度是固定的，左右是朝里的箭头表示宽度是自适应的。

有一次产品经理告诉我，希望app启动页的logo位于页面的黄金分割点的位置，当时我是采用根据屏幕高度动态设置y坐标的方法在代码层面里实现的，现在回头想想，其实利用`layout_constraintVertical_bias`这个属性可以很方便设置黄金分割点，只需要在xml中将`layout_constraintVertical_bias`的值设置为0.382即可，而不再需要代码设置。

需要注意的是View的四个margin值也与View的最终位置有关，如果设置了某个方向的margin值，那么只有设置相对应方向的约束条件，这个方向的margin值才生效。比如设置了`android:layout_marginStart="20dp"`，那么当且仅当设置了start的约束条件`app:layout_constraintStart_toStartOf`或者`app:layout_constraintStart_toEndOf`这个margin值才生效。还要注意的是ConstraintLayout不支持负数的margin，如果是负数的话效果和0一样的，当然负数的padding是没问题的哈。

如果你想实现负数margin的效果的话，也有技巧，需要借助也Space这个类，如图所示，图片引用自[参考3](https://medium.com/google-developers/building-interfaces-with-constraintlayout-3958fa38a9f7)：

![](https://upload-images.jianshu.io/upload_images/1323444-7e8427c90dd27d71.gif?imageMogr2/auto-orient/strip|imageView2/2/w/1161/format/webp)

SpaceAsNegativeMargin.gif

如果一个ViewA的visibility为INVISIBLE或者GONE，那么ViewB所有依赖它的约束条件会全部集中到这个ViewA的中心点上。如果是INVISIBLE，那么这个ViewA的margin还是有效的，如果是GONE，那么ViewA的margin全部会归为0，而ViewA的margin会影响最终ViewA的中心点的位置。这时候属性`app:layout_goneMarginTop`就登场了。  
假设ViewA的marginTop是100dp，ViewB的top约束于ViewA的top(`app:layout_constraintTop_toTopOf="ViewA"`)，那么ViewB距离屏幕顶部的距离也是100dp。如果ViewA设置为GONE，那么ViewB距离屏幕顶部的距离就变成了0dp，但是如果ViewB设置了属性`app:layout_goneMarginTop="100dp"`那么即使ViewA被GONE掉了，ViewB距离屏幕顶部的距离仍然是100dp。

##### 2，迷之API：

我不知道这5个API是干啥的：

```
app:layout_constraintBaseline_creator="0"
app:layout_constraintLeft_creator="0"
app:layout_constraintTop_creator="0"
app:layout_constraintRight_creator="0"
app:layout_constraintBottom_creator="0"

```

你看下ConstraintLayout源码的使用：

```
else if (attr == R.styleable.ConstraintLayout_Layout_layout_constraintBaseline_creator) {
    
}

```

直接Skip了，什么都没干，估计以后的版本会对这5个属性进行扩展。

##### 3，宽高API：

宽高相关的API可以分为三类，下面一一介绍。

第一类：

```
app:layout_constraintWidth_default
app:layout_constraintHeight_default

```

这两个API咱们只讲`app:layout_constraintWidth_default`，因为`app:layout_constraintHeight_default`同理哈。  
`app:layout_constraintWidth_default`只有在View的宽度定义为0dp(又叫match_constraint)的时候才生效，其余情况下设置这个属性是不起任何作用的。`app:layout_constraintWidth_default`有三个值：`wrap`、`spread`和`percent`。

1.  `wrap`：等价于`android:layout_width="wrap_content"`
2.  `spread`：等价于`android:layout_width="match_parent"`
3.  `percent`：设置View的宽度为parent的比例值，比例值默认是100%，即宽度是match_parent。这个比例值通过属性`app:layout_constraintWidth_percent`设置。

第二类：

```
app:layout_constraintWidth_percent
app:layout_constraintHeight_percent

```

这两个API咱们只讲`app:layout_constraintWidth_percent`，因为`app:layout_constraintHeight_percent`同理哈。`app:layout_constraintWidth_percent`当且仅当View的宽度为0dp并且`app:layout_constraintWidth_default`的属性值为percent时才起作用。它的值范围是0-1，默认是1即100%，表示View的宽高占parent的比例值。

第三类：

```
app:layout_constraintDimensionRatio="4:9"

```

这个API用来设置宽高比例，只有下面两种情况下才生效：

1.  至少设置了一个方向的约束条件，宽高有且仅有一个是0dp，那么根据这个宽高比和那个非0dp的值，可以计算出0dp代表的最终的值；
2.  至少设置了一个方向的约束条件，宽高都是0dp，如果`app:layout_constraintWidth_default`和`app:layout_constraintHeight_default`只要有一个属性设置了或者都设置的话，那么这个比例值也会生效，而且这个比例值的优先级最高，View的宽高最终有比例值决定，而不是`app:layout_constraintWidth_default`或者`app:layout_constraintHeight_default`。

其实第二种情况只是第一种情况的特殊情景，在第二种情况下，虽然宽高都是0dp，但是其中一个0dp对应的数值是确定并且能够计算出来。因此，这个API只有在这种情况下才生效： &lt;font&gt;a&lt;/font&gt; ***至少设置了一个方向的约束条件，宽高中必须有一个是0dp，而且另一个不管是否是0dp但必须是可计算出的值***。

## GuideLine导航线

可以在ConstraintLayout中添加GuideLine导航线，它只是一个辅助工具，帮助开发者进行UI设计，不会绘制到屏幕上，也不会展现给用户。可以如下图方式添加导航线：

![](https://upload-images.jianshu.io/upload_images/1323444-101754e33020b738.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

GuideLine

```
<android.support.constraint.Guideline
    android:id="@+id/guideline"
    android:layout_width="5dp"
    android:layout_height="match_parent"
    android:orientation="vertical"
    app:layout_constraintGuide_percent="0.25"/>

```

与导航线相关的API只需要设置`orientation`和`layout_constraintGuide_percent`即可，宽高属性不起任何作用。  
导航线的顶部有一个按钮，点击可以切换左距离、右距离、百分比三种模式。  
上图中还能添加Group和Barrier，下面会介绍道。

## Group组

Group也是一个辅助类，不会绘制到屏幕上，也不会展现给用户。  
Group通过属性`app:constraint_referenced_ids`将一些View组成组进行集体操作，最常见的操作是setVisibility。

```
<android.support.constraint.Group
        android:id="@+id/group1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:constraint_referenced_ids="button3, textView2"/>

```

上面代码中的宽高不起任何作用，只需要设置`id`和`app:constraint_referenced_ids`即可。如果`group1.setVisibility(View.GONE);`那么button3和textView2将同时消失，是不是很cool？

## Barrier屏障

Barrier也是一个辅助类，不会绘制到屏幕上，也不会展现给用户。它通过属性`constraint_referenced_ids`将一些View包裹在一起形成一个屏障，然后通过属性`barrierDirection`向左上右下四个方向给某个View提供约束条件，或者叫做屏障方向。使用这些约束条件(屏障方向)的View可以防止屏障内的View覆盖自己，当屏障内的某个View要覆盖自己的时候，屏障会自动移动，避免自己被覆盖。

![](https://upload-images.jianshu.io/upload_images/1323444-6f5c4a2060a8e934.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

barrier

上图中共有6个实心箭头表示6个约束条件，我们需要`textView3`同时位于`textView1`和`textView2`的右边，因为`textView1`文案较长，我们把`textView3`的start约束在textView1的end，这样做很符合我们的需要。但是多语言的时候就麻烦了，英文显示的话`textView1`文案“China”短于`textView2`的文案“Hello，friend”，这时候`textView3`就被`textView2`覆盖了。你可能想到了一个解决办法：用代码实现，在英文环境中将`textView3`的start约束在`textView2`的end，代码如下所示：

```
ConstraintLayout constraintLayout = findViewById(R.id.constraintLayout);
ConstraintSet constraintSet = new ConstraintSet();
constraintSet.clone(constraintLayout);
constraintSet.connect(R.id.textView3, ConstraintSet.START, R.id.textView2, ConstraintSet.END, 0);
constraintSet.applyTo(constraintLayout);

```

但是这样做实在是太low了。  
或者使用`TableLayout`，或者把`textView1`和`textView2`包裹在一个垂直的`LinearLayout`中，然后让`textView3`的start约束在这个`LinearLayout`的end。但是我们有更好的办法：Barriers。直接上代码：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    android:id="@+id/constraintLayout"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:text="China"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="8dp"
        android:text="Hello，friend"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView1" />

    <android.support.constraint.Barrier
        android:id="@+id/barrier7"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="end"
        app:constraint_referenced_ids="textView2,textView1"/>

    <TextView
        android:id="@+id/textView3"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="16dp"
        android:text="3333"
        app:layout_constraintStart_toEndOf="@+id/barrier7"
        app:layout_constraintTop_toTopOf="parent" />
</android.support.constraint.ConstraintLayout>

```

屏障只需要设置两个属性：属性`constraint_referenced_ids`和属性`barrierDirection`，前者将一些View包裹在一起形成一个屏障，后者决定了屏障是水平屏障还是垂直屏障，如果是值top和bottom那么结果就是水平屏障，如果是值start(left)和end(right)那么就是垂直屏障，同时这个值也是提供给`textView3`使用的约束条件，或者叫做屏障方向。在上面代码中`barrierDirection`值是end(right)，那么屏障位于end(right)方向。  
Android oreo(8.0)提供了新的API自适应字体Autosizing TextViews [链接](https://www.jianshu.com/p/d81d3b6d22b2)，自适应字体结合屏障Barrier会更加方便进行UI适配。  
更多信息可以参考：[参考4](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2017/1017/8601.html)

## 创建约束链

约束链的原理如图所示，图片引用自[参考3](https://medium.com/google-developers/building-interfaces-with-constraintlayout-3958fa38a9f7)：

![](https://upload-images.jianshu.io/upload_images/1323444-3264bc069fa14c74.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

constraintChain

图中三个View相互约束形成一个水平居中的约束链。在单向约束链的基础上可以一键生成双向的水平居中或者垂直居中的约束链，流程如图gif所示：

![](https://upload-images.jianshu.io/upload_images/1323444-f7435611ed718c31.gif?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

constraintChain.gif

生成双向约束链之后的代码：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/constraintLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="68dp"
        android:layout_height="54dp"
        android:gravity="center"
        android:text="China"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/textView2"
        app:layout_constraintHorizontal_chainStyle="spread"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>

    <TextView
        android:id="@+id/textView2"
        android:layout_width="68dp"
        android:layout_height="43dp"
        android:gravity="center"
        android:text="Japan"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/textView3"
        app:layout_constraintStart_toEndOf="@+id/textView1"
        app:layout_constraintTop_toTopOf="parent"/>

    <TextView
        android:id="@+id/textView3"
        android:layout_width="90dp"
        android:layout_height="46dp"
        android:gravity="center"
        android:text="German"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/textView2"
        app:layout_constraintTop_toTopOf="parent"/>
</android.support.constraint.ConstraintLayout>

```

生成双向约束链之后，每个View的右下方会多出一个chainStyle按钮，这个按钮和一个属性API相关，这个属性API叫做`app:layout_constraintHorizontal_chainStyle`。这个属性API有三个值：`spread`、`spread_inside`、`packed`，默认值是`spread`，点击右下角的按钮会来回切换这三个chainStyle属性值，这三个到底取用哪个值，UI的结果还会受属性`app:layout_constraintHorizontal_weight`的影响。需要注意的是双向约束链中，每个View算是一个节点，只有头节点的`app:layout_constraintHorizontal_chainStyle`属性值才生效，其余节点的`app:layout_constraintHorizontal_chainStyle`属性不起任何作用。  
我们称这三个TextView占用父类宽度后剩余的宽度为remainWidth，这三个TextView在水平方向分割为四个区域：domainA、domainB、domainC、domainD，那么：

1.  `spread`：remainWidth将平均分配给domainA、domainB、domainC、domainD，如图
    
    ![](https://upload-images.jianshu.io/upload_images/1323444-aaa4924fb5823252.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
    
    spread
    
2.  `spread_inside`：remainWidth将平均分配给domainB、domainC，如图
    
    ![](https://upload-images.jianshu.io/upload_images/1323444-b8387aa0d7a25b44.png?imageMogr2/auto-orient/strip|imageView2/2/w/1030/format/webp)
    
    spread_inside
    
3.  `packed`：remainWidth将平均分配给domainA、domainD，如图
    
    ![](https://upload-images.jianshu.io/upload_images/1323444-52c0d789e7ac1a78.png?imageMogr2/auto-orient/strip|imageView2/2/w/962/format/webp)
    
    packed
    

chainStyle为`spread`或者`spread_inside`的时候，如果某个View的宽度为0dp，那么remainWidth的分配策略会根据属性`app:layout_constraintHorizontal_weight`进行分配，这类似LinearLayout的`layout_weight`属性，相信大家都很熟悉。

## 如何居中和对齐

居中对齐有一些技巧，四张图片全部引用自[参考3](https://medium.com/google-developers/building-interfaces-with-constraintlayout-3958fa38a9f7)：  
在其它View之间水平居中：

![](https://upload-images.jianshu.io/upload_images/1323444-58ff43f2e0cc1c57.gif?imageMogr2/auto-orient/strip|imageView2/2/w/579/format/webp)

CenterHorizontallyBetweenOtherViews.gif

在父类水平居中：

![](https://upload-images.jianshu.io/upload_images/1323444-7a3bb211f160837f.gif?imageMogr2/auto-orient/strip|imageView2/2/w/579/format/webp)

CenterInParent.gif

两个View垂直居中：

![](https://upload-images.jianshu.io/upload_images/1323444-5c95e661e1eb116d.gif?imageMogr2/auto-orient/strip|imageView2/2/w/466/format/webp)

centerVerticalTwoViews.gif

一个View居中于另一个View的边：

![](https://upload-images.jianshu.io/upload_images/1323444-e6a0528030ae2949.gif?imageMogr2/auto-orient/strip|imageView2/2/w/466/format/webp)

AligningTheCenterOfOneViewWithAnotherViewSide.gif

## 动画API

这个[视频](https://www.youtube.com/watch?v=OHcfs6rStRo&feature=youtu.be)讲解了如何结合ConstraintSet和TransitionManager做出一些很酷炫的动画。  
只需要7行代码就能实现下面gif所展示的动画：

```
fun updateConstraints(@LayoutRes id: Int) {
    ConstraintSet().apply {
        clone(this@MainActivity, id)
        applyTo(root)
    }
    TransitionManager.beginDelayedTransition(root, ChangeBounds().apply {
        interpolator = OvershootInterpolator()
    })
}

```

![](https://upload-images.jianshu.io/upload_images/1323444-05e9a69adeb0cb8c.gif?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

ConstraintLayoutAnim.gif

原理很简单，感兴趣的同许多可以查看下面的原文地址[参考5](https://hellsoft.se/animations-with-constraintlayout-and-constraintset-b4634d38981f)。

此外，利用Placeholder也能实现一些动画。  
我们用Placeholder创建一个名为placeholder.xml的模版：

```
//placeholder.xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       tools:parentTag="android.support.constraint.ConstraintLayout">

    <android.support.constraint.Placeholder
        android:id="@+id/placeholder_bg_main"
        android:layout_width="0dp"
        android:layout_height="217dp"
        app:content="@+id/content_bg_main"
        app:layout_constraintDimensionRatio="16:9"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"/>


    <android.support.constraint.Placeholder
        android:id="@+id/placeholder_plus"
        android:layout_width="50dp"
        android:layout_height="50dp"
        app:content="@+id/content_plus"
        app:layout_constraintBottom_toBottomOf="@id/placeholder_bg_main"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintHorizontal_bias="0.8"
        app:layout_constraintTop_toBottomOf="@id/placeholder_bg_main"/>

</merge>

```

Activity对应的layout是test.xml，里面对placeholder.xml模板的使用：

```
//test.xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/constraintLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include layout="@layout/placeholder"/>

    <ImageView
        android:id="@+id/content_bg_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:scaleType="fitXY"
        android:src="@mipmap/user_photo_defualt"/>
    <ImageButton
        android:id="@+id/content_plus"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:scaleType="centerCrop"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@id/content_left"
        android:src="@mipmap/club_manager_add_normal"/>

    <ImageButton
        android:id="@+id/content_left"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginStart="8dp"
        android:scaleType="centerCrop"
        android:src="@mipmap/btn_share_wechat_normal"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/content_middle"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toEndOf="@+id/content_plus"/>

    <ImageButton
        android:id="@+id/content_middle"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:scaleType="centerCrop"
        android:src="@mipmap/channel_personal_select"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/content_right"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toEndOf="@+id/content_left"/>

    <ImageButton
        android:id="@+id/content_right"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:scaleType="centerCrop"
        android:src="@mipmap/btn_share_moments_normal"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toEndOf="@+id/content_middle"/>

</android.support.constraint.ConstraintLayout>

```

Activity逻辑代码：

```

package com.crl.zzh.customrefreshlayout.test

import android.app.Activity
import android.os.Bundle
import android.support.transition.ChangeBounds
import android.support.transition.TransitionManager
import android.view.View
import android.view.animation.OvershootInterpolator
import com.crl.zzh.customrefreshlayout.R
import kotlinx.android.synthetic.main.placeholder.*
import kotlinx.android.synthetic.main.test.*

class Test : Activity(), View.OnClickListener {
    public override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.test)
        content_plus.setOnClickListener(this)
        content_left.setOnClickListener(this)
        content_middle.setOnClickListener(this)
        content_right.setOnClickListener(this)
    }

    override fun onClick(v: View) {
        placeholder_plus.setContentId(v.id)
        TransitionManager.beginDelayedTransition(constraintLayout, ChangeBounds().apply {
            interpolator = OvershootInterpolator()
            duration = 1000
        })
    }
}

```

最终的效果图：

![](https://upload-images.jianshu.io/upload_images/1323444-1c297c607e3786bd.gif?imageMogr2/auto-orient/strip|imageView2/2/w/445/format/webp)

palceholderAnim.gif

原文地址[参考6](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2017/1019/8618.html)

ConstraintLayout动画要求API不能低于19，如果API低于19的话也没什么大问题，只是没有动画效果而已。

* * *

最后，我采用一篇文章[参考1](http://ethanhua.cn/archives/155)提出的建议：

![](https://upload-images.jianshu.io/upload_images/1323444-ad60aa91256a5d56.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

suggest.png

有错误的地方希望指出，谢谢。