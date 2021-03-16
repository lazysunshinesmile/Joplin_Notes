shape、selector文件使用

shape、selector文件使用

*参考
https://www.jianshu.com/p/6eb3a9bb0e97
https://developer.android.com/guide/topics/resources/drawable-resource?hl=zh-cn*

# 总览

- [总览]()
    - [shape]()
        - [shape]()
            - [属性]()
            - [corners]()
            - [gradient]()
            - [padding]()
            - [size]()
            - [solid]()
            - [stroke]()
    - [selector]()
        - [总览]()
        - [selector]()
        - [item]()

## shape

> 官方全部shape参数

	<?xml version="1.0" encoding="utf-8"?>
	<shape
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:shape=["rectangle" | "oval" | "line" | "ring"] >   --- 默认为rectangle
	    <corners  -- shape=“rectangle”时使用，
	        android:radius="integer"  -- 半径，会被下边的属性覆盖，默认为1dp，
	        android:topLeftRadius="integer"
	        android:topRightRadius="integer"
	        android:bottomLeftRadius="integer"
	        android:bottomRightRadius="integer" />
	    <gradient  -- 渐变
	        android:angle="integer"
	        android:centerX="integer"
	        android:centerY="integer"
	        android:centerColor="integer"
	        android:endColor="color"
	        android:gradientRadius="integer"
	        android:startColor="color"
	        android:type=["linear" | "radial" | "sweep"]
	        android:useLevel=["true" | "false"] />
	    <padding
	        android:left="integer"
	        android:top="integer"
	        android:right="integer"
	        android:bottom="integer" />
	    <size    -- 指定大小，一般用在imageview配合scaleType属性使用。大小一般会适配滴
	        android:width="integer"
	        android:height="integer" />
	    <solid    -- 填充颜色，可是是十六进制颜色。（比如想设置半透明效果，直接使用十六就只就OK）
	        android:color="color" />
	    <stroke    -- 指定边框，border，dashWidth和dashGap有一个为0dp则为实线
	        android:width="integer"
	        android:color="color"
	        android:dashWidth="integer"    -- 虚线宽度
	        android:dashGap="integer" />    -- 虚线间隔宽度
	</shape>

### shape

> 形状，可绘制。这必须是根元素

#### 属性

- xmlns:android：字符串，必备。定义XML的命名空间，必须是

`"http://schemas.android.com/apk/res/android"`

- android:shape: 关键字，定义形状类型

|值|描述|
|--|--|
|rectangle|填充包含视图的矩形。这是默认形状。|
|oval|适应包含视图尺寸的椭圆形状。|
|line|跨越包含视图宽度的水平线。此形状需要 [object Object] 元素定义线宽。|
|ring|环形|

***仅当 android:shape="ring" 如下时才使用以下属性：***

- android:innerRadius

尺寸。环内部（中间的孔）的半径，以尺寸值或尺寸资源表示。

- android:innerRadiusRatio

浮点型。环内部的半径，以环宽度的比率表示。例如，如果 android:innerRadiusRatio="5"，则内半径等于环宽度除以 5。此值被 android:innerRadius 覆盖。默认值为 9。

- android:thickness

尺寸。环的厚度，以尺寸值或尺寸资源表示。

- android:thicknessRatio

浮点型。环的厚度，表示为环宽度的比率。例如，如果 android:thicknessRatio="2"，则厚度等于环宽度除以 2。此值被 android:innerRadius 覆盖。默认值为 3。

- android:useLevel

布尔值。如果这用作 LevelListDrawable，则此值为“true”。这通常应为“false”，否则形状不会显示

#### corners

> 为形状产生圆角。仅当形状为矩形时适用。

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:radius="integer" | 尺寸。所有角的半径，以尺寸值或尺寸资源表示。对于每个角，这会被以下属性覆盖。 | 整型  |
| android:topLeftRadius="integer" | 尺寸。左上角的半径 | 整型  |
| android:topRightRadius="integer" | 尺寸。右上角的半径 | 整型  |
| android:bottomLeftRadius="integer" | 尺寸。左下角的半径 | 整型  |
| android:bottomRightRadius="integer" | 尺寸。右下角的半径 | 整型  |

