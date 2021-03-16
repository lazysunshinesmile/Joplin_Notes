深入分析Java反射(二)-数组和枚举 - Throwable

深入分析Java反射(二)-数组和枚举 - Throwable

## 前提
 
Java反射的API在JavaSE1.7的时候已经基本完善，**但是本文编写的时候使用的是Oracle JDK11**，因为JDK11对于sun包下的源码也上传了，可以直接通过IDE查看对应的源码和进行Debug。

本文主要介绍反射中可能用到的两个比较特殊的类型，数组和枚举，分别对应`java.lang.reflect.Array`和`java.lang.Enum`，后者其实并不是反射类库包中的类，但是反射的基础类库里面有使用枚举类型的方法。

<a id="more"></a>

## 数组类型

数组是一种包含固定数量的相同类型组件(Component)的引用类型对象，也就是说数组的长度是不可变，它的每个元素都是相同类型的。创建数组实例需要定义数组的长度和组件的类型。**数组是由Java虚拟机实现(这一点很重要，这就是为什么JDK类库中没有数组对应的类型的原因，array也不是Java中的保留关键字，操作数组的底层方法都是native方法)**，数组类型只有继承自`java.lang.Object`的方法，数组的`length`方法实际上并不属于数组类型的一部分，数组的`length`方法其实最终调用的是`java.lang.reflect.Array#getLength()`，注意到这个方法是native方法。`java.lang.reflect.Array`是基于反射操作数组的核心类。

使用非反射方式创建数组实例的过程如下：

```
fully_qualified_class_name[] variable_name  = {val1，val2，val3，...};

fully_qualified_class_name[] variable_name = new fully_qualified_class_name[${fix_length}];

例如：int[] arr = new int[10];

```

使用反射方式就是使用`java.lang.reflect.Array`中的相关方法：

```
Class<?> c = Class.forName(cName);
Object o = Array.newInstance(c, n);
for (int i = 0; i < n; i++) {
    String v = cVals[i];
    Constructor ctor = c.getConstructor(String.class);
    Object val = ctor.newInstance(v);
    Array.set(o, i, val);
}
Object[] oo = (Object[]) o;

```

下面列举一下`java.lang.reflect.Array`中的方法：

| 方法  | 功能  |
| --- | --- |
| static Object newInstance(Class&lt;?&gt; componentType, int length) | 指定组件类型和数组固定长度创建一维数组 |
| static Object newInstance(Class&lt;?&gt; componentType, int… dimensions) | 指定组件类型和多个固定长度创建多维数组，维度的最大值为255 |
| static native int getLength(Object array) | 获取数组长度 |
| static native Object get(Object array, int index) | 通过下标访问数组元素 |
| static native void set(Object array, int index, Object value) | 通过下标设置数组元素 |

这里省略了一部分对于`Int`、`Boolean`等原始类型的Setter和Getter方法。

在`java.lang.Class`和数组相关的方法：

| 方法  | 功能  |
| --- | --- |
| native boolean isArray() | 判断类型是否数组类型 |
| Class&lt;?&gt; getComponentType() | 如果是数组类型则返回其组件类型，否则返回null |

这里举个例子加深下印象：

```
public class ArrayCreationMain {

	
	private static String R = "java.math.BigInteger[] bi = {123,234,345}";
	private static final String[] S = new String[]{"123", "234", "345"};

	public static void main(String[] args) throws Exception {
		Class<BigInteger> componentType = BigInteger.class;
		Object arrayObject = Array.newInstance(componentType, 3);
		for (int i = 0; i < S.length; i++) {
			String each = S[i];
			Constructor<BigInteger> constructor = componentType.getConstructor(String.class);
			BigInteger value = constructor.newInstance(each);
			Array.set(arrayObject, i, value);
		}
		Object[] result = (Object[]) arrayObject;
		System.out.println(String.format("%s[] = %s", componentType, Arrays.toString(result)));
		int length = Array.getLength(arrayObject);
		System.out.println("Length = " + length);
		for (int i = 0; i < length; i++) {
			System.out.println(String.format("index = %d,value = %s", i, Array.get(arrayObject, i)));
		}
		Class<?> arrayObjectClass = arrayObject.getClass();
		System.out.println("Is array type:" + arrayObjectClass.isArray());
		System.out.println("Component type:" + arrayObjectClass.getComponentType());
	}
}

```

