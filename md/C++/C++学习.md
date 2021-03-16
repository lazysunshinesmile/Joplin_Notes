C++学习

C++学习

C++ 学习记录

类型                 含义                    最小尺寸（字节byte）
-----------------------------------------------------
bool                                未定义
char                                1
wchar_t            宽字节                2
char16_t        Unicode字符            2
char32_t        Unicode字符            4
short                                2
int                                    2
long                                4
long long                            8

1byte = 8 bit

signed 有字符 表示可正可负
unsigned 无字符 >0

'a' 表示char型
"a" 表示字符串型

::variable 显示的调用全局变量variable

int &var1 = var2; 即将var1 和var2 指向了同一个内存地址，var2必须是一个对象，不能是字面化初始值。并且var1与var2类型相同。不能再对&var1再次赋值

int *var1 = &var2; var1存放了var2的地址，var1是指向var2的指针。使用*var= 这种方式赋值时，等号右侧必须是地址。

&var 取变量var的地址
*var 取地址var中保存的值，改变此值，所有指向这个地址的变量的值都会改变。

指针是对象，引用不是。

void* 可以存放任意对象的地址   void* var = &obj;   obj可以为任意对象。

？？？？？？？？？？？？？？？？？？？？？？？？？？？
typedef char *pstring;
const pstring var1 = 0 与
const char *var1 = 0 的区别

pstring = &char 是地址， const pstring 也就是对地址常量，那就是顶层const

const char *var1  = const char var2; 因为 *var1 = var2;都是代表的值。const *var1 就是对值常量，那就是底层const。

？？？？？？？？？？？？？？ ？？？？？？？？？？？？？？？
后者是值是固定的，是底层const。前者是顶层const，指针是个常量。

    int var4 = 8;

    typedef int *pstring;
    const pstring var1 = &var4; //（1）

    *var1 = 78;
    std::cout << *var1;
    //78

    int var9 = 211;
    var1 = &var9;
    std::cout << *var1;
    //报错，var1不能修改。

    int *const var5 = &var4;
    var4 = 99;
    std::cout << *var5;
    // 99

    int var9 = 211;
    var5 = &var9;
    std::cout << *var5;
    //报错，var5是一个常量指针，不能修改。

    //由上面的实验可以看出，（1）处类似定义了一个常量指针，顶层const。即地址值不能修改。

    //------------------------------------------------------------------------

    const int *var2 = 0;
    std::cout << var2;
    // 0

    int var3 = 4;
    const int *var2 = &var3;  //底层const，能修改var2代表的地址，但无法修改*var2
    std::cout << *var2; //4

    *var2 = 8;        //报错，无法修改。

    var3 = 6;        //var3 不是常量，能够修改，修改之后，*var2 也修改了。
    std::cout << *var2;     // 6

    int var6 = 19;
    var2 = &var6;    //改变var2指针指向的地址。
    std::cout << *var2;
    //19，因为指针指向的地址变为了var6的地址。所以var2的值也变了。

从上面的实验得出结论：

如果是底层const，指向的值不能修改。指向的值完全可以通过其他途径修改，只要不通过*var的方式就可以了。底层const： const int *var 对象是个常量

如果是顶层const，指针指向的地址不能修改，内部的值可以随意修改。包括*var的方式。 顶层const： int *const var     指针是个常量

定义的typedef int *pstring;定义了int的别名，再使用const pstring var,就相当于是使用的顶层const,而var是一个常量指针。

使用auto,用初始值确认变量的类型， 声明变量时必须带有初始值。

decltype(f()) m = x;  使用函数f()返回值作为变量m 的类型，但不会实际去调用f();

引用与指针：
引用：对已存变量的别名。int i = 8; &p = i;
    （1）不存在空引用，引用必须链接到一个合法的内存。
    （2）一旦引用被指向了某一个对象，就不能更改为其他对象了。指针随时可以转换成其他任意对象。
    （3）引用必须在创建的时候初始化，指针可以在任何时候初始化。

    int i = 8;
    int &p = i;    p为i的引用
    std::cout << p; //8

    int m = 9;
    p = m;
    std::cout << p;    //9
    std::cout << i;    //9

    此实验说明了p为i的整型引用，p=m这句话只是将m的值拷贝过去，放到了p指向的内存地址中，根据输出i的值也为9可以看出。

cin >> var1遇到空格就会退出， getline(cin, string)遇到换行时退出。cin表示输入流，string表示string类型对象。getline获取到的字符串不包括最后一个换行符。

有符号与无符号的一起参加运算，有符号的会转换为无符号的值继续运算。

+ 两边不能都是字符串字面值。因为字符串字面值不是string类型。