***注：（最初）必须为每个角提供大于 1 的角半径，否则无法产生圆角。如果希望特定角不要倒圆角，解决方法是使用 android:radius 设置大于 1 的默认角半径，然后使用实际所需的值替换每个角，为不希望倒圆角的角提供零（“0dp”）。***

#### gradient

> 指定形状的渐变颜色。

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:angle | 渐变的角度（度）。0 为从左到右，90 为从下到上。必须是 45 的倍数。默认值为 0。 | 整型  |
| android:centerX | 渐变中心的相对 X 轴位置 (0 - 1.0) | 浮点型 (centerColor起始位置) |
| android:centerY | 渐变中心的相对 Y 轴位置 (0 - 1.0) | 浮点型 |
| android:centerColor | 起始颜色与结束颜色之间的可选颜色，以十六进制值或颜色资源表示。() | 颜色  |
| android:endColor | 结束颜色，表示为十六进制值或颜色资源 | 颜色  |
| android:gradientRadius | 渐变的半径。仅在 android:type="radial" 时适用 | 浮点型 |
| android:startColor | 起始颜色，表示为十六进制值或颜色资源 | 颜色  |
| android:type | 要应用的渐变图案的类型，详情见下表 | 关键字 |
| android:useLevel | 如果这用作 LevelListDrawable，则此值为“true” | 布尔值 |

- android:type：

| 值   | 说明  |
| --- | --- |
| linear | 线性渐变。这是默认值。 |
| radial | 径向渐变。起始颜色为中心颜色。 |
| sweep | 流线型渐变 |

#### padding

> 要应用到包含视图元素的内边距（这会填充视图内容的位置，而非形状）。

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:left | 左内边距 | 尺寸  |
| android:top | 上内边距 | 尺寸  |
| android:right | 右内边距 | 尺寸  |
| android:bottom | 下内边距 | 尺寸  |

#### size

> 形状的大小。

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:height | 形状的高度 | 尺寸  |
| android:width | 形状的高度 | 尺寸  |

#### solid

> 用于填充形状的纯色

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:color | 应用于形状的颜色，以十六进制值或颜色资源表示 | 颜色  |

#### stroke

> 形状的笔划中线，描绘最终形状外围的线

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:width | 线宽  | 尺寸  |
| android:color | 线的颜色 | 颜色  |
| android:dashGap | 就像标刻度似得那种短划线,短划线的间距，以尺寸值或尺寸资源表示。仅在设置了 android:dashWidth 时有效。 | 尺寸  |
| android:dashWidth | 每个短划线的大小，以尺寸值或尺寸资源表示。仅在设置了 android:dashGap 时有效 | 尺寸  |

## selector

> 在每个状态变更期间，将从上到下遍历状态列表，并使用第一个与当前状态匹配的项目 —此选择并非基于“最佳匹配”，而是选择符合状态最低条件的第一个项目。

### 总览
```
	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:constantSize=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:variablePadding=["true" | "false"] >
	    <item
	        android:drawable="@[package:]drawable/drawable_resource"
	        android:state_pressed=["true" | "false"]
	        android:state_focused=["true" | "false"]
	        android:state_hovered=["true" | "false"]
	        android:state_selected=["true" | "false"]
	        android:state_checkable=["true" | "false"]
	        android:state_checked=["true" | "false"]
	        android:state_enabled=["true" | "false"]
	        android:state_activated=["true" | "false"]
	        android:state_window_focused=["true" | "false"] />
	</selector>
```
### selector

