深入分析Java反射(七)-简述反射调用的底层实现 - Throwable

深入分析Java反射(七)-简述反射调用的底层实现 - Throwable

## 前提
 
Java反射的API在JavaSE1.7的时候已经基本完善，**但是本文编写的时候使用的是Oracle JDK11**，因为JDK11对于sun包下的源码也上传了，可以直接通过IDE查看对应的源码和进行Debug。

本文主要介绍反射调用的底层实现，当然还没有能力分析JVM的实现，这里只分析到最终Native方法的调用点。底层会依赖到Unsafe类，可以的话可以看下笔者之前写的一篇文章《神奇的魔法类和双刃剑-Unsafe》。

<a id="more"></a>

## 反射调用的底层实现探究

主要考虑下面的情况：

- 属性操作：`java.lang.reflect.Field#set(Object obj, Object value)`和`java.lang.reflect.Field#get(Object obj)`。
- 构造器调用：`java.lang.reflect.Constructor#newInstance(Object ... initargs)`。
- 方法调用：`java.lang.reflect.Method#invoke(Object obj, Object... args)`。

### 处理属性操作的底层实现

属性操作方法`Field#set(Object obj, Object value)`和`Field#get(Object obj)`底层都是委托到`jdk.internal.reflect.FieldAccessor`实现：

```
public interface FieldAccessor {
    
    public Object get(Object obj) throws IllegalArgumentException;

    
    public boolean getBoolean(Object obj) throws IllegalArgumentException;

    
    public byte getByte(Object obj) throws IllegalArgumentException;

    
    public char getChar(Object obj) throws IllegalArgumentException;

    
    public short getShort(Object obj) throws IllegalArgumentException;

    
    public int getInt(Object obj) throws IllegalArgumentException;

    
    public long getLong(Object obj) throws IllegalArgumentException;

    
    public float getFloat(Object obj) throws IllegalArgumentException;

    
    public double getDouble(Object obj) throws IllegalArgumentException;

    
    public void set(Object obj, Object value)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setBoolean(Object obj, boolean z)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setByte(Object obj, byte b)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setChar(Object obj, char c)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setShort(Object obj, short s)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setInt(Object obj, int i)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setLong(Object obj, long l)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setFloat(Object obj, float f)
        throws IllegalArgumentException, IllegalAccessException;

    
    public void setDouble(Object obj, double d)
        throws IllegalArgumentException, IllegalAccessException;
}

```

`FieldAccessor`接口有很多的实现，`FieldAccessor`接口实例是通过`jdk.internal.reflect.ReflectionFactory`这个工厂构造的：

```
    public FieldAccessor newFieldAccessor(Field field, boolean override) {
        checkInitted();

        Field root = langReflectAccess.getRoot(field);
        if (root != null) {
            
            
            if (root.getModifiers() == field.getModifiers() || !override) {
                field = root;
            }
        }
        return UnsafeFieldAccessorFactory.newFieldAccessor(field, override);
    }

```

最终委托到`UnsafeFieldAccessorFactory#newFieldAccessor()`：