运行后输出：

```
class java.math.BigInteger[] = [123, 234, 345]
Length = 3
index = 0,value = 123
index = 1,value = 234
index = 2,value = 345
Is array type:true
Component type:class java.math.BigInteger

```

需要注意的是，`java.lang.reflect.Array`中的Setter和Getter方法如果越界操作数组元素，会抛出`ArrayIndexOutOfBoundsException`，通过Setter设置和数组初始化时候的组件类型不一致的元素会抛出`IllegalArgumentException`。

### 细议数组类型

前面说到了数组类型的一些基础特性，这里补充一些比较高级的使用方法。

**创建特定元素类型的数组：**

因为Java泛型擦除的问题，实际上我们使用`Array#newInstance`方法只能得到一个Object类型的结果实例，其实这个结果实例的类型就是`ComponentType[]`，这里只是返回了它的父类(Object)类型实例，因此我们可以直接强转，例如：

```
String[] strArray = (String[]) Array.newInstance(String.class, 3);

```

**获取数组类型：**

在非反射方式下，我们可以通过`数组实例.class`通过class字面量直接获取数组类型，例如：

```
Class stringArrayClass = String[].class;

```

反射条件下，可以通过`Class.forName()`获取数组类型，但是调用此方法的时候有个限制，类名必须使用JVM可以识别的签名形式，就是`[L${ComponentType};`，注意`Class.forName()`无法获取原始类型(如int、boolean)的类型，例如：

```

Class stringArrayClass = Class.forName("[Ljava.lang.String;");


Class intClass1 = Class.forName("I");
Class intClass2 = Class.forName("int");

```

**获取数组元素(组件)类型：**

目前获取数组组件类型只能通过数组类型实例去调用`Class#getComponentType()`。

## 枚举类型

枚举是一种语言结构(Language Construct)，用于定义可以使用一组固定的名值对表示的类型安全的枚举(原文是：An enum is a language construct that is used to define type-safe enumerations which can be used when a fixed set of named values is desired)。所有枚举都继承自`java.lang.Enum`。枚举可以包含一个或者多个枚举常量，这些枚举常量都是该枚举的实例。枚举的声明其实和一个普通的Class的声明相似，因为它可以包含字段、方法和构造函数之类的成员。

因为枚举就是普通的Java类，因此反射相关类库中并没有添加一个`java.lang.reflect.Enum`类型，反射中的API和枚举相关的有：

- `boolean java.lang.Class#isEnum()`：判断类型是否枚举类型。
- `T[] java.lang.Class#getEnumConstants()`：获取类型中所有的枚举常量。
- `boolean java.lang.reflect.Field#isEnumConstant()`：判断属性是否枚举类型。

如果实例中的成员属性为枚举，那么枚举的反射操作实际上就是`java.lang.reflect.Field`的相关操作。

举个例子：

```
public class EnumerationMain {

	enum Color {
		RED, BLACK, BLUE
	}

	public static class ColorHolder {

		private Color color = Color.BLACK;

	}

	public static void main(String[] args) throws Exception {
		Class<Color> colorClass = Color.class;
		System.out.println("Color class is enum:" + colorClass.isEnum());
		System.out.println("Color values:" + Arrays.toString(colorClass.getEnumConstants()));
		ColorHolder colorHolder = new ColorHolder();
		Class<ColorHolder> holderClass = ColorHolder.class;
		Field field = holderClass.getDeclaredField("color");
		field.setAccessible(true);
		System.out.println("Old color:" + field.get(colorHolder));
		field.set(colorHolder, Color.RED);
		System.out.println("New color:" + field.get(colorHolder));
	}
}

```

