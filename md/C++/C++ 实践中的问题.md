C++ 实践中的问题

C++ 实践中的问题

**1、如何写初始化失败？**

**2、为什么不是类的方法就不能使用宏**
错误，是能使用的。原因如下：

#define  MY_LOGE(fmt, arg...) CAM_LOGE("(%d)(%s)[%s] " fmt, ::gettid(), getShotName(), __FUNCTION__, ##arg)

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

说明：由上可以看出，getShotName()函数是A的父类B 的成员函数。在A 自己的方法(A())中，能够直接调用父类B 的方法。但在非A 的成员函数（isRight()）中，不能直接调用getShotName()方法。

**3、调用指定父类的成员函数：**

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

**4、多态使用**
class A {
     virtual public getName() {
         return "A";
     }
}
class B : public A{
    public getName() {
         return "B";
     }
}

class C : public A {
    public getName() {
        return "C";
     }
}

int main() {
    A* a;
    B b = B();
    C c = C();
    a = &b;
    cout << a->getName();      //输出 B
    a = &c;
    cout << a->getName();     //输出 C
}

注意在父类 A 中，需要被不同子类重写的方法getName()，要写成 virtual 即定义为虚函数。要不然在main函数中的调用输出的始终是A。因为这里会被编译器设置成父类的getName()，