```
class UnsafeFieldAccessorFactory {
    static FieldAccessor newFieldAccessor(Field field, boolean override) {
        Class<?> type = field.getType();
        boolean isStatic = Modifier.isStatic(field.getModifiers());
        boolean isFinal = Modifier.isFinal(field.getModifiers());
        boolean isVolatile = Modifier.isVolatile(field.getModifiers());
        boolean isQualified = isFinal || isVolatile;
        boolean isReadOnly = isFinal && (isStatic || !override);
        if (isStatic) {
            
            
            
            UnsafeFieldAccessorImpl.unsafe.ensureClassInitialized(field.getDeclaringClass());

            if (!isQualified) {
                if (type == Boolean.TYPE) {
                    return new UnsafeStaticBooleanFieldAccessorImpl(field);
                } else if (type == Byte.TYPE) {
                    return new UnsafeStaticByteFieldAccessorImpl(field);
                } else if (type == Short.TYPE) {
                    return new UnsafeStaticShortFieldAccessorImpl(field);
                } else if (type == Character.TYPE) {
                    return new UnsafeStaticCharacterFieldAccessorImpl(field);
                } else if (type == Integer.TYPE) {
                    return new UnsafeStaticIntegerFieldAccessorImpl(field);
                } else if (type == Long.TYPE) {
                    return new UnsafeStaticLongFieldAccessorImpl(field);
                } else if (type == Float.TYPE) {
                    return new UnsafeStaticFloatFieldAccessorImpl(field);
                } else if (type == Double.TYPE) {
                    return new UnsafeStaticDoubleFieldAccessorImpl(field);
                } else {
                    return new UnsafeStaticObjectFieldAccessorImpl(field);
                }
            } else {
                if (type == Boolean.TYPE) {
                    return new UnsafeQualifiedStaticBooleanFieldAccessorImpl(field, isReadOnly);
                } else if (type == Byte.TYPE) {
                    return new UnsafeQualifiedStaticByteFieldAccessorImpl(field, isReadOnly);
                } else if (type == Short.TYPE) {
                    return new UnsafeQualifiedStaticShortFieldAccessorImpl(field, isReadOnly);
                } else if (type == Character.TYPE) {
                    return new UnsafeQualifiedStaticCharacterFieldAccessorImpl(field, isReadOnly);
                } else if (type == Integer.TYPE) {
                    return new UnsafeQualifiedStaticIntegerFieldAccessorImpl(field, isReadOnly);
                } else if (type == Long.TYPE) {
                    return new UnsafeQualifiedStaticLongFieldAccessorImpl(field, isReadOnly);
                } else if (type == Float.TYPE) {
                    return new UnsafeQualifiedStaticFloatFieldAccessorImpl(field, isReadOnly);
                } else if (type == Double.TYPE) {
                    return new UnsafeQualifiedStaticDoubleFieldAccessorImpl(field, isReadOnly);
                } else {
                    return new UnsafeQualifiedStaticObjectFieldAccessorImpl(field, isReadOnly);
                }
            }
        } else {
            if (!isQualified) {
                if (type == Boolean.TYPE) {
                    return new UnsafeBooleanFieldAccessorImpl(field);
                } else if (type == Byte.TYPE) {
                    return new UnsafeByteFieldAccessorImpl(field);
                } else if (type == Short.TYPE) {
                    return new UnsafeShortFieldAccessorImpl(field);
                } else if (type == Character.TYPE) {
                    return new UnsafeCharacterFieldAccessorImpl(field);
                } else if (type == Integer.TYPE) {
                    return new UnsafeIntegerFieldAccessorImpl(field);
                } else if (type == Long.TYPE) {
                    return new UnsafeLongFieldAccessorImpl(field);
                } else if (type == Float.TYPE) {
                    return new UnsafeFloatFieldAccessorImpl(field);
                } else if (type == Double.TYPE) {
                    return new UnsafeDoubleFieldAccessorImpl(field);
                } else {
                    return new UnsafeObjectFieldAccessorImpl(field);
                }
            } else {
                if (type == Boolean.TYPE) {
                    return new UnsafeQualifiedBooleanFieldAccessorImpl(field, isReadOnly);
                } else if (type == Byte.TYPE) {
                    return new UnsafeQualifiedByteFieldAccessorImpl(field, isReadOnly);
                } else if (type == Short.TYPE) {
                    return new UnsafeQualifiedShortFieldAccessorImpl(field, isReadOnly);
                } else if (type == Character.TYPE) {
                    return new UnsafeQualifiedCharacterFieldAccessorImpl(field, isReadOnly);
                } else if (type == Integer.TYPE) {
                    return new UnsafeQualifiedIntegerFieldAccessorImpl(field, isReadOnly);
                } else if (type == Long.TYPE) {
                    return new UnsafeQualifiedLongFieldAccessorImpl(field, isReadOnly);
                } else if (type == Float.TYPE) {
                    return new UnsafeQualifiedFloatFieldAccessorImpl(field, isReadOnly);
                } else if (type == Double.TYPE) {
                    return new UnsafeQualifiedDoubleFieldAccessorImpl(field, isReadOnly);
                } else {
                    return new UnsafeQualifiedObjectFieldAccessorImpl(field, isReadOnly);
                }
            }
        }
    }
}

```

这里注意一下属性修饰符的判断：

- isStatic：静态属性，也就是static关键字修饰的属性。
- isFinal：final关键字修饰的属性。
- isVolatile：valatile关键字修饰的属性。
- isQualified：valatile关键字或者final关键字修饰的属性。
- isReadOnly：是否只读属性，final关键字修饰的属性或者static关键字修饰并且不能覆盖(override = false)的属性。

通过上面修饰符做判断，得到最终的`FieldAccessor`实现。这里挑一个例子进行分析，例如**一个普通非静态没有volatile和final关键字修饰属性**最终就会得到`UnsafeObjectFieldAccessorImpl`的实例：