运行后输出：

```
Color class is enum:true
Color values:[RED, BLACK, BLUE]
Old color:BLACK
New color:RED

```

之前写过一篇文章《JDK中枚举的底层实现》，从枚举类的字节码翻译出类的代码逻辑，这里翻出来那个例子(手机操作系统枚举)说一下：

```
public enum PhoneOsEnum {

	
	ANDROID(1, "android"),

	
	IOS(2, "ios");


	private final Integer type;
	private final String typeName;

	PhoneOsEnum(Integer type, String typeName) {
		this.type = type;
		this.typeName = typeName;
	}

	public Integer getType() {
		return type;
	}

	public String getTypeName() {
		return typeName;
	}
}

```

这个是我们使用Java的关于枚举的语法创建出来的枚举类型，是编译前我们看到的Java类文件，实际上，编译完成之后，枚举类型会变成一个普通的Java类，它有以下特点：

- 1、枚举类型会变成一个普通Java类，这个Java类会继承`java.lang.Enum`，并且把自身类型作为泛型参数类型，构造函数中必定包含name(字符串类型String)、ordinal(整型int)参数，因为父类`java.lang.Enum`的构造要求传入这两个参数。
- 2、所有的枚举成员属性都变成`static final`修饰的在第1步中提到的Java类的实例，属性的名称和原来枚举的名字一致，实例在静态代码块中创建。
- 3、新增了一个`static final`修饰的第1步中提到的Java类的数组实例，名称为`$VALUES`，此数组在静态代码块中创建，基于此数组还新增了一个静态方法`values()`，此方法就是直接返回数组的克隆。

也就是上面提到的`PhoneOsEnum`在编译完成之后会变成：

```
public final class PhoneOsEnumeration extends Enum<PhoneOsEnumeration> {

	public PhoneOsEnumeration(String name, int ordinal, Integer type, String typeName) {
		super(name, ordinal);
		this.type = type;
		this.typeName = typeName;
	}

	public Integer getType() {
		return type;
	}

	public String getTypeName() {
		return typeName;
	}

	public static PhoneOsEnumeration[] values() {
		return $VALUES.clone();
	}

	public static PhoneOsEnumeration valueOf(String name) {
		return Enum.valueOf(PhoneOsEnumeration.class, name);
	}
	
	private final Integer type;
	private final String typeName;
	public static final PhoneOsEnumeration ANDROID;
	public static final PhoneOsEnumeration IOS;
	private static final PhoneOsEnumeration[] $VALUES;

	static {
		ANDROID = new PhoneOsEnumeration("ANDROID", 0, 1, "android");
		IOS = new PhoneOsEnumeration("IOS", 1, 2, "ios");
		$VALUES = new PhoneOsEnumeration[]{ANDROID, IOS};
	}
}

```

实际上，如果你直接编写一个Java类去继承`java.lang.Enum`会编译报错，也就是Java希望把枚举的行为和特性交由自身控制而不是开发者去控制，从编译层面控制枚举的类型安全。如果细心一点会发现，枚举中`valueOf(String name)`也是由`java.lang.Class`提供的，追溯到最里层是`T[] java.lang.Class#getEnumConstants()`方法，其实有可能在构造`$VALUES`属性的时候也是用到这个方法，这一点就没有深究下去，编译层面的东西可能会牵涉很多方面的知识，还没有到达那种水平。

## 小结

数组和枚举在Java中的使用频率也是比较高的，特别是算法或者框架中，本文尝试从反射角度介绍这两个类型的使用方式，掌握它们对数组或者枚举的使用有很大的帮助。

(本文完 e-2018122-c-1-d r-2018124-c-1-d)