？？？？？？？？？？？？？？？？？？
c='s';
char u = toupper(c);
cout << u;
输出为S；
c = 's';
cout << toupper(c);
输出为83
？？？？？？？？？？？？？？？？？？

const string s = "sdfsdf,dsdf.ssdf";
for (const char &c : s) {
    cout << c;
}
因为s为常量，即地址不能修改，则必须使用常量引用。

    vector<string> v = {"nihao1", "nihao2", "nihao3"};
    for (auto it = v.begin(); it != v.end() && !it->empty() ; it++) {
        if(*it == v[1]) {
            *it = "nnn";
            v.push_back("dddff");
        }

        cout << *it;
    }

    这里只会打印nihao1，因为迭代器循环中的push_back使迭代器失效了

const 和 constexpr 变量之间的主要区别在于：const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。所有 constexpr 变量均为常量，因此必须使用常量表达式初始化。

字符串是一个字符数组，最后一个元素是一个空字符"\0"

int *p[10];  p是储存了10个整型指针的数组。
int &p[10];  因为引用不是对象，所以不存在存储了引用的数组。
int (*pArray)[10] = &arr pArray是指向包含10个整数的数组的指针
int (&pArray)[10] = arr pArray是对包含10个整数的数组的引用。
int *(&array)[10] = arr array是数组的引用，这个数组包含了10个指针。

int *begin = std::begin(arr); 和 int *begin = arr;  这里的begin都是指向arr第一个元素的指针。
int *end = std::end(arr); end指向arr最后一个元素下一位的指针。

int *a = &b[2];
a[2]; 表示b[4];
a[-1]; 表示b[1];
内置的下标运算符可以为负数。

int arr[10] = {...}
vector<int> v(begin(arr),end(arr))  利用数组初始化vector

别名：
using other_name = int[4]
typedef int other_name[4]

++i : 前置版本，优先使用
i++ : 后置版本，尽量不使用

vector<string> v= {"nihoa", "nihao2"};
vector<string>::iterator iter = v.begin();
cout << *iter++; // 和*(iter++)一样。都是先输出*iter的值之后，再iter++;

*iter.empty();先运行iter.empty(),再解引用，所以会报错。
*iter++.empty();相当于(*iter).empty();iter++;

？？？？？？？？？？？？？？？？？？？？？？
    vector<string> v= {"nihoa", "sjdflksdjflk"};
    vector<string>::iterator iter = v.begin();
    cout << ++iter->size();

    报错lvalue required as increment operand

    iter->size() 返回的是一个右值，左值是能获取地址的，右值不能获取地址。

    ++x 与 x++ 假定x的定义为 int x=0;,那么前者是lvalue，后者是rvalue。前者修改自身值，并返回自身；后者先创建一个临时对像，为其赋值，而后修改x的值，最后返回临时对像。

？？？？？？？？？？？？？？？？？？？？？？？？

小转大：unsigned int + int  && unsigned >= int，-> 有符号类型会转化为无符号类型。

        unsigned int + int && unsined < int，-> 比较两者内存空间，本应该转化为有符号的，但如果无符号的空间比有符号的空间大，那就都转化为无符号的。

switch case 语句中，case中的表达式必须是常量表达式。

声明函数时可以设置默认值，如：void func1(int, int = 6, int = 9); 但声明了第一个默认值，往后的所有参数都要设置。
如下情况可以是特殊的：
void func1(int, int, int = 9);
void func1(int = 8, int = 9, int);
第一个声明了默认值，第二个是对第一个声明的补充，这样相当于三个参数都声明了默认值，你调用func1()也不会错。

？？？？？？？？？？？？？？？？？？？？？？？？？
1、
double Sales_data::avg_price() const {}
const double Sales_data::avg_price() const {}
这两个函数分别代表什么意义,有什么作用？应用场景是什么？

？？？？？？？？？？？？？？？？？？？？？？？？？

IO 类型不能被拷贝，所以参数的传递使用引用。又因为流的使用会改变流的内容，所以不能使用常量引用。

？？？？？？？？？？？？？？？？？？？？？？？？？？？？
struct 与class的区别,使用场景。
？？？？？？？？？？？？？？？？？？？？？？？？？？？？

？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？
友元函数的应用场景？需要对作为友元的类或者函数做制定，增大了耦合性。类似于Java的对成员变量私有，对函数公有，能很大程度上降低耦合性。
？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

cin.tie(&cout) 流不能当参数，所以使用引用，这里将cin与cout关联

打开文件的几种mode:
1、in 读
2、out 写，清除已有内容
3、app 写，每次写都是从末尾开始写
4、ate 打开文件即定位到末尾，即读不到东西了，直接跳过。
5、trunc 截断文件，打开文件时，清空文件。
6、binary 以二进制读写文件