> 必备。这必须是根元素。包含一个或多个 `<item>`>  元素。

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| xmlns:android | 必备。定义 XML 命名空间，其必须是"http://schemas.android.com/apk/res/android" | 字符串 |
| android:constantSize | 如果可绘制对象报告的内部大小在状态变更时保持不变，则值为“true”（大小是所有状态的最大值）；如果大小根据当前状态而变化，则值为“false”。默认值为 false。 | 布尔值 |
| android:dither | 值为“true”时，将在位图的像素配置与屏幕不同时（例如：ARGB 8888 位图和 RGB 565 屏幕）启用位图的抖动；值为“false”时则停用抖动。默认值为 true。 | 布尔值 |
| android:variablePadding | 如果可绘制对象的内边距应根据选择的当前状态而变化，则值为“true”；如果内边距应保持不变（基于所有状态的最大内边距），则值为“false”。启用此功能要求您在状态变更时处理执行布局，这通常不受支持。默认值为 false。 | 布尔值 |

### item

> 定义要在某些状态期间使用的可绘制对象，如其属性所述。必须是 `<selector>`

| 参数  | 说明  | 类型  |
| --- | --- | --- |
| android:drawable | 必备。引用可绘制对象资源 | 可绘制对象资源 |
| android:state_pressed | 如果在按下对象（例如触摸/点按某按钮）时应使用此项目，则值为“true”；如果在默认的未按下状态时应使用此项目，则值为“false” | 布尔值 |
| android:state_focused | 如果在对象具有输入焦点（例如当用户选择文本输入时）时应使用此项目，则值为“true”；如果在默认的非焦点状态时应使用此项目，则值为“false”。 | 布尔值 |
| android:state_hovered | 如果当光标悬停在对象上时应使用此项目，则值为“true”；如果在默认的非悬停状态时应使用此项目，则值为“false”。通常，这个可绘制对象可能与用于“聚焦”状态的可绘制对象相同。 | 布尔值 |
| android:state_selected | 设置是否选中状态，true表示已选中，false表示未选中 | 布尔值 |
| android:state_checked | 如果在对象已选中时应使用此项目，则值为“true”；如果在对象未选中时应使用此项目，则值为“false” | 布尔值 |
| android:state_checkable | 如果当对象可选中时应使用此项目，则值为“true”；如果当对象不可选中时应使用此项目，则值为“false”。（仅当对象可在可选中与不可选中小部件之间转换时才有用。） | 布尔值 |
| android:state_enabled | 如果在对象启用（能够接收触摸/点击事件）时应使用此项目，则值为“true”；如果在对象停用时应使用此项目，则值为“false”。 | 布尔值 |
| android:state_activated | 如果在对象激活作为持续选择（例如，在持续导航视图中“突出显示”之前选中的列表项）时应使用此项目，则值为“true”；如果在对象未激活时应使用此项目，则值为“false”。 | 布尔值 |
| android:state_window_focused | 如果当应用窗口有焦点（应用在前台）时应使用此项目，则值为“true”；如果当应用窗口没有焦点（例如，通知栏下拉或对话框出现）时应使用此项目，则值为“false”。 | 布尔值 |

***注：请记住，Android 将应用状态列表中第一个与对象当前状态匹配的项目。因此，如果列表中的第一个项目不含上述任何状态属性，则每次都会应用它，这就是默认值应始终放在最后的原因（如以下示例所示）。***
```
	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:state_pressed="true"
	          android:drawable="@drawable/button_pressed" /> *<!-- pressed -->*
	    <item android:state_focused="true"
	          android:drawable="@drawable/button_focused" /> *<!-- focused -->*
	    <item android:state_hovered="true"
	          android:drawable="@drawable/button_focused" /> *<!-- hovered -->*
	    <item android:drawable="@drawable/button_normal" /> *<!-- default -->*
	</selector>
	*<!-- 应用到button上 -->*
	<Button
	    android:layout_height="wrap_content"
	    android:layout_width="wrap_content"
	    android:background="@drawable/button" />

```