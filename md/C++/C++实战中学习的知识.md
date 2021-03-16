C++实战中学习的知识

C++实战中学习的知识

*知识技巧：*
一般是不用指针的指针的，将指针想象成一个对象，你想要保存这个对象的值，就需要传入这个指针的引用(&data).

class A{
public:
void getData(char data);
}

int main() {
    char* data;
    A a;
    a.getData(&data);
}

*问题：*

1. Fatal signal 11 (SIGSEGV), code 1, fault addr 0x0 in tid 26021 (ndkapplication1)

这个可能是你某个参数没有初始化，或者使用的是{0}进行的初始化。要使用new

2. 为什么不是类的方法就不能使用宏
错误，是能使用的。那里错误的原因如下：
```
#define MY_LOGE(fmt, arg...) CAM_LOGE("(%d)(%s)[%s] " fmt, ::gettid(), getShotName(), __FUNCTION__, ##arg)

class B {
public:
    string getShotName();
}

class A : public B {
public:
    A() {
          //无错。
        MY_LOGE(....);
     }
}

extern
bool
isRight() {

    // 编译报错：error: use of undeclared identifier 'getShotName'
    MY_LOGE(....);
}
```
说明：由上可以看出，getShotName()函数是A的父类B 的成员函数。在A 自己的方法(A())中，能够直接调用父类B 的方法。但在非A 的成员函数（isRight()）中，不能直接调用getShotName()方法。

3. 调用指定父类的成员函数：
```
class A {
    public:
            string getName();
}

class B {
    public:
          string getName();
}

class C {
    public:
          string getName();
}

class D : public A, public B, public C {
    public:
          string getParentName() {
                A::getName();
         }
}

int main() {
    D d;
    d.B::getName();
}
```

4. explict 作用

能够避免一些不合时宜的隐式类型转换。google 规范是在单参数的构造函数必须加上explict。只有极少数情况下拷贝构造函数可以不声明称explicit。例如作为其他类的透明包装器的类。

5. 打印size_t类型变量

```
    ALOGD("%s, cameraIds size=%zu", __FUNCTION__, cameraIds->size());
```