```
class UnsafeObjectFieldAccessorImpl extends UnsafeFieldAccessorImpl {
    UnsafeObjectFieldAccessorImpl(Field field) {
        super(field);
    }

    public Object get(Object obj) throws IllegalArgumentException {
        ensureObj(obj);
        return unsafe.getObject(obj, fieldOffset);
    }

    public void set(Object obj, Object value)
        throws IllegalArgumentException, IllegalAccessException{
        ensureObj(obj);
        if (isFinal) {
            throwFinalFieldIllegalAccessException(value);
        }
        if (value != null) {
            if (!field.getType().isAssignableFrom(value.getClass())) {
                throwSetIllegalArgumentException(value);
            }
        }
        unsafe.putObject(obj, fieldOffset, value);
    }  

    public boolean getBoolean(Object obj) throws IllegalArgumentException {
        throw newGetBooleanIllegalArgumentException();
    }

    public byte getByte(Object obj) throws IllegalArgumentException {
        throw newGetByteIllegalArgumentException();
    }     

    
}    

```

可见`UnsafeObjectFieldAccessorImpl`中除了`get(Object obj)`和`set(Object obj, Object value)`方法，其他方法都是直接抛出IllegalArgumentException。而`get(Object obj)`和`set(Object obj, Object value)`底层分别依赖于`jdk.internal.misc.Unsafe`的`putObject(obj, fieldOffset, value)`和`getObject(obj, fieldOffset)`方法。而属性的内存偏移地址是在`UnsafeObjectFieldAccessorImpl`的父类`UnsafeFieldAccessorImpl`的构造函数中计算出来的：

```
abstract class UnsafeFieldAccessorImpl extends FieldAccessorImpl {
    static final Unsafe unsafe = Unsafe.getUnsafe();

    protected final Field   field;
    protected final long    fieldOffset;
    protected final boolean isFinal;

    UnsafeFieldAccessorImpl(Field field) {
        this.field = field;
        if (Modifier.isStatic(field.getModifiers()))
            fieldOffset = unsafe.staticFieldOffset(field);
        else
            fieldOffset = unsafe.objectFieldOffset(field);
        isFinal = Modifier.isFinal(field.getModifiers());
    }
    
}    

```

这里可以做个小结，属性反射操作`Field`的`setXX`和`getXX`方法最终委托到`jdk.internal.misc.Unsafe`的`putXX`和`getXX`方法，而属性的内存偏移地址是通过`jdk.internal.misc.Unsafe`的`staticFieldBase()`、`staticFieldOffset`和`objectFieldOffset`几个方法计算的。

### 处理构造器调用的底层实现

`Constructor#newInstance()`方法调用依赖到`ConstructorAccessor`：

```
    public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, clazz, modifiers);
        }
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }


public interface ConstructorAccessor {
    
    public Object newInstance(Object[] args)
        throws InstantiationException,
               IllegalArgumentException,
               InvocationTargetException;
}

```

而获取`ConstructorAccessor`实例也是通过反射工厂类`ReflectionFactory`，具体是`ReflectionFactory#newConstructorAccessor`：

```
    public ConstructorAccessor newConstructorAccessor(Constructor<?> c) {
        checkInitted();
        Class<?> declaringClass = c.getDeclaringClass();
        
        if (Modifier.isAbstract(declaringClass.getModifiers())) {
            return new InstantiationExceptionConstructorAccessorImpl(null);
        }
        
        if (declaringClass == Class.class) {
            return new InstantiationExceptionConstructorAccessorImpl
                ("Can not instantiate java.lang.Class");
        }

        
        Constructor<?> root = langReflectAccess.getRoot(c);
        if (root != null) {
            c = root;
        }

        
        if (Reflection.isSubclassOf(declaringClass,
                                    ConstructorAccessorImpl.class)) {
            return new BootstrapConstructorAccessorImpl(c);
        }
        
        if (noInflation && !ReflectUtil.isVMAnonymousClass(c.getDeclaringClass())) {
            return new MethodAccessorGenerator().
                generateConstructor(c.getDeclaringClass(),
                                    c.getParameterTypes(),
                                    c.getExceptionTypes(),
                                    c.getModifiers());
        } else {
            NativeConstructorAccessorImpl acc =
                new NativeConstructorAccessorImpl(c);
            DelegatingConstructorAccessorImpl res =
                new DelegatingConstructorAccessorImpl(acc);
            acc.setParent(res);
            return res;
        }
    }

```

可见最终得到的`ConstructorAccessor`实例为`DelegatingConstructorAccessorImpl`，而`DelegatingConstructorAccessorImpl`只是一个委托实现，底层是调用`NativeConstructorAccessorImpl`：