多种模式同时使用采用|连接。

fstrm.getline(str, FILE_LENGTH) //char str[100];每行能读取的个数，超过就报错。 FILE_LENGTH 你估计的文件的长度。

vector 和 array都是数组， vectory 大小可变，array大小不可变。
list是双向链表
forward_list 单项列表

数组，连续存储，所以使用下标查看数值很快，但插入和删除，需要集体移位，很慢。
链表，不连续存储，使用指针指向下一个存储位置，所以插入和删除快，但查看数值时，需要从头查找，随意会慢。

？？？？？？？？？？？？？？？？？？？？？？
代码书写规范
？？？？？？？？？？？？？？？？？？？？？？

list<string> l = {"1", "2"};
auto begin = l.begin(); // list<string>::iterator
auto begin = l.rbegin(); //list<string>::reverse_iterator
auto begin = l.cbeign(); //list<string>::const_iterator
auto begin = l.crbegin(); //list<string>::const_reverse_iterator

容器比较，挨个元素比较，小的就小，大的就大

emplace_back 性能比push_back 好。对于增加匿名对象，emplace_back(var1, var2, var3),var1, var2, var3 是初始化class1的参数。那么emplace_back会在目标内存区域构造。push_back会在外面初始化，然后拷贝到容器末尾，再销毁临时变量。对于增加有名对象，两者性能一样。

auto &v = c.back();
v = 1023; //会改变c中最后一个值
auto v = c.back();
v = 1023; //不会改变c中最后一个值

vector 与数组类似，是一段连续的地址，首地址不会改变。
list 是链表不是连续的地址
如下是应用：
    vector<User> v = {User("xiang", 12), User("sun", 11)};
    auto u5 = v.end() -1;
    auto u6 = v.erase(u5);     //该方法删除u5，并指向u5的下一个元素。
    if(u6 == v.end()) {
        cout << "equal" << endl;  //输出equal
    }
    cout << (*u6).getName()<< endl;    //输出sun

vector调用了erase方法，会把最后一个user删除。其实是把第二个user存储地址作为了end()的返回。但本身第二个user对象并没有擦除。所以再调用(*u6).getName() 还是能输出数据。但list链表形式的就不一样了：

    list<User> v = {User("xiang", 12), User("sun", 11)};
    auto u5 = --v.end();
    auto u6 = v.erase(u5);
    cout << (*u6).getName()<< endl;        //崩溃
    for(User u : v) {
        cout << u.getName() <<endl;
    }
链表地址不变，改变了链表指向，所以第二个user相当于被删除了。

str1.replace(int var1, int var2, string var3)
从var1开始，替换var2个字符，替换为var3

？？？？？？？？？？？？？？？？？？？？？？？？？？/
equal(v1.cbegin(), v1.cend(), v2.cbegin())
该方法必须v2的长度大于v1吗？
测试无误，但书中说必须，不然会报错。
？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

unique(v1.begin(), v1.end());只会校验相邻字符的重复。

？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？
void printWords(vector<string> v) {
    for(auto str : v) {
        cout << str << endl;
    }
}

void elimDups(vector<string> &v) {
    sort(v.begin(),v.end());
    printWords(v);
    cout << "----" <<endl;
    auto end = unique(v.begin(), v.end());
    // v.erase(end, v.end());
    // cout << *end;
    printWords(v);
}

int main()
{
    vector<string> vec = {"3333", "34222", "34222", "3333","sdfsdf","sdfsdf"};
    elimDups(vec);
}

输出：
3333
3333
34222
34222
sdfsdf
sdfsdf
----
3333
34222
sdfsdf
34222

sdfsdf

这个程序为什么输出是这样的？为什么会把字符串"3333"置为了空。而且如果把上段程序中的两个字符串"34222"改为"222",那置为空就是"222"不是"3333"。

？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？/
    vector<int> v = { 1,2,4,-1,-5,-76};

    transform(v.begin(), v.end(), v.begin(), [](int i) {
        if( i < 0)
            return -i;
        if(i > 0)
            return i;
    });

这段程序书上说有误，需要指明返回类型，但实际测试无误。
？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

find(beg, end, val) 在beg到end范围内，查找第一次出现==val的位置
find_if(beg, end, pred) 在beg到end范围内，查找第一次令pred为真的元素

迭代器分为输入，输出，前向，后向，双向

在map存储key时，会将key从小到大依次排序,lower_bound(var)返回第一个不小于var的元素的迭代器，upper_bound(var),返回第一个大于var的元素。

