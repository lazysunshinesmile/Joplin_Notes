深入分析Java反射(六)-反射调用异常处理 - Throwable

深入分析Java反射(六)-反射调用异常处理 - Throwable

## 前提
 
Java反射的API在JavaSE1.7的时候已经基本完善，**但是本文编写的时候使用的是Oracle JDK11**，因为JDK11对于sun包下的源码也上传了，可以直接通过IDE查看对应的源码和进行Debug。

本文主要介绍一个使用反射一定会遇到的问题-反射调用异常处理。

<a id="more"></a>

## 反射调用异常处理

反射调用出现异常的方法主要考虑下面的情况：

- 属性操作：`java.lang.reflect.Field#set(Object obj, Object value)`和`java.lang.reflect.Field#get(Object obj)`。
- 构造器调用：`java.lang.reflect.Constructor#newInstance(Object ... initargs)`。
- 方法调用：`java.lang.reflect.Method#invoke(Object obj, Object... args)`。

### 处理属性操作异常

先看设置属性的方法：

```
public void set(Object obj, Object value) throws IllegalArgumentException, IllegalAccessException

```

实际上，通过方法注释可以得知会抛出四种异常：

- IllegalAccessException：非法访问异常，**注意它是检查(checked)异常**，也就是需要显示捕获，此异常会在修饰符禁用访问的时候抛出，可以通过`setAccessible(true)`抑制修饰符检查来避免抛出此异常。
- IllegalArgumentException：非法参数异常，它是运行时异常，当入参实例obj不是当前Field所在类(包括父类、子类和接口)的时候会抛出此异常。
- NullPointerException：空指针异常，当入参实例obj为null的时候会抛出此异常。
- ExceptionInInitializerError：初始化器调用异常导致的错误，如果由于`set(Object obj, Object value)`方法引发的初始化失败会包装成ExceptionInInitializerError，此异常的父类为Error，常见的发生情况就是静态成员或者静态代码块依赖到反射属性设置。

前面三种异常都很好理解，最后一个ExceptionInInitializerError可能有点陌生，它的抛出条件是：在静态代码块初始化解析过程总抛出异常或者静态变量初始化的时候抛出异常。笔者尝试了很多例子都没办法造出案例，从Stackoverflow找到一个例子：

```
public class Example {
    public static void main(String[] args) throws Exception {
        Field field = Fail.class.getDeclaredField("number");
        field.set(null, 42); 
    }
}

class Fail {
    static int number;
    static {
        boolean val = true;
        if (val)
            throw new RuntimeException(); 
    }
}

```

简单来说就是：静态代码块和静态变量的初始化顺序和它们在类文件编写的顺序是一致的，如果一个类未初始化直接使用它的静态代码块和静态变量通过`Field#set(Object obj, Object value)`调用就会出现ExceptionInInitializerError异常。

属性的获取方法抛出的异常和设置值方法是一致的，这里不做详细展开：

```
public Object get(Object obj) throws IllegalArgumentException, IllegalAccessException

```

### 处理构造器调用异常

构造器调用主要是用于对象的实例化，先看`newInstance`方法的签名：

```
public T newInstance(Object ... initargs) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException

```

- InstantiationException：实例化异常，抛出此异常的一般情况是：当前构造所在类型为一个抽象类型。
- IllegalAccessException：非法访问异常。
- IllegalArgumentException：非法参数异常，下面的情况会抛出此异常：参数数量或者类型不匹配，参数列表为原始类型但是实际使用了包装类型、参数列表为原始类型但是实际使用了包装类型、构造所在的类是枚举类型等。
- InvocationTargetException：目标调用异常，这个是需要处理的重点异常，在下一节"处理方法调用异常"详细探讨。

这里只举个例子说明一下InstantiationException出现的场景：

