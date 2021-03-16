你真的理解AIDL中的in，out，inout么

这其实是一个很小的知识点，大部分人在使用AIDL的过程中也基本没有因为这个出现过错误，正因为它小，所以在大部分的网上关于AIDL的文章中，它都被忽视了——或者并没有，但所占篇幅甚小，且基本上都是官方文档的译文，译者读者其实都不知其然。这几天在研究AIDL，偏偏我又是个执拗的性子，遇着不清不楚的东西就是想把它捋清楚，就下了些功夫研究了下AIDL中的定向tag，研究了下它的 in ， out ， inout 。

整理而成此博文。

## 1，概述

首先要说的是定向tag是AIDL语法的一部分，而 in ， out ， inout 是三个定向tag，所以读者要有一定的对于Android中AIDL的了解，关于AIDL相关的知识大家可以参考这篇博文：[Android：学习AIDL，这一篇文章就够了(上)](http://blog.csdn.net/luoyanglizi/article/details/51980630) 或者[Android：学习AIDL，这一篇文章就够了(上) - 程序园](../../Android/Android_NDK/Android：学习AIDL，这一篇文章就够了%28上%29%20-%20程序园.md)。 另外，这篇文章基本上可以说是我研究这个东西的心路历程，可能会有些絮叨，请各位看官见谅。

## 2，官方文档

Android官网上在讲到AIDL的地方关于定向tag是这样介绍的：

> All non-primitive parameters require a directional tag indicating which way the data goes . Either in , out , or inout . Primitives are in by default , and connot be otherwise .

直译过来就是：**所有的非基本参数都需要一个定向tag来指出数据流通的方式，不管是 in , out , 还是 inout 。基本参数的定向tag默认是并且只能是 in 。**对于这个定向tag我的心里是有一些疑问的。首先，数据流通的方式是指什么？其次， in ， out ， inout 分别代表了什么，它们有什么区别？很显然，官方文档并没有把这些东西交代清楚。那么接下来我就只能自己把这些问题搞清楚了。

本着不重复造轮子的原则，我首先在 Google 上查找了一下这方面的资料，看能不能有比较好的答案，结果确实找到了一些说法——但是进过论证，似乎都有一些漏洞。所以没办法，只能自己开始着手研究它是怎么回事了。

## 3，开始研究

### 3.1，输入输出？NO！

首先第一个跑到我脑海里的猜测就是：in表示输入，也即方法的传参，out表示输出，也即方法的返回值。有这样的猜测很合情合理，但是这样的猜测很不合情合理。原因如下：

- 文档里说了，基本参数的定向tag默认且只能是 in ，但是很显然，基本参数既有可能是方法的传参，也有可能是方法的返回值，所以这个猜测本身就站不住脚。
- 如果 in 表示输入， out 表示输出，那 inout 应该表示什么？
- 进过实测，定向tag只能用来修饰AIDL中方法的输入参数，并不能修饰其返回值。

综合以上几点考虑，基本可以排除这种猜测的可能性。

### 3.2，way？way！

排除掉上面的想法后，我开始进一步猜测 in ， out ，inout 可能代表的意义——在某一个瞬间我灵光一闪：除了输入输出，in ，out 还总是被用来表示数据的流向！同时我惊觉，似乎我对官方文档的理解有一些偏差：way有方法的意思，但是它也有道路的意思！如果按照道路理解，那么官网的译文就应当是：**所有的非基本参数都需要一个定向tag来指出数据的流向，不管是 in , out , 还是 inout 。基本参数的定向tag默认是并且只能是 in 。**

如果按照这个意思的话，似乎它们的含义就很清晰了：in 与 out 分别表示客户端与服务端之间的两条单向的数据流向，而 inout 则表示两端可双向流通数据。基于这种猜测，我设计了一个实验来验证它，AIDL文件是这样的：

```

package com.lypeer.ipcclient;

parcelable Book;
```

```

package com.lypeer.ipcclient;
import com.lypeer.ipcclient.Book;

interface BookManager {    
    
    List<Book> getBooks();

    
    Book addBookIn(in Book book);
    Book addBookOut(out Book book);
    Book addBookInout(inout Book book);
}
```

对其中的AIDL的语法不熟悉的，可以再去温故一下：**[Android：学习AIDL，这一篇文章就够了(上)](http://blog.csdn.net/luoyanglizi/article/details/51980630)** ，我这里就不赘叙了。我主要讲一下实验的思路。可以看到，有定位tag的那三个方法它们的传参和返回值都是 Book 对象（Book是我自定义的一个实现了Parcelable的类，里面只有两个参数，**String name** 和 **int price**），并且它们的定位tag都不一样，接下来我会在客户端调用这三个方法，然后分别在客户端和服务端打印相关信息，从而验证我的猜想。下面贴上客户端和服务端的代码：

```
/**
 * 客户端的AIDLActivity.java
 * 由于测试机的无用debug信息太多，故log都是用的e
 *
 * Created by lypeer on 2016/7/17.
 */
public class AIDLActivity extends AppCompatActivity {

    
    private BookManager mBookManager = null;

    
    private boolean mBound = false;

    
    private List<Book> mBooks;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_aidl);
    }

    /**
     * 按钮的点击事件，点击之后调用服务端的addBookIn方法
     *
     * @param view
     */
    public void addBookIn(View view) {
        
        if (!mBound) {
            attemptToBindService();
            Toast.makeText(this, "当前与服务端处于未连接状态，正在尝试重连，请稍后再试", Toast.LENGTH_SHORT).show();
            return;
        }
        if (mBookManager == null) return;

        Book book = new Book();
        book.setName("APP研发录In");
        book.setPrice(30);
        try {
            
            Book returnBook = mBookManager.addBookIn(book);
            Log.e(getLocalClassName(), returnBook.toString());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    public void addBookOut(View view) {
        if (!mBound) {
            attemptToBindService();
            Toast.makeText(this, "当前与服务端处于未连接状态，正在尝试重连，请稍后再试", Toast.LENGTH_SHORT).show();
            return;
        }
        if (mBookManager == null) return;

        Book book = new Book();
        book.setName("APP研发录Out");
        book.setPrice(30);
        try {
            Book returnBook = mBookManager.addBookOut(book);
            Log.e(getLocalClassName(), returnBook.toString());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    public void addBookInout(View view) {
        if (!mBound) {
            attemptToBindService();
            Toast.makeText(this, "当前与服务端处于未连接状态，正在尝试重连，请稍后再试", Toast.LENGTH_SHORT).show();
            return;
        }
        if (mBookManager == null) return;

        Book book = new Book();
        book.setName("APP研发录Inout");
        book.setPrice(30);
        try {
            Book returnBook = mBookManager.addBookInout(book);
            Log.e(getLocalClassName(), returnBook.toString());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    /**
     * 尝试与服务端建立连接
     */
    private void attemptToBindService() {
        Intent intent = new Intent();
        intent.setAction("com.lypeer.aidl");
        intent.setPackage("com.lypeer.ipcserver");
        bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (!mBound) {
            attemptToBindService();
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (mBound) {
            unbindService(mServiceConnection);
            mBound = false;
        }
    }

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.e(getLocalClassName(), "service connected");
            mBookManager = BookManager.Stub.asInterface(service);
            mBound = true;

            if (mBookManager != null) {
                try {
                    mBooks = mBookManager.getBooks();
                    Log.e(getLocalClassName(), mBooks.toString());
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.e(getLocalClassName(), "service disconnected");
            mBound = false;
        }
    };
}
```

客户端这边的思路是很清晰的，不管怎么样，先连接上服务端再说，连接上之后再分别的调用AIDL中定义的三个方法，然后观察返回值的变化。接下来贴上服务端：

```
/**
 * 服务端的AIDLService.java
 *
 * Created by lypeer on 2016/7/17.
 */
public class AIDLService extends Service {

    public final String TAG = this.getClass().getSimpleName();

    
    private List<Book> mBooks = new ArrayList<>();

    
    private final BookManager.Stub mBookManager = new BookManager.Stub() {
        @Override
        public List<Book> getBooks() throws RemoteException {
            synchronized (this) {
                Log.e(TAG, "invoking getBooks() method , now the list is : " + mBooks.toString());
                if (mBooks != null) {
                    return mBooks;
                }
                return new ArrayList<>();
            }
        }

        @Override
        public Book addBookIn(Book book) throws RemoteException {
            synchronized (this) {
                if (mBooks == null) {
                    mBooks = new ArrayList<>();
                }
                if(book == null){
                    Log.e(TAG , "Book is null in In");
                    book = new Book();
                }
                
                book.setPrice(2333);
                if (!mBooks.contains(book)) {
                    mBooks.add(book);
                }
                
                Log.e(TAG, "invoking addBooks() method , now the list is : " + mBooks.toString());
                return book;
            }
        }

        @Override
        public Book addBookOut(Book book) throws RemoteException {
            synchronized (this) {
                if (mBooks == null) {
                    mBooks = new ArrayList<>();
                }
                if(book == null){
                    Log.e(TAG , "Book is null in Out");
                    book = new Book();
                }
                book.setPrice(2333);
                if (!mBooks.contains(book)) {
                    mBooks.add(book);
                }
                Log.e(TAG, "invoking addBooks() method , now the list is : " + mBooks.toString());
                return book;
            }
        }

        @Override
        public Book addBookInout(Book book) throws RemoteException {
            synchronized (this) {
                if (mBooks == null) {
                    mBooks = new ArrayList<>();
                }
                if(book == null){
                    Log.e(TAG , "Book is null in Inout");
                    book = new Book();
                }
                book.setPrice(2333);
                if (!mBooks.contains(book)) {
                    mBooks.add(book);
                }
                Log.e(TAG, "invoking addBooks() method , now the list is : " + mBooks.toString());
                return book;
            }
        }
    };

    @Override
    public void onCreate() {
        Book book = new Book();
        book.setName("Android开发艺术探索");
        book.setPrice(28);
        mBooks.add(book);
        super.onCreate();
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.e(getClass().getSimpleName(), String.format("on bind,intent = %s", intent.toString()));
        return mBookManager;
    }
}
```

同样脉络是很清楚的，首先接受客户端连接的请求，并把服务端处理好的BookManager.Stub的IBinder接口回传给客户端。在BookManager.Stub里面实现的方法里面，主要是接收客户端传过来的Book对象，并试图对其进行修改，然后把修改过的对象再传回去。

通过这样的实验，我们可以监测到客户端的数据流向服务端的情况，也可以监测到服务端的数据流向客户端的情况。根据这些数据，就可以验证上面我们的猜测的正确与否。根据我们的猜测，in ， out ， inout ，实际上是标志数据的流向的，那么这样的话用 in 或者 out 标志的数据应该只能单向传输，反向无效，而 inout 的数据则可以双向传输。结果数据是不是显示这样的特征的呢？

首先把这两个应用都装到手机上，然后都打开，并且客户端依次执行 *addBookIn()* ， *addBookOut()* ， *addBookInout()* 的点击事件，最后得到的两端的 log 信息分别是这样的：

```

1，on bind,intent = Intent { act=com.lypeer.aidl pkg=com.lypeer.ipcserver }
2，invoking getBooks() method , now the list is : [name : Android开发艺术探索 , price : 28]
3，on bind,intent = Intent { act=com.lypeer.aidl pkg=com.lypeer.ipcserver }
4，invoking getBooks() method , now the list is : [name : Android开发艺术探索 , price : 28]
5，invoking addBooks() method , now the list is : [name : Android开发艺术探索 , price : 28, name : APP研发录In , price : 2333]
6，invoking addBooks() method , now the list is : [name : Android开发艺术探索 , price : 28, name : APP研发录In , price : 2333, name : null , price : 2333]
7，invoking addBooks() method , now the list is : [name : Android开发艺术探索 , price : 28, name : APP研发录In , price : 2333, name : null , price : 2333, name : APP研发录Inout , price : 2333]
```

可以看到，服务端的 log 信息基本上是符合预期的。前四行是客户端在绑定服务端，list里面的那个元素是初始化进去的，可以不用管它。后三行分别体现了当客户端在调用 *addBookIn()* ， *addBookOut()* ， *addBookInout()* 方法的时候服务端接收到的数据：在 in 和 inout 作为定向 tag 的方法里，服务端能够正常的接收到客户端传过来的数据，但是在用 out 作为定向 tag 的方法里，服务端受到的是一个参数为空的 Book 对象！可是明明三个方法都是传的具有相同参数的 Book 对象过来！通过服务端的数据，结合之前的猜测，我们基本可以确定，之前的猜测是正确的，并且 in 作为定向 tag 表示数据只能由客户端流向服务端，out 反之，inout 则为数据可以双向流通。如果是这样的话，那么客户端的 log 信息我们也可以有一些猜测了：既然 in 表示数据只能由客户端流向服务端，那么客户端里它的返回值应当是一个参数为空的 Book 对象了；而 out 和 inout 作为定向 tag 的方法里它们的返回值则应当是与服务端里一样的。那么是不是这样的呢？看一下：

```

1，service connected
2，[name : Android开发艺术探索 , price : 28]
3，service connected
4，[name : Android开发艺术探索 , price : 28]
5，APP研发录In , price : 2333
6，name : null , price : 2333
7，name : APP研发录Inout , price : 2333
```

同样，前四行是连接的信息，后面三行则是调用*addBookIn()* ， *addBookOut()* ， *addBookInout()* 方法的时候服务端返回的数据。**结果这三行数据都和服务端的数据一模一样！**既然数据出现了问题，那么很显然的，前面的推测也肯定有问题。

### 3.3，数据流向！

我们再来捋一捋思路。首先，从服务端得到的数据来看，确实用 out 作为定向 tag 的方法他的数据是不能够从客户端流向服务端的——服务端收到的是一个空的对象！这说明定向 tag 与数据的流向有关系这个大的方向是没有问题的。那么问题出在哪里呢？**出在怎样看待数据流向这件事情上。**之前我把数据流向简单的看作了方法的参数输入和返回值输出，现在想来是有些问题的：

- 如果将数据从服务端流向客户端看成是方法将返回值传回客户端，那么为什么不将 out 设计成写在返回值前面呢？还要一个根本没有用的输入干嘛？设计这门语言的那些人那么腻害，没可能没想到这一点吧？
- AIDL里面的默认类型的定向 tag 默认且只能是 in ，难道它们只能作为参数输入，不能成为返回值？

所以，问题应该就处在把方法的返回值当作是数据从服务端流向客户端这件事上——虽然在某种意义上方法的返回值也可以说成是数据从服务端流向客户端，但是不是我们这里说的数据从服务端流向客户端——虽然有点绕，但是看到这里的读者应该是能明白我的意思的。那么到底应该如何理解数据从服务端流向客户端呢？现在方法的返回值已经被否决了，那么数据流回去的载体是什么呢——不管怎么样，数据总是要有一个载体的，不然我们怎么知道它已经回来了？**既然载体不是方法返回来的对象，那么必然是在调用方法之前就已经存在的对象。**虽然感觉很不可思议：我这个对象在客户端，而方法的实现是在服务端，那么它怎么能变动这个对象？但是既然是推导出来的结果，那么就做个试验看看不就清楚了。

要验证这个很简单，直接在客户端里面 log 输出服务端返回信息那里，把原本的输出 returnBook.toString() 改为 book.toString() 就可以了（ book 对象是方法的传参），具体的代码就不贴了，只有一点点变动。如果上面的猜测是正确的，那么输出的结果应当是 in 为定向 tag 的方法处 book 对象的参数不变，而 out ，inout 为定向 tag 的方法处 book 对象的参数与在服务端的参数一致。接下来再看下实际的 log 值：

```
1，service connected
2，[name : Android开发艺术探索 , price : 28]
3，name : APP研发录In , price : 30
4， name : null , price : 2333
5，name : APP研发录Inout , price : 2333
```

同样后三行表示了调用*addBookIn()* ， *addBookOut()* ， *addBookInout()* 方法之后 book 对象的参数。可以看到，输出的 log 信息终于和我前面预计的结果一致了！这说明前面的猜测是正确的！

### 3.4，得出结论

到这里基本上就可以下结论了：AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的参数为空的对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。

## 4，源码分析

上面我们通过猜测分析，设计实验等等手段得到了一个结论，那么接下来我们将进行源码分析，来看看在理论上能不能为我们的结论提供证明。首先我找到了as根据 **BookManager.aidl** 文件生成的 **BookManager.java** 文件，然后从中抽取了相关的代码片段：

```java
@Override
public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
    switch (code) {
        case INTERFACE_TRANSACTION: {
            reply.writeString(DESCRIPTOR);
            return true;
        }
        case TRANSACTION_getBooks: {
            data.enforceInterface(DESCRIPTOR);
            java.util.List<com.lypeer.ipcclient.Book> _result = this.getBooks();
            reply.writeNoException();
            reply.writeTypedList(_result);
            return true;
        }
        case TRANSACTION_addBookIn: {
            data.enforceInterface(DESCRIPTOR);
            
            com.lypeer.ipcclient.Book _arg0;
            
            if ((0 != data.readInt())) {
                _arg0 = com.lypeer.ipcclient.Book.CREATOR.createFromParcel(data);
            } else {
                _arg0 = null;
            }
            
            this.addBookIn(_arg0);
            
            reply.writeNoException();
            return true;
        }
        case TRANSACTION_addBookOut: {
            data.enforceInterface(DESCRIPTOR);
            com.lypeer.ipcclient.Book _arg0;
            
            
            _arg0 = new com.lypeer.ipcclient.Book();
            
            this.addBookOut(_arg0);
            reply.writeNoException();
            
            
            if ((_arg0 != null)) {
                reply.writeInt(1);
                _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
            } else {
                reply.writeInt(0);
            }
            return true;
        }
        case TRANSACTION_addBookInout: {
            data.enforceInterface(DESCRIPTOR);
            com.lypeer.ipcclient.Book _arg0;
            
            if ((0 != data.readInt())) {
                _arg0 = com.lypeer.ipcclient.Book.CREATOR.createFromParcel(data);
            } else {
                _arg0 = null;
            }
            this.addBookInout(_arg0);
            reply.writeNoException();
            if ((_arg0 != null)) {
                reply.writeInt(1);
                _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
            } else {
                reply.writeInt(0);
            }
            return true;
        }
    }
    return super.onTransact(code, data, reply, flags);
}

private static class Proxy implements com.lypeer.ipcclient.BookManager {
    private android.os.IBinder mRemote;

    Proxy(android.os.IBinder remote) {
        mRemote = remote;
    }

    @Override
    public android.os.IBinder asBinder() {
        return mRemote;
    }

    public java.lang.String getInterfaceDescriptor() {
        return DESCRIPTOR;
    }

    
    @Override
    public java.util.List<com.lypeer.ipcclient.Book> getBooks() throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        java.util.List<com.lypeer.ipcclient.Book> _result;
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            mRemote.transact(Stub.TRANSACTION_getBooks, _data, _reply, 0);
            _reply.readException();
            _result = _reply.createTypedArrayList(com.lypeer.ipcclient.Book.CREATOR);
        } finally {
            _reply.recycle();
            _data.recycle();
        }
        return _result;
    }

    
    @Override
    public void addBookIn(com.lypeer.ipcclient.Book book) throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            
            
            
            if ((book != null)) {
                _data.writeInt(1);
                book.writeToParcel(_data, 0);
            } else {
                _data.writeInt(0);
            }
            
            
            
            mRemote.transact(Stub.TRANSACTION_addBookIn, _data, _reply, 0);
            _reply.readException();
        } finally {
            _reply.recycle();
            _data.recycle();
        }
    }

    @Override
    public void addBookOut(com.lypeer.ipcclient.Book book) throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            
            mRemote.transact(Stub.TRANSACTION_addBookOut, _data, _reply, 0);
            _reply.readException();
            
            
            if ((0 != _reply.readInt())) {
                book.readFromParcel(_reply);
            }
        } finally {
            _reply.recycle();
            _data.recycle();
        }
    }

    @Override
    public void addBookInout(com.lypeer.ipcclient.Book book) throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            
            if ((book != null)) {
                _data.writeInt(1);
                book.writeToParcel(_data, 0);
            } else {
                _data.writeInt(0);
            }
            mRemote.transact(Stub.TRANSACTION_addBookInout, _data, _reply, 0);
            _reply.readException();
            if ((0 != _reply.readInt())) {
                book.readFromParcel(_reply);
            }
        } finally {
            _reply.recycle();
            _data.recycle();
        }
    }
}
```

在 AIDL 文件生成的 java 文件中，在进行远程调用的时候基本的调用顺序是先从 Proxy 类中调用相关方法，然后在这些方法中调用 transact() 方法，这个时候 Stub 中的 onTransact() 方法就会被调用，然后在这个方法里面再调用具体的业务逻辑的方法——当然，在这几个方法调用的过程中，总是会有一些关于数据的写入读出的操作，因为这些是跨线程操作，必须将数据序列化传输。读者看代码以及看注释的时候，最好跟着这条方法调用的线来，这样的话对于这整体数据的流向会清晰很多，也更加简明易读。

通过分析源码，我们可以很轻易的得出和之前分析的时候一样的结论，这样一来，基本上 AIDL 中定向 tag 是什么，in , out , inout 它们分别表示什么，有些什么区别这些问题，也就迎刃而解了。

首先还是对于复述一遍得出的结论：AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的参数为空的对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。（没错，这就是从上面复制粘粘的：））

最后，再说两个问题。

一个是可能有些读者不太明白，为什么不一上来就看源码？那样得出的结论必然是对的！确实如此。但是一方面，在通过探究得出一个结论之后去看源码，和直接去看源码，难度是不一样的——一开始就去看源码，未必看得懂，即算看得懂，未必能得出正确的结论——这话听起来似乎有些匪夷所思，但是那些经常看源码的同学应该是会有相同的体悟的。另一方面，源码，已经是成品了，我们去看它也许能够得出结论，但是很难得到那种作者在设计这个东西的时候的心路历程，那种在不同方案中取舍，最后选择了最优方案的心路历程——没有感受到这个，那么我觉得也许我们还需要在这个东西上面再多花些功夫来静下心的研究。

再就是这篇文章其实更多的想呈现的是那种对于技术的探究的态度。这个点只是一个很小的点，但是我找了很多的文章，很多的网站都没有找到一个很好的答案，这是为什么？是因为大家都没有去好好的静下心来研究它。花点时间，每个人都可以得出相同的答案，但是大多数人匆匆忙忙的看见了它，又匆匆忙忙的忽略了它——或者随意的翻查一下，似乎网上大家都说的挺有道理的，那么就这样了吧。这样是很没道理的。

文中相关代码可点击 **[传送门](http://download.csdn.net/detail/luoyanglizi/9582255)** 下载。

谢谢大家。