```
class NativeConstructorAccessorImpl extends ConstructorAccessorImpl {
    private final Constructor<?> c;
    private DelegatingConstructorAccessorImpl parent;
    private int numInvocations;

    NativeConstructorAccessorImpl(Constructor<?> c) {
        this.c = c;
    }

    public Object newInstance(Object[] args)
        throws InstantiationException,
               IllegalArgumentException,
               InvocationTargetException
    {
        
        
        
        if (++numInvocations > ReflectionFactory.inflationThreshold()
                && !ReflectUtil.isVMAnonymousClass(c.getDeclaringClass())) {
            ConstructorAccessorImpl acc = (ConstructorAccessorImpl)
                new MethodAccessorGenerator().
                    generateConstructor(c.getDeclaringClass(),
                                        c.getParameterTypes(),
                                        c.getExceptionTypes(),
                                        c.getModifiers());
            parent.setDelegate(acc);
        }

        return newInstance0(c, args);
    }

    void setParent(DelegatingConstructorAccessorImpl parent) {
        this.parent = parent;
    }
    
    private static native Object newInstance0(Constructor<?> c, Object[] args)
        throws InstantiationException,
               IllegalArgumentException,
               InvocationTargetException;
}

```

`NativeConstructorAccessorImpl#newInstance0()`就是最终构造实例化对象的Native方法。当然有例外的情况，例如非正常调用下，如果构造器的宿主类是一个抽象类，那么最终会返回一个`InstantiationExceptionConstructorAccessorImpl`实例，里面直接抛出InstantiationException异常。

### 处理方法调用的底层实现

`Method#invoke()`调用依赖于`MethodAccessor`：

```

public interface MethodAccessor {
    
    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException;
}

    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException{
        if (!override) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz,
                        Modifier.isStatic(modifiers) ? null : obj.getClass(),
                        modifiers);
        }
        MethodAccessor ma = methodAccessor;             
        if (ma == null) {
            ma = acquireMethodAccessor();
        }
        return ma.invoke(obj, args);
    }

```

获取`MethodAccessor`实例的逻辑和前两节类似，是通过`ReflectionFactory#newMethodAccessor()`：

```
    public MethodAccessor newMethodAccessor(Method method) {
        checkInitted();

        if (Reflection.isCallerSensitive(method)) {
            Method altMethod = findMethodForReflection(method);
            if (altMethod != null) {
                method = altMethod;
            }
        }

        
        Method root = langReflectAccess.getRoot(method);
        if (root != null) {
            method = root;
        }

        if (noInflation && !ReflectUtil.isVMAnonymousClass(method.getDeclaringClass())) {
            return new MethodAccessorGenerator().
                generateMethod(method.getDeclaringClass(),
                               method.getName(),
                               method.getParameterTypes(),
                               method.getReturnType(),
                               method.getExceptionTypes(),
                               method.getModifiers());
        } else {
            NativeMethodAccessorImpl acc =
                new NativeMethodAccessorImpl(method);
            DelegatingMethodAccessorImpl res =
                new DelegatingMethodAccessorImpl(acc);
            acc.setParent(res);
            return res;
        }
    }

```

最终会委托到`NativeMethodAccessorImpl#invoke(Object obj, Object[] args)`：

```
class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private final Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method method) {
        this.method = method;
    }

    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException
    {
        
        
        
        if (++numInvocations > ReflectionFactory.inflationThreshold()
                && !ReflectUtil.isVMAnonymousClass(method.getDeclaringClass())) {
            MethodAccessorImpl acc = (MethodAccessorImpl)
                new MethodAccessorGenerator().
                    generateMethod(method.getDeclaringClass(),
                                   method.getName(),
                                   method.getParameterTypes(),
                                   method.getReturnType(),
                                   method.getExceptionTypes(),
                                   method.getModifiers());
            parent.setDelegate(acc);
        }

        return invoke0(method, obj, args);
    }

    void setParent(DelegatingMethodAccessorImpl parent) {
        this.parent = parent;
    }

    private static native Object invoke0(Method m, Object obj, Object[] args);
}

```

而`NativeMethodAccessorImpl#invoke0()`就是方法调用的最终调用的Native方法。

## 小结

学习知识过程总是阶梯式上升的，JDK中的类库设计也类似这样，如果提前熟悉`Unsafe`类的相关方法，其实反射调用的底层实现也能够相对轻易地理解。属性、构造和方法反射调用底层的实现(**只考虑正常调用的情况下**)如下：

- 对于属性(Field)：`Field#setXX()`和`Field#getXX()`分别对应`Unsafe`的`putXX()`和`getXX()`方法，也就是说完全依赖`Unsafe`中的Native方法。
- 对于构造(Constructor)：`Constructor#newInstance()`底层调用`NativeConstructorAccessorImpl#newInstance0()`。
- 对于方法(Method)：`Method#invoke()`底层调用`NativeMethodAccessorImpl#invoke0()`

(本文完 e-a-20181216 c-1-d)