```
public abstract class AbstractSample {

	public AbstractSample() {
	}

	public static void main(String[] args) throws Exception{
		Constructor<AbstractSample> declaredConstructor = AbstractSample.class.getDeclaredConstructor();
		declaredConstructor.newInstance();
	}
}

```

像上面的抽象类`AbstractSample`包含一个默认的公有构造，使用`Constructor#newInstance()`会抛出InstantiationException异常：

```
Exception in thread "main" java.lang.InstantiationException
	at java.base/jdk.internal.reflect.InstantiationExceptionConstructorAccessorImpl.newInstance(InstantiationExceptionConstructorAccessorImpl.java:48)
	at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:490)
	at club.throwable.jdk.sample.reflection.reflect.AbstractSample.main(AbstractSample.java:18)

```

### 处理方法调用异常

方法调用是反射中使用频率最高的反射操作，主要是`Method#invoke(Object obj, Object... args)`方法：

```
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException

```

主要包括以下几种异常：

- IllegalAccessException：非法访问异常。
- IllegalArgumentException：非法参数异常，下面的情况会抛出此异常：入参obj并不是当前实例方法对应的实例对象、参数数量或者类型不匹配，参数列表为原始类型但是实际使用了包装类型、参数列表为原始类型但是实际使用了包装类型等等。
- NullPointerException：空指针异常，入参obj为null时候会抛出此异常。
- ExceptionInInitializerError：初始化器调用异常导致的错误。
- InvocationTargetException：目标调用异常。

重点看InvocationTargetException(继承自ReflectiveOperationException，而ReflectiveOperationException继承自Exception，**也就是它是checked异常，必须显式捕获**)：

```
public class InvocationTargetException extends ReflectiveOperationException {

    private static final long serialVersionUID = 4085088731926701167L;
    
    
    private Throwable target;

    public InvocationTargetException(Throwable target) {
        super((Throwable)null);  
        this.target = target;
    }

    public InvocationTargetException(Throwable target) {
        super((Throwable)null);  
        this.target = target;
    } 

    public Throwable getTargetException() {
        return target;
    }

    public Throwable getCause() {
        return target;
    }    
}    

```

从注释中得知：方法(Method)或者构造(Constructor)调用异常会抛出此InvocationTargetException异常，用于包装源异常，源异常实例作为目标被InvocationTargetException通过成员target持有，可以通过`InvocationTargetException#getTargetException()`或者`InvocationTargetException#getCause()`获取原始的目标异常。这里注意到，InvocationTargetException在覆盖父类构造的时候使用了null，所以调用其`getMessage()`方法会得到null。

举个例子：

```
public class InvocationTargetExceptionMain {

	public void method() {
		throw new NullPointerException("Null");
	}

	public static void main(String[] args) throws NoSuchMethodException, SecurityException {
		InvocationTargetExceptionMain main = new InvocationTargetExceptionMain();
		Method method = InvocationTargetExceptionMain.class.getDeclaredMethod("method");
		try {
			method.invoke(main);
		} catch (IllegalAccessException e) {
			
		} catch (InvocationTargetException e) {
			System.out.println("InvocationTargetException#message:" + e.getMessage());
			if (e.getTargetException() instanceof NullPointerException) {
				NullPointerException nullPointerException = (NullPointerException) e.getTargetException();
				System.out.println("NullPointerException#message:" + nullPointerException.getMessage());
			}
		}
	}
}

```

运行后输出：

```
InvocationTargetException#message:null
NullPointerException#message:Null

```

构造器`Constructor#newInstance()`中抛出InvocationTargetException的场景是类似的。

## 小结

在反射操作中，方法调用的频次是最高的，其次是通过构造器实例化对象。需要重点关注这两个地方的异常处理，特别是异常类型`InvocationTargetException`，紧记需要获取原始目标异常类型再进行判断，否则很容易导致逻辑错误(最近笔者在做一个功能的时候刚好踩了这个坑)。

(本文完 e-a-20181215 c-2-d)