内置指针：int*，智能指针：shared_ptr

（1）不使用相同的内置指针值初始化（或reset)
（2）不delete get()返回的指针
（3）不使用get()初始化或reset另一个智能指针
（4）如果你使用get()返回的指针，记住最后一个对应的智能指针销毁后，你的指针就变为无效了。
（5）如果你使用智能指针管理的资源不是new 分配的内存，记住传递给他一个删除器。

智能指针的作用就是能够自动释放内存
动态内存就是可以申请一片内存给你存储数据手动delete。
两者配合使用shared_ptr<int> k = make_shared(new int(12));或者shared_ptr<int> k(new int)

delete p;    //p必须指向一个动态分配的对象或为空
delete [] pa;    //pa必须指向一个动态分配的数组或为空

explict:不能发生隐式转换

拷贝构造函数
MyClass(const MyClass&) {
    ....
}
参数是常量的本类的引用，不一定非要常量。

合成拷贝赋值运算符：
MyClass& MyClass::operator=(const MyClass& m) {
    var1 = m.var1;
    var2 = m.var2;
    var3 = m.var3;
    return *this;
}
返回一个=左侧对象的引用。

析构函数：
~MyClass() 没有返回值，没有参数，与类名一致

一个对象只有在所有指向他的指针销毁了，才会销毁。
当一个类定义了析构函数，肯定要定义他的拷贝构造函数和拷贝赋值运算符

!!!!!!!!!!!!!!!!!!!!!!!!!一个类使用了拷贝构造函数，一定会要拷贝赋值函数。要不然使用MyClass() 和= 两种方式初始化就不一致了。 反之亦然。！！！！！！！！！！！！！！！！！！！！！！！

=delete必须出现在第一次声明的时候，=default 可以在类外定义的时候写上。
MyClass(const MyClass&) = delete     //阻止拷贝
MyClass& operator=(const MyClass&) = delete //组织赋值
析构函数不能删除

当一个类中有引用成员时，合成拷贝赋值运算符被定义为删除的。

class myclass {
public:
    int& getM(){return var;}
private:
    int var;
}

auto k = myclass.getM();      k的类型是int，改变k并不能改变var
int& k = myclass.getM();    k的类型是int&，改变k能改变var

string getContent() const{}  表示调用这个函数的对象必须是const的

int i = 42;
int &r = i;                    //正确：r引用i
int &&rr = i;                //错误，不能将一个右值引用绑定到左值上
int &r2 = i*42;                //错误：i*42是一个右值
const int &r3 = i*42;        //正确：我们可以将一个const的引用绑定到一个右值上
int &&rr2 = i*42;            //正确：将rr2绑定到乘法结果上。

只有当一个类没有定义任何自己版本的拷贝控制成员，且类的每个非static数据成员都可以移动时，编译器才会为它合成移动构造函数或移动赋值运算符。

通过在类代码中小心的使用move，可以大幅度提升性能，而如果随意在普通用户代码（与类实现代码相对）中使用移动操作，很可能莫名其妙的，难以查找的错误，而难以提升应用性能。

（在移动构造函数何和移动赋值运算符这些类实现代码之外的地方，只有当你确信需要进行移动操作且移动操作是安全的，才可以使用std::move。注意最好不要使用move,而要使用std::move）

如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符。

通常情况下，不应该重载逗号，取地址，逻辑与和逻辑或运算符。

前置--/++ 返回一个引用，没有参数，--需要先-后检查是否合法，++需要先检查后+。
后置--/++ 返回一个值，有一个不用的参数int
A A::operator++(int) {
    A ret = *this;        //记录当前值
    ++*this;            //向前移动一个元素，前置++会检查递增的有效性
    return ret;            //返回之前的状态。
}

处理显式地想bool类型的转换外，我们应该尽量避免定义类型转换函数并尽可能限制那些“显然正确”的非显示构造函数。

定义了纯虚函数的类类似于抽象类，每个子类要去实现该方法。

关于signed与Unsigned之间相互转换的问题好多人都容易糊涂，其实记住两点：
1. Unsigned赋给signed 将补码直接赋给signed 高位补0，若signed位数不足则只填充Unsigned的低位。
2. signed赋给Unsigned 将补码直接赋给Unsigned高位按照signed的符号位填补，若signed的位数不足则只填充signed的低位。

类设计者工具
13章：440-486/46：拷贝控制
14章：490-523/33：重载运算与类型转换
15章：526-575/49：面向对象程序设计
16章：578-630/52：模板与泛型编程

高级主题
17章：636-680/44：标准库特殊设施
18章：683-722/39：用于大型程序的工具
19章：725-762/37：特殊工具与技术