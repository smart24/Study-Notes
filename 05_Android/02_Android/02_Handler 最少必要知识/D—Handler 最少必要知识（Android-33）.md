**目录**
---

[一、 Handler 基本用法](#1)

[二、 Handler 工作原理](#2)

[三、 如何自己实现一个简单的 Handler-Looper 框架？](#3)

[四、 主线程的 Looper 为什么不会导致应用 ANR ？](#4)

[五、 同步屏障](#5)

[~~六、 Android 中为什么非 UI 线程不能更新 UI ？~~](#6)

[七、 参考链接](#7)


<h2 id="1">一、 Handler 基本用法</h2>

### 1. 在主线程中直接使用 Handler

#### 1.1 Handler post 系方法（7 个）

##### 1. API

```java

// 类名：android.os.Handler

Causes the Runnable r to be added to the message queue. The runnable will be run on the thread to which this handler is attached.
boolean post(@NonNull Runnable r)

boolean postAtFrontOfQueue(@NonNull Runnable r)

boolean postAtTime(@NonNull Runnable r, long uptimeMillis)

boolean postAtTime(@NonNull Runnable r, @Nullable Object token, long uptimeMillis)

boolean postDelayed(Runnable r, int what, long delayMillis)

boolean postDelayed(@NonNull Runnable r, long delayMillis)

boolean postDelayed(@NonNull Runnable r, @Nullable Object token, long delayMillis)
```


##### 2. 示例

1. 创建 Handler 类的实例；
2. 通过 Handler 类的实例调用 post(@NonNull Runnable r) 方法发送消息；

```java
public class HandlerPostActivity extends AppCompatActivity {
    private String TAG = "HANDLER";
    private ImageView mImageView;
    private Button mButton;
    private Handler mHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler_post);
        initView();
        initData();
    }

    private void initView() {
        mImageView = findViewById(R.id.image_view);
        mButton = findViewById(R.id.button);
        mButton.setOnClickListener((view) -> {
            // 2. 通过 Handler 类的实例调用 post(@NonNull Runnable r) 方法发送消息  ❷
            mHandler.post(() -> {
                // 3. 业务处理逻辑  ❸
                mImageView.setImageResource(R.drawable.animal_woodpecker);
            });
        });
    }

    private void initData() {
        // 1. 创建 Handler 类的实例  ❶
        mHandler = new Handler();
    }
}
```

上面的代码中，由于 mHandler 是跟 UI 线程关联的，所以，mHandler post 的 Runnable 会在 UI 线程执行。


#### 1.2 Handler send 系方法（7 个）

##### 1. API

```java

// 类名：android.os.Handler

Sends a Message containing only the what value.

boolean sendEmptyMessage(int what)

boolean sendEmptyMessageAtTime(int what, long uptimeMillis)

boolean sendEmptyMessageDelayed(int what, long delayMillis)

boolean sendMessage(@NonNull Message msg)

boolean sendMessageAtFrontOfQueue(@NonNull Message msg)

boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis)

boolean sendMessageDelayed(@NonNull Message msg, long delayMillis)
```


##### 2. 示例

1. 创建 Handler 匿名类的实例，并重写 Handler 类的 handleMessage(@NonNull Message msg) 方法；
2. 通过 Handler 匿名类的实例调用 sendMessage(@NonNull Message msg) 方法发送消息；

```java
public class HandlerPostActivity extends AppCompatActivity {
    private String TAG = "HANDLER";
    private ImageView mImageView;
    private Button mButton;
    // 1. 创建 Handler 匿名类的实例，并重写 Handler 类的 handleMessage(@NonNull Message msg) 方法  ❶
    private Handler mHandler = new Handler(Looper.myLooper()) {
        @Override
        public void handleMessage(@NonNull Message msg) {
            super.handleMessage(msg);
            // 3. 业务处理逻辑  ❸
            mImageView.setImageResource(R.drawable.animal_woodpecker);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler_post);
        initView();
    }

    private void initView() {
        mImageView = findViewById(R.id.image_view);
        mButton = findViewById(R.id.button);
        mButton.setOnClickListener((view) -> {
            // 2. 通过 Handler 匿名类的实例调用 sendMessage(@NonNull Message msg) 方法发送消息  ❷
            Message msg = Message.obtain();
            msg.what = 0x0001;
            mHandler.sendMessage(msg);
        });
    }
}
```

从上面的代码可以看出：由于 mHandler 是跟 UI 线程关联的，所以，Handler 匿名类的 handleMessage() 方法会在 UI 线程执行。因为 mHandler sendMessage() 方法发送的 Message 最终会传递到 Handler 类的 handleMessage() 方法中，所以，mHandler sendMessage() 方法发送的 Message 最终会在 UI 线程中被处理。


### 2. 如何创建一个与（非 UI ）线程相关的 Handler？

#### 2.1 步骤

1. 创建 Thread 类；
2. 在 Thread 的 run() 方法中，调用 Looper.prepare()；
3. 在 Thread 的 run() 方法中，Looper.prepare() 的下面创建 Handler 匿名类的实例，并重写 Handler 类的 handleMessage() 方法；
4. 在 Thread 的 run() 方法中，创建 Handler 匿名类实例的下面调用 Looper.loop();
5. 调用 Thread 类的 start() 方法启动线程；
6. 用第 3 步创建的 Handler 匿名类的实例 post/send 消息；


#### 2.2 示例

```java
public class LooperThreadActivity extends AppCompatActivity implements View.OnClickListener {

    private String TAG = "HANDLER";
    private ImageView mImageView;
    private Button mButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_looper_thread);
        initView();
    }

    private void initView() {
        mImageView = findViewById(R.id.image_view);
        mButton = findViewById(R.id.button);
        mButton.setOnClickListener(this);
    }


    @Override
    public void onClick(View view) {
        int id = view.getId();
        switch (id) {
            case R.id.button:
                createLooperThread();
                break;
        }
    }

    private void createLooperThread() {
        // 5. 调用 Thread 类的 start() 方法启动线程  ❺
        LooperThread looperThread = new LooperThread();
        looperThread.start();
        while (looperThread.mHandler == null) {
            SystemClock.sleep(1);
        }
        
        // 6. 用第 3 步创建的 Handler 匿名类的实例 post/send 消息  ❻
        Message message = new Message();
        message.what = 1024;
        looperThread.mHandler.sendMessage(message);
    }

    // 1. 创建 Thread 类  ❶
    class LooperThread extends Thread {
        public Handler mHandler;

        @Override
        public void run() {
            // 2. 在 Thread 的 run() 方法中，调用 Looper.prepare()  ❷
            Looper.prepare();

            // 3. 在 Thread 的 run() 方法中，Looper.prepare() 的下面创建 Handler 匿名类的实例  ❸
            mHandler = new Handler(Looper.myLooper()) {
                @Override
                public void handleMessage(Message msg) {
                    // 7. 业务处理逻辑  ❼
                    Log.e(TAG, "handleMessage: " + msg.what);
                }
            };
            
            // 4. 在 Thread 的 run() 方法中，创建 Handler 匿名类实例的下面调用 Looper.loop()  ❹
            Looper.loop();
        }
    }
}
```


<h2 id="2">二、 Handler 工作原理</h2>

### 1. Handler 工作原理

创建一个与（非 UI ）线程相关的 Handler 的关键步骤如下：

1. 创建 Thread 类的子类，重写 run() 方法；
2. 在 Thread 类的子类的 run() 方法中，调用 Looper.prepare()；
3. 在 Thread 类的子类的 run() 方法中，Looper.prepare() 的下面创建 Handler 匿名类的实例，并重写 Handler 类的 handleMessage() 方法；
4. 在 Thread 类的子类的 run() 方法中，创建 Handler 匿名类实例的下面调用 Looper.loop();
5. 调用 Thread 类的 start() 方法启动线程；
6. 用第 3 步创建的 Handler 匿名类的实例 post/send 消息；

接下来，我们就按这个步骤解析 Handler 工作原理，看看每一步都干了什么，最终我们要搞明白以下问题：

- Message 是怎么被添加到 MessageQueue 的？
    - Message 是怎么发出去的？
    - Message 是怎么被接收的？
    - Message 是怎么被处理的？
    - Message 是怎么被回收的？
- Handler 能切线程，它是怎么做到的？
    - Handler 发送的 Message 为什么能在其他线程（非发送 Message 的线程）执行？
    - 线程切换是什么时候发生的？
    - 怎么发生的？


#### 1.1 创建 Thread 类的子类，重写 run() 方法

既然是创建一个与（非 UI ）线程相关的 Handler，所以，第一步当然是创建一个自己的线程。


#### 1.2 在 Thread 类的子类的 run() 方法中，调用 Looper.prepare()

其实，Looper 类的注释里面已经把要做的事说清楚了：

> Class used to run a message loop for a thread.  Threads by default do not have a message loop associated with them; to create one, call {@link #prepare} in the thread that is to run the loop, and then {@link #loop} to have it process messages until the loop is stopped.
> 
> Most interaction with a message loop is through the {@link Handler} class.
>
> This is a typical example of the implementation of a Looper thread, using the separation of {@link #prepare} and {@link #loop} to create an initial Handler to communicate with the Looper.
> 
> class LooperThread extends Thread {
> &ensp;&ensp;&ensp;&ensp;public Handler mHandler;
> 
> &ensp;&ensp;&ensp;&ensp;public void run() {
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;Looper.prepare();
> 
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;mHandler = new Handler(Looper.myLooper()) {
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;mHandler = new Handler(Looper.myLooper()) {
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;// process incoming messages here
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;};
> 
> &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;Looper.loop();
> &ensp;&ensp;&ensp;&ensp;}
> }

通过 Looper 的类注释，我们知道了：默认情况下，是没有 Looper 与线程关联的，所以，为了给线程创建一个与之关联的 Looper，我们需要在线程的 run() 方法里干三件事：

1. 首先，执行 Looper.prepare()；
2. 然后，创建 Handler 匿名类；
3. 最后，执行 Looper.loop()；

接下来，我们看看 Looper.prepare() 方法内部都干了什么事。

```java

// 类名：android.os.Looper

public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

在 prepare(boolean quitAllowed) 方法里有一个 sThreadLocal，sThreadLocal 是什么，它是什么时候被创建的？

```java

// 类名：android.os.Looper

public final class Looper {
    
    ...
    
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    
    ...
    
}
```

sThreadLocal 是 Looper 类的一个静态成员变量，所以，Looper 类首次加载时，sThreadLocal 就会被创建。另外，因为 sThreadLocal 是 Looper 类的一个静态成员变量，所以，所有 Looper 的对象都共用这一个 sThreadLocal。

接下来，我们看看 sThreadLocal.get() 方法内部都干了什么事。

```java

// 类名：java.lang.ThreadLocal

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

ThreadLocal 的 get() 方法执行完之后，如果执行会它的 get() 方法的线程没有 ThreadLocalMap，则会创建一个 ThreadLocalMap，并把创建的 ThreadLocalMap 的对象赋值给执行 ThreadLocal 的 get() 方法的线程的成员变量 threadLocals，与此同时，会把 ThreadLocal initialValue() 方法创建的初始值存到刚创建的 ThreadLocalMap 对象中，并以当前 ThreadLocal 作为 key。

另外，由于 sThreadLocal 是 Looper 类的一个静态成员变量，所以，当在不同的线程中执行 Looper.prepare() 方法之后，构建过程会为每一个执行过 Looper.prepare() 方法的线程创建一个 ThreadLocalMap，同时会以 Looper 中的 sThreadLocal 作为 key 在 ThreadLocalMap 存储初始值。也就是说，每一个执行过 Looper.prepare() 方法的线程的 ThreadLocalMap 中的 key 其实都是一样的。

就像下面这个样子：

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5pGRvnAECSEcxKFBwVv.rDBc0hfv7MNWdzGyI0.5pawkpqydGZB3bIg40tK4jAwRzFQtFbriWRpgLIlxqjDpu6c!/b&bo=gAc4BAAAAAADB5k!&rf=viewer_4" width=720 height=427 />

由于目前没有 Looper 与当前线程关联，所以，Looper prepare() 方法中 sThreadLocal.get() 方法执行完之后，会返回 null。因此，接下来会执行 sThreadLocal.set() 方法，并把新创建的 Looper 对象作为参数传入。

接下来，我们看看 sThreadLocal.set() 方法内部都干了什么事。

```java

// 类名：android.os.Looper

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}

// 类名：android.os.MessageQueue

MessageQueue(boolean quitAllowed) {
    mQuitAllowed = quitAllowed;
    mPtr = nativeInit();
}

// 类名：java.lang.ThreadLocal

public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

在 Looper 构造方法内部会调用 MessageQueue 的构造方法，并把创建的 MessageQueue 对象赋值给 Looper 的成员变量 mQueue，于此同时，会把当前线程赋值给 Looper 的成员变量 mThread。

在 ThreadLocal 的 set() 方法内部，会把传入的 Looper 对象以当前 ThreadLocal 为 key 存到当前线程的 ThreadLocalMap 中。

也就是说，到目前为止，当前线程的 ThreadLocalMap 中只存了一个 Looper 对象，并且与这个 Looper 对象对应的 key 是当前 ThreadLocal 对象，也就是 Looper 中的静态成员变量 sThreadLocal。


#### 1.3 在 Thread 类的子类的 run() 方法中，Looper.prepare() 的下面创建 Handler 匿名类的实例，并重写 Handler 类的 handleMessage() 方法

老样子，先通过 Handler 的类注释看看它的主要功能：

> A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue. Each Handler instance is associated with a single thread and that thread's message queue. When you create a new Handler it is bound to a Looper. It will deliver messages and runnables to that Looper's message queue and execute them on that Looper's thread.
>
> There are two main uses for a Handler: (1) to schedule messages and runnables to be executed at some point in the future; and (2) to enqueue an action to be performed on a different thread than your own.
>
> Scheduling messages is accomplished with the post, postAtTime(Runnable, long), postDelayed, sendEmptyMessage, sendMessage, sendMessageAtTime, and sendMessageDelayed methods. The post versions allow you to enqueue Runnable objects to be called by the message queue when they are received; the sendMessage versions allow you to enqueue a Message object containing a bundle of data that will be processed by the Handler's handleMessage method (requiring that you implement a subclass of Handler).
>
> When posting or sending to a Handler, you can either allow the item to be processed as soon as the message queue is ready to do so, or specify a delay before it gets processed or absolute time for it to be processed. The latter two allow you to implement timeouts, ticks, and other timing-based behavior.
>
> When a process is created for your application, its main thread is dedicated to running a message queue that takes care of managing the top-level application objects (activities, broadcast receivers, etc) and any windows they create. You can create your own threads, and communicate back with the main application thread through a Handler. This is done by calling the same post or sendMessage methods as before, but from your new thread. The given Runnable or Message will then be scheduled in the Handler's message queue and processed when appropriate.

通过 Handler 的类注释，我们知道了：当创建一个 Handler 对象时，它会绑定到 一个 Looper，并且，它将传递 Message 和 Runnable 到它绑定的 Looper 的 MessageQueue 中，并在它绑定的 Looper 所在的线程执行执行 Message 和 Runnable。

另外，我们还知道了 Handler 的主要功能：

1. 延迟 post/send 消息；
2. 切换线程（线程间通信）；

接下来，我们看看 Handler 的构造方法内部都干了什么事，它又是如何绑定到 Looper 的？

Handler 的构造方法有 7 个，分别是：

```java

// 类名：android.os.Handler

@Deprecated
public Handler() {  ❶
    this(null, false);
}

@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
public Handler(boolean async) {  ❷
    this(null, async);
}

@Deprecated
public Handler(@Nullable Callback callback) {  ❸
    this(callback, false);
}

hide
public Handler(@Nullable Callback callback, boolean async) {  ❹
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

public Handler(@NonNull Looper looper) {  ❺
    this(looper, null, false);
}

public Handler(@NonNull Looper looper, @Nullable Callback callback) {  ❻
    this(looper, callback, false);
}

@UnsupportedAppUsage
public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {  ❼
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

不难看出，不传 Looper 的构造方法最终都会调用 Handler(Callback callback, boolean async) ，传 Looper 的构造方法最终都会调用 Handler(Looper looper, Callback callback, boolean async)。

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5ubCTE1OomRtulVFWozFrpy8R7oVMNUF6BwY00b9A9JlHXeNf*RAotI58pP*CQrbEpcTwvDW0WmLrctsq571uPs!/b&bo=gAc4BAAAAAADB5k!&rf=viewer_4" width=720 height=427 />

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5ubCTE1OomRtulVFWozFrpzeBfVXaJhYJ0Zg4UuTYlPZDjho.vyuLgvHSk2QPuEkjqJFG3z0mNlGCHC.2bgr*y8!/b&bo=gAc4BAAAAAADJ7k!&rf=viewer_4" width=720 height=427 />

Handler(Callback callback, boolean async) 和 Handler(Looper looper, Callback callback, boolean async) 内部的处理逻辑，基本一样，唯一的不同就是： Handler(Callback callback, boolean async) 构造方法中的 Looper 对象是通过调用 Looper.myLooper() 获取的，而 Handler(Looper looper, Callback callback, boolean async) 构造方法中的 Looper 对象是外部传入的。

接下来，我们看看 Looper.myLooper() 方法内部都干了什么事，它又是如何获取到 Looper 对象的？

```java

// 类名：android.os.Looper

public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

// 类名：java.lang.ThreadLocal

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

因为之前在调用 Looper.prepare() 方法的时候，已经把 Looper 对象存到当前线程的 ThreadLocalMap 中，所以，此处可以获取到 Looper.prepare() 方法调用时创建的 Looper 对象。

简单总结一下：在 Handler 的构造方法里，首先，会把通过调用 Looper.myLooper() 获取到的 Looper 对象或 Handler 构造方法传入的 Looper 对象赋值给 Handler 的成员变量 mLooper；其次，会把 Looper 的成员变量 mQueue 赋值给 Handler 的成员变量 mQueue；然后，把传入的 callback 赋值给 Handler 的成员变量 mCallback；最后，把传入的 async 赋值给 Handler 的成员变量 mAsynchronous。

代码也看完了，问题来了，Handler 到底是如何绑定到 Looper 的？不过是把通过调用 Looper.myLooper() 获取到的 Looper 对象或 Handler 构造方法传入的 Looper 对象赋值给 Handler 的成员变量 mLooper，仅此而已。


#### 1.4 在 Thread 类的子类的 run() 方法中，创建 Handler 匿名类实例的下面调用 Looper.loop()

接下来，我们看看 Looper.loop() 方法内部都干了什么事，它是如何让消息队列跑起来的？

```java

// 类名：android.os.Looper

/**
 * Run the message queue in this thread. Be sure to call
 * {@link #quit()} to end the loop.
 */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    if (me.mInLoop) {
        Slog.w(TAG, "Loop again would have the queued messages be executed"
                + " before this one completed.");
    }

    me.mInLoop = true;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    // Allow overriding a threshold with a system prop. e.g.
    // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
    final int thresholdOverride =
            SystemProperties.getInt("log.looper."
                    + Process.myUid() + "."
                    + Thread.currentThread().getName()
                    + ".slow", 0);

    me.mSlowDeliveryDetected = false;

    // ❶ 执行死循环
    for (;;) {  
        if (!loopOnce(me, ident, thresholdOverride)) {
            return;
        }
    }
}

/**
 * Poll and deliver single message, return true if the outer loop should continue.
 */
private static boolean loopOnce(final Looper me,
        final long ident, final int thresholdOverride) {
    // ❷ 从 MessageQueue 取消息，没消息时可能会阻塞
    Message msg = me.mQueue.next(); // might block
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return false;
    }

    // This must be in a local variable, in case a UI event sets the logger
    final Printer logging = me.mLogging;
    if (logging != null) {
        logging.println(">>>>> Dispatching to " + msg.target + " "
                + msg.callback + ": " + msg.what);
    }
    // Make sure the observer won't change while processing a transaction.
    final Observer observer = sObserver;

    final long traceTag = me.mTraceTag;
    long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
    long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
    if (thresholdOverride > 0) {
        slowDispatchThresholdMs = thresholdOverride;
        slowDeliveryThresholdMs = thresholdOverride;
    }
    final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
    final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

    final boolean needStartTime = logSlowDelivery || logSlowDispatch;
    final boolean needEndTime = logSlowDispatch;

    if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
        Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
    }

    final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
    final long dispatchEnd;
    Object token = null;
    if (observer != null) {
        token = observer.messageDispatchStarting();
    }
    long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
    try {
        // ❹ 取到 Message，调用 Message 的 target 成员变量的 dispatchMessage() 方法，完成消息的分发
        //
        // Message 的 target 成员变量是发送此 Message 的 Handler 对象
        //
        // 参考【2-1.6 用第 3 步创建的 Handler 匿名类的实例 post/send 消息】
        msg.target.dispatchMessage(msg);
        if (observer != null) {
            observer.messageDispatched(token, msg);
        }
        dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
    } catch (Exception exception) {
        if (observer != null) {
            observer.dispatchingThrewException(token, msg, exception);
        }
        throw exception;
    } finally {
        ThreadLocalWorkSource.restore(origWorkSource);
        if (traceTag != 0) {
            Trace.traceEnd(traceTag);
        }
    }
    if (logSlowDelivery) {
        if (me.mSlowDeliveryDetected) {
            if ((dispatchStart - msg.when) <= 10) {
                Slog.w(TAG, "Drained");
                me.mSlowDeliveryDetected = false;
            }
        } else {
            if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                    msg)) {
                // Once we write a slow delivery log, suppress until the queue drains.
                me.mSlowDeliveryDetected = true;
            }
        }
    }
    if (logSlowDispatch) {
        showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
    }

    if (logging != null) {
        logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
    }

    // Make sure that during the course of dispatching the
    // identity of the thread wasn't corrupted.
    final long newIdent = Binder.clearCallingIdentity();
    if (ident != newIdent) {
        Log.wtf(TAG, "Thread identity changed from 0x"
                + Long.toHexString(ident) + " to 0x"
                + Long.toHexString(newIdent) + " while dispatching to "
                + msg.target.getClass().getName() + " "
                + msg.callback + " what=" + msg.what);
    }

    // ❺ 回收用完的 Message
    msg.recycleUnchecked();

    return true;
}

// 类名：android.os.MessageQueue

Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        // ❸ 陷入 Native 层，进入休眠状态，让出 CPU
        nativePollOnce(ptr, nextPollTimeoutMillis);

        ...
        
    }
}

private native void nativePollOnce(long ptr, int timeoutMillis);
```

在 Looper 的 loop() 方法中，一共干了 5 件事，已在上面的代码中标记出来，分别是：

1. 执行死循环；
2. 通过调用 MessageQueue 的 next() 方法获取 Message，此方法在没有消息时会阻塞；
3. 在 MessageQueue 的 next() 方法内部，会调用 nativePollOnce() 方法进入休眠状态；
4. 取到 Message 之后，会调用发送此 Message 的 Handler 的 dispatchMessage() 方法，完成消息的分发；
5. 回收用完的 Message；

上面有一个地方没有说，那就是 MessageQueue 调用完 nativePollOnce() 方法进入休眠状态之后，什么时候被唤醒。想想也知道，当然是有消息的时候被唤醒啦。

~~接下来，我们看看 Handler 匿名类的实例 post/send 消息之后，MessageQueue 的 nativePollOnce() 方法是如何被唤醒的。~~

不急，都会说到的。别忘了创建的线程还没启动呢，Looper 的 loop 循环还没有转起来呢。还没学会走，你就要跑吗？一步一步来，不能急功近利，不能胡子眉毛一把抓，不能乱了自己的节奏。


#### 1.5 调用 Thread 类的 start() 方法启动线程

这一步的主要目的是：执行 Thread 的 run() 方法，让上面的整个流程跑起来。

毕竟 Looper 的 loop() 方法，只有 Thread 的 run() 方法执行了之后，才会跑起来。


#### 1.6 用第 3 步创建的 Handler 匿名类的实例 post/send 消息

接下来，我们看看 Handler 匿名类的实例 post/send 的消息是如何被发送其他线程中执行的？

Handler post 系方法：

```java

// 类名：android.os.Handler

public final boolean post(@NonNull Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

public final boolean postAtFrontOfQueue(@NonNull Runnable r) {
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}

public final boolean postAtTime(@NonNull Runnable r, long uptimeMillis) {
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}

public final boolean postAtTime(@NonNull Runnable r, @Nullable Object token, long uptimeMillis) {
    return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
}

public final boolean postDelayed(Runnable r, int what, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r).setWhat(what), delayMillis);
}

public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}

public final boolean postDelayed(@NonNull Runnable r, @Nullable Object token, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r, token), delayMillis);
}
```

仔细观察发现，Handler 的 post 系方法最终都会调用 Handler send 系方法，同时，会通过调用 getPostMessage(Runnable r) 或 getPostMessage(Runnable r, Object token) 方法把 Runnable 包装成 Message：

```java

// 类名：android.os.Handler

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

private static Message getPostMessage(Runnable r, Object token) {
    Message m = Message.obtain();
    m.obj = token;
    m.callback = r;
    return m;
}
```

接下来，我们看看 Handler send 系方法内部都干了什么事。

```java

// 类名：android.os.Handler

public final boolean sendEmptyMessage(int what) {
    return sendEmptyMessageDelayed(what, 0);
}

public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageAtTime(msg, uptimeMillis);
}

public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}

public final boolean sendMessage(@NonNull Message msg) {
    return sendMessageDelayed(msg, 0);
}

public final boolean sendMessageAtFrontOfQueue(@NonNull Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}

public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

Handler 的 send 系方法有点多，看着有点乱，把 Handler 的 send 系方法稍微整理一下：

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5ubCTE1OomRtulVFWozFrpy84*vTXrzIHlznJEof4kKUwQsAiQsIc9lajggYW0pMv*0F9Iq.XImY9Q*ympGbMXY!/b&bo=gAc4BAAAAAADJ7k!&rf=viewer_4" width=720 height=427 />

这样看起来是不是清晰多了？在上面分析 Handler 的 post 系方法时曾说过「Handler 的 post 系方法最终都会调用 Handler send 系方法」，所以，上面的图加上 Handler 的 post 系方法之后，就变成了下面这个样子：

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5nvx5sVJdrRfinJ24*cSIOhcwjKKURPTLwwPJnf1suvH.0YFMEK0hJ2PWanr*DKS3WJXQ7hgRWVC9o*GNCOD2fs!/b&bo=gAc4BAAAAAADB5k!&rf=viewer_4" width=720 height=427 />

通过上面的图可知：Handler post/send 系方法最终都会调用 Handler 的 enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) 方法。

接下来，我们看看 Handler 的 enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) 方法内部都干了什么事。

```java

// 类名：android.os.Handler

private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg, long uptimeMillis) {
    // 把当前 Handler 对象赋值给 Message 的 target 成员变量
    //
    // 后续在 Looper 的 loopOnce() 方法中，正是通过调用 Message 的 target 成员变量的 dispatchMessage(msg) 方法完成了消息分发
    //
    // 参考【2-1.4 在 Thread 类的子类的 run() 方法中，创建 Handler 匿名类实例的下面调用 Looper.loop()】
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}

// 类名：android.os.MessageQueue

boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }

    synchronized (this) {
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        // 1. 把传入的 Message 插入 mMessages 链表中  ❶
        if (p == null || when == 0 || when < p.when) {
            // 1.1 如果 mMessages 链表中没有 Message 或传入 Message 的 when 为 0 或传入 Message 的 when 比 mMessages 链表中的第一个元素的
            // when 还小，则把传入的 Message 插入到 mMessages 链表的头部，并唤醒阻塞的 nativePollOnce(long ptr, int timeoutMillis) 方法
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            // 1.2 如果 mMessages 链表不为 null 且传入 Message 的 when 比 mMessages 链表中的第一个元素的 when 还大，则按时间顺序把传入的 
            // Message 插入到 mMessages 链表的中间，通常我们不需要唤醒事件队列，除非 mMessages 链表的头部存在一个屏障（barrier），并且传入
            // 的 Message 是 mMessages 链表中最早的异步消息
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            // 2. 唤醒 MessageQuque 中阻塞的 nativePollOnce(long ptr, int timeoutMillis) 方法，进而让 MessageQuque 的  ❷
            // next() 方法继续往下执行
            nativeWake(mPtr);
        }
    }
    return true;
}

Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        
        // 3. 没有消息时，会陷入 Native 层，进入休眠状态，让出 CPU  ❸
        //
        // MessageQueue 的 nativeWake(long ptr) 方法会唤醒此方法（nativePollOnce(long ptr, int timeoutMillis)），进而
        // 让 next() 方法继续往下执行
        //
        // nextPollTimeoutMillis = -1，表示无消息，会进入休眠状态；
        // nextPollTimeoutMillis 正数，表示下一个消息到来前还需要等待的时长，时长走完，此方法返回，继续走下面的
        // 「寻找合适的 Message」的处理逻辑；
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            // 4. 因为通过我们创建的 Handler 匿名类对象发送的 Message 都是有 target 的，所以，我们创建的 Handler 匿名类对象  ❹
            // 发送的 Message 不会进这个 if 判断
            //
            // 顺便说一句，这块的处理逻辑是和同步屏障相关的
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                // 5. 如果 mMessages 链表的头元素的 when 比 now 大，则继续等；否则，返回 mMessages 链表的头元素  ❺
                if (now < msg.when) {
                    // 5.1 如果 mMessages 链表的头元素的 when 比 now 大，则继续
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 5.2 返回 mMessages 链表的头元素
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }

            // If first time idle, then get the number of idlers to run.
            // Idle handles only run if the queue is empty or if the first message
            // in the queue (possibly a barrier) is due to be handled in the future.
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                mBlocked = true;
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // Run the idle handlers.
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler

            boolean keep = false;
            try {
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}

// 类名：android.os.Looper

private static boolean loopOnce(final Looper me,final long ident, final int thresholdOverride) {
    // 6. 之前因为 MessageQueue next() -> nativePollOnce() 方法而阻塞，现在因为 MessageQueue next() 取到了 Message，  ❻
    // 所以，此处可以继续往下走了
    Message msg = me.mQueue.next(); // might block
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return false;
    }

    ...
    
    long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
    try {
        // 7. 取到 Message，调用 Message 的 target 成员变量的 dispatchMessage() 方法，完成消息的分发  ❼
        msg.target.dispatchMessage(msg);
        if (observer != null) {
            observer.messageDispatched(token, msg);
        }
        dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
    } catch (Exception exception) {
        if (observer != null) {
            observer.dispatchingThrewException(token, msg, exception);
        }
        throw exception;
    } finally {
        ThreadLocalWorkSource.restore(origWorkSource);
        if (traceTag != 0) {
            Trace.traceEnd(traceTag);
        }
    }
    
    ...

    // 8. 回收用完的 Message  ❽
    msg.recycleUnchecked();

    return true;
}
```

通过上面的代码可以发现：在 Handler 的 enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) 方法内部，首先，会把自己赋值给待发送 Message 的 target 成员变量，然后，调用 MessageQueue 的 enqueueMessage(Message msg, long when) 方法。

在 MessageQueue 的 enqueueMessage(Message msg, long when) 方法内部，首先，会把传入的 Message 插入到 MessageQueue 内部的 mMessages 链表的合适的位置，然后，唤醒 MessageQuque 中阻塞的 nativePollOnce(long ptr, int timeoutMillis) 方法。

这样处理之后，MessageQuque 中阻塞的 nativePollOnce(long ptr, int timeoutMillis) 方法就不再阻塞了，进而 MessageQuque 的 next() 方法就能继续往下走了。

在 MessageQuque 的 next() 方法中，会根据传入的 Message 的情况决定是继续往下等，还是直接返回合适的 Message。如果是继续往下等，那啥事也没有，等就完了，等时间到了，再返回合适的 Message；如果是直接返回合适的 Message，那 Looper 的 loopOnce() 方法也能继续往下走了，然后 MessageQuque 的 next() 方法返回的 Message 的成员变量 target 的 dispatchMessage() 方法就会被调用，并且，还会在 dispatchMessage() 方法中传入 MessageQuque 的 next() 方法返回的 Message。

在 Looper 的 loopOnce() 方法内部，接下来会调用 Message 的 recycleUnchecked() 方法将该 Message 回收。

至此，Message 的整个流程（发出 -> 接收 -> 处理 -> 回收）就走完了。

简单画个图总结一下：

<img src="http://m.qpic.cn/psc?/V10aaGcc22Xzag/ruAMsa53pVQWN7FLK88i5uO.FDcJALGjUfG9z2WYgj5YWcIxMRTBdmmZ5WaGJrWnO*82Fz*z4lwrePosnQwxC*bOpnQdK5nmFraMyEF7NQU!/b&bo=gAc4BAAAAAADB5k!&rf=viewer_4" width=720 height=427 />


### 2. 为什么 UI 线程没有调用 Looper.prepare() 和 Looper.loop() 却能直接使用 Handler？

UI 线程和非 UI 线程使用 Handler 方式是一样的，也需要调用 Looper.prepare() 和 Looper.loop()。

```java

类名：android.app.ActivityThread

public final class ActivityThread extends ClientTransactionHandler implements ActivityThreadInternal {

    ...
    
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // Install selective syscall interception
        AndroidOs.install();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        // Call per-process mainline module initialization.
        initializeMainlineModules();

        Process.setArgV0("<pre-initialized>");

        // 1. 创建 UI 线程的 Looper  ❶
        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        // 2. 执行 Looper.loop()  ❷
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
    
    ...
    
}

// 类名：android.os.Looper

public final class Looper {

    ...

    public static void prepareMainLooper() {
        // 3. 创建 Looper  ❸
        // 参考【2-1.2 在 Thread 类的子类的 run() 方法中，调用 Looper.prepare()】
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
    
    ...
    
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
    ...
    
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
    
    ...
    
}
```

ActivityThread 的 main() 方法是一个 Android 应用程序的入口。在 ActivityThread 的 main() 方法中，会创建一个新的应用进程，并初始化 Android 运行时环境。也就是说，ActivityThread 的 main() 方法所在的线程就是主线程，也就是 UI 线程。

在 ActivityThread 的 main() 方法中，我们看到了熟悉的 Looper.prepareMainLooper() 和 Looper.loop()，正是因为有了这两个方法的调用，所以，我们才可以在 Activity 里面直接使用 Handler。也就是说，在创建应用进程的时候，就给 UI 线程绑定 Looper 了。


<h2 id="3">三、 如何自己实现一个简单的 Handler-Looper 框架？</h2>

通过上面的分析可以知道 Handler-Looper 框架核心共有 4 个，分别是：

1. Handler；
2. Looper；
3. Message；
4. MessageQueue；

每个类的功能如下：

| 类名 | 功能 |
| --- | --- |
| Handler | 发消息；处理消息 |
| SimpleLooper | 关联线程；执行死循环，不停地取消息 |
| Message | 创建消息；回收消息 |
| MessageQueue | 存消息，给消息排序；取消息 |

功能已经拆分完了，其实实现更简单：

```java

类名：SimpleHandler

public class SimpleHandler {
    final SimpleLooper mLooper;
    final SimpleMessageQueue mQueue;

    public SimpleHandler() {
        mLooper = SimpleLooper.myLooper();
        mQueue = mLooper.mQueue;
    }

    public SimpleHandler(SimpleLooper looper) {
        mLooper = looper;
        mQueue = looper.mQueue;
    }

    // 发消息
    public final void sendMessage(SimpleMessage msg) {
        System.out.println("SimpleHandler sendMessage()         CURRENT THREAD: " + Thread.currentThread());
        SimpleMessageQueue queue = mQueue;
        enqueueMessage(queue, msg);
    }

    private void enqueueMessage(SimpleMessageQueue queue, SimpleMessage msg) {
        System.out.println("SimpleHandler enqueueMessage()      CURRENT THREAD: " + Thread.currentThread());
        msg.target = this;
        queue.enqueueMessage(msg);
    }

    // 处理消息
    public void dispatchMessage(SimpleMessage msg) {
        System.out.println("SimpleHandler dispatchMessage()     CURRENT THREAD: " + Thread.currentThread());
        handleMessage(msg);
    }

    public void handleMessage(SimpleMessage msg) {
    }
}

类名：SimpleLooper

public class SimpleLooper {
    static final ThreadLocal<SimpleLooper> sThreadLocal = new ThreadLocal<>();
    final SimpleMessageQueue mQueue;
    final Thread mThread;

    private SimpleLooper() {
        mQueue = new SimpleMessageQueue();
        mThread = Thread.currentThread();
    }

    // 关联线程
    public static void prepare() {
        System.out.println("SimpleLooper prepare()              CURRENT THREAD: " + Thread.currentThread());
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new SimpleLooper());
    }

    public static SimpleLooper myLooper() {
        return sThreadLocal.get();
    }

    // 执行死循环，不停地取消息
    public static void loop() {
        for (;;) {
            System.out.println("SimpleLooper loop() enter           CURRENT THREAD: " + Thread.currentThread());
            SimpleMessage msg = myLooper().mQueue.next();
            msg.target.dispatchMessage(msg);
            msg.recycle();
            System.out.println("SimpleLooper loop() exit            CURRENT THREAD: " + Thread.currentThread());
        }
    }
}

类名：SimpleMessage

public class SimpleMessage implements Delayed {
    public static final Object sPoolSync = new Object();
    private static final int MAX_POOL_SIZE = 50;
    private static SimpleMessage sPool;
    private static int sPoolSize = 0;
    public int what;
    public int arg1;
    public int arg2;
    public Object obj;
    public long when;
    Runnable callback;
    SimpleHandler target;
    SimpleMessage next;

    public SimpleMessage() {
    }

    // 创建消息
    public static SimpleMessage obtain() {
        System.out.println("SimpleMessage obtain()              CURRENT THREAD: " + Thread.currentThread());
        synchronized (sPoolSync) {
            if (sPool != null) {
                SimpleMessage m = sPool;
                sPool = m.next;
                m.next = null;
                sPoolSize--;
                return m;
            }
        }
        return new SimpleMessage();
    }

    // 回收消息
    public void recycle() {
        System.out.println("SimpleMessage recycle()             CURRENT THREAD: " + Thread.currentThread());
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        when = 0;
        target = null;
        callback = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(when - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed o) {
        return (int) (when - ((SimpleMessage) o).when);
    }
}

类名：SimpleMessageQueue

public class SimpleMessageQueue {
    private DelayQueue<SimpleMessage> mMessages = new DelayQueue<>();

    // 存消息，给消息排序
    void enqueueMessage(SimpleMessage msg) {
        System.out.println("SimpleMessageQueue enqueueMessage() CURRENT THREAD: " + Thread.currentThread());
        mMessages.add(msg);
    }

    // 取消息
    SimpleMessage next() {
        try {
            System.out.println("SimpleMessageQueue next() enter     CURRENT THREAD: " + Thread.currentThread());
            SimpleMessage simpleMessage = mMessages.take();
            System.out.println("SimpleMessageQueue next() exit      CURRENT THREAD: " + Thread.currentThread());
            return simpleMessage;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

是骡子是马，拉出来溜溜

```java

类名：Main

public class Main {
    public static void main(String[] args) {
        System.out.println("Main main()                         CURRENT THREAD: " + Thread.currentThread());

        SimpleLooperThread looperThread = new SimpleLooperThread("SimpleLooperThread");
        looperThread.start();
        while (looperThread.mHandler == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        SimpleMessage message = SimpleMessage.obtain();
        message.what = 1024;
        looperThread.mHandler.sendMessage(message);
    }

    static class SimpleLooperThread extends Thread {
        public SimpleHandler mHandler;

        public SimpleLooperThread(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println("SimpleLooperThread run()            CURRENT THREAD: " + Thread.currentThread());

            SimpleLooper.prepare();

            mHandler = new SimpleHandler(SimpleLooper.myLooper()) {
                @Override
                public void handleMessage(SimpleMessage msg) {
                    super.handleMessage(msg);
                    System.out.println("SimpleHandler handleMessage()       CURRENT THREAD: " + Thread.currentThread());
                }
            };

            SimpleLooper.loop();
        }
    }
}

执行结果：

Main main()                         CURRENT THREAD: Thread[main,5,main]
SimpleLooperThread run()            CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleLooper prepare()              CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleLooper loop() enter           CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleMessageQueue next() enter     CURRENT THREAD: Thread[SimpleLooperThread,5,main]  ❶
SimpleMessage obtain()              CURRENT THREAD: Thread[main,5,main]  
SimpleHandler sendMessage()         CURRENT THREAD: Thread[main,5,main]
SimpleHandler enqueueMessage()      CURRENT THREAD: Thread[main,5,main]
SimpleMessageQueue enqueueMessage() CURRENT THREAD: Thread[main,5,main]  ❷
SimpleMessageQueue next() exit      CURRENT THREAD: Thread[SimpleLooperThread,5,main]  ❸
SimpleHandler dispatchMessage()     CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleHandler handleMessage()       CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleMessage recycle()             CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleLooper loop() exit            CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleLooper loop() enter           CURRENT THREAD: Thread[SimpleLooperThread,5,main]
SimpleMessageQueue next() enter     CURRENT THREAD: Thread[SimpleLooperThread,5,main]
```

执行结果是符合预期的，在 ❶ 处因为没有消息，所以阻塞了；在 ❷ 处有消息进来，所以 ❶ 处阻塞的地方通了，即走到  ❸ 处了。

另外，还可以看到从 main 线程发送的 SimpleMessage 被切到 SimpleLooperThread 线程中处理了。


<h2 id="4">四、 主线程的 Looper 为什么不会导致应用 ANR ？</h2>

<h2 id="5">五、 同步屏障</h2>

### 1. 同步屏障是什么？作用是啥？

MessageQueue 中的 Message 是在它的成员变量 mMessages 链表中存储的。同步屏障实际上就是一个 target 为 null  的 Message。当它被插入到 mMessages 链表之后，它之后的同步消息都将被「挡住」，而只执行异步消息。直到同步屏障被移除，mMessages 链表中同步屏障之后的消息才会被执行。

> When the barrier is encountered, later synchronous messages in the queue are stalled (prevented from being executed) until the barrier is released by calling removeSyncBarrier and specifying the token that identifies the synchronization barrier. 

上面提到了同步消息、异步消息，它们分别是什么，又是如何创建的？为什么要做这样的区分呢？接下来，一一说明。

Handler 的 Message 可以分为三类，分别是：

1. 同步消息（普通消息）
2. 异步消息
3. 同步屏障消息


#### 1. 什么是同步消息？如何创建？

同步消息就是平时我们开发时通过调用 new Message()、Message.obtain() 或 Handler.obtainMessage() 获取到的 Message，它的特点是 Message 的 target 不为 null，且 target 是发送它的 Handler 对象。

创建同步消息的方法大致可分为三种：

```java
// 1. new Message()
Message message = new Message()

// 2. Message.obtain()
Message message = Message.obtain()

// 3. Handler.obtainMessage()
Message message = Handler.obtainMessage()
```


#### 2. 什么是异步消息？如何创建？

异步消息和同步消息基本上一样，唯一的不同就是异步 Message 的 flags 属性是 FLAG_ASYNCHRONOUS，同步 Message 的 flags 属性是 ～FLAG_ASYNCHRONOUS。

创建异步消息的方法大致可分为两种：

```java
// 1. 创建 Message，调用 Message 的 setAsynchronous() 方法，setAsynchronous() 方法参数里填 true
Message message = Message.obtain();
message.setAsynchronous(true);

// 2. 创建 Handler 时，将 Handler 的构造方法里的 async 参数设置为 true
// 这样处理之后，在 Handler 的 enqueueMessage() 方法里面，会调用所有待发送 Message 的 setAsynchronous() 方法且 
// setAsynchronous() 方法参数里填 true
Handler handler = new Handler(true);
Message message = Message.obtain();
handler.sendMessage(message);


类名：android.os.Handler

public Handler(boolean async) {
    
    ...
    
}

public Handler(@Nullable Callback callback, boolean async) {
    
    ...
    
}

public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {
    
    ...
    
}
    
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```


#### 3. 什么是同步屏障消息？如何创建？

同步屏障消息其实就是一个 target 为 null  的 Message。当我们通过 Handler post/send Message 的时候，在 Handler 的 enqueueMessage() 方法里面，都会把当前 Handler 设置为待发送 Message 的 target：

```java

类名：android.os.Handler

private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

所以，同步屏障消息我们是创建不了的，因为我们创建的 Message 都会被 Handler 把它的 target 设置为非 null。

简单总结一下：


| 类型 | 特点 |
| --- | --- |
| 同步消息 | target != null && flags == FLAG_ASYNCHRONOUS |
| 异步消息 | target != null && flags == ~FLAG_ASYNCHRONOUS |
| 同步屏障消息 | target == null && flags 不影响 |


#### 4. 为什么要做这样的区分？

考虑一个场景，当我们在子线程通过 Handler 给 UI 线程发了一条消息，让它更新某个 View。而此时 UI 线程的消息队列里面消息已经很多了，那我们发送的这条消息就迟迟得不到处理，进而导致「很卡」，进而导致用户一气之下卸载我们开发的应用。

此时，同步屏障就派上用场了。如果消息队列里面存在了同步屏障消息，那么它就会优先寻找我们想要先处理的消息，把它从队列里面取出来，可以理解为加急处理。那同步屏障机制怎么知道我们想优先处理的是哪条消息呢？如果一条消息如果是异步消息，那同步屏障机制就会优先对它处理。


### 2. 怎么添加同步屏障消息？

怎么添加同步屏障消息呢？很简单，调用 MessageQueue 的 postSyncBarrier() 方法：

```java

类名：android.os.MessageQueue

/**
 * Posts a synchronization barrier to the Looper's message queue.
 *
 * Message processing occurs as usual until the message queue encounters the
 * synchronization barrier that has been posted.  When the barrier is encountered,
 * later synchronous messages in the queue are stalled (prevented from being executed)
 * until the barrier is released by calling {@link #removeSyncBarrier} and specifying
 * the token that identifies the synchronization barrier.
 *
 * This method is used to immediately postpone execution of all subsequently posted
 * synchronous messages until a condition is met that releases the barrier.
 * Asynchronous messages (see {@link Message#isAsynchronous} are exempt from the barrier
 * and continue to be processed as usual.
 *
 * This call must be always matched by a call to {@link #removeSyncBarrier} with
 * the same token to ensure that the message queue resumes normal operation.
 * Otherwise the application will probably hang!
 *
 * @return A token that uniquely identifies the barrier.  This token must be
 * passed to {@link #removeSyncBarrier} to release the barrier.
 *
 * @hide
 */
@UnsupportedAppUsage
@TestApi
public int postSyncBarrier() {
    return postSyncBarrier(SystemClock.uptimeMillis());
}

private int postSyncBarrier(long when) {
    // Enqueue a new sync barrier token.
    // We don't need to wake the queue because the purpose of a barrier is to stall it.
    synchronized (this) {
        // 1. 创建同步屏障消息  ❶
        final int token = mNextBarrierToken++;
        final Message msg = Message.obtain();
        msg.markInUse();
        msg.when = when;
        msg.arg1 = token;

        Message prev = null;
        Message p = mMessages;
        // 2. 如果同步屏障消息的 when 不为 0，则按时间顺序在 mMessages 链表中寻找合适的位置，但不执行插入操作  ❷
        if (when != 0) {
            while (p != null && p.when <= when) {
                prev = p;
                p = p.next;
            }
        }
        if (prev != null) { // invariant: p == prev.next
            // 3. 将同步屏障消息插入到上一步找到的（ mMessages 链表中的）合适的位置  ❸
            msg.next = p;
            prev.next = msg;
        } else {
            // 4. 将同步屏障消息插入 mMessages 链表的头部  ❹
            msg.next = p;
            mMessages = msg;
        }
        return token;
    }
}
```

这样处理之后，在 MessageQueue 的 next() 方法中，同步屏障消息之后的同步消息就会被「挡住」，只执行异步消息：

```java

类名：android.os.MessageQueue

Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            // 5. 发现同步屏障消息  ❺
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                // 6. 在 mMessages 链表中寻找异步消息，直至找到第一条异步消息，或遍历完链表中的所有元素  ❻
                //     - 如果找到了，则继续后面的处理逻辑，也就是处理异步消息；
                //     - 如果遍历完了链表中的所有元素都没有找到异步消息，则进入休眠状态，让出 CPU；
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            
            ...
            
        }

        ...

    }
}
```


### 3. 怎么移除同步屏障消息？

怎么移除同步屏障呢？很简单，调用 MessageQueue 的 removeSyncBarrier() 方法：

```java
/**
 * Removes a synchronization barrier.
 *
 * @param token The synchronization barrier token that was returned by
 * {@link #postSyncBarrier}.
 *
 * @throws IllegalStateException if the barrier was not found.
 *
 * @hide
 */
@UnsupportedAppUsage
@TestApi
public void removeSyncBarrier(int token) {
    // Remove a sync barrier token from the queue.
    // If the queue is no longer stalled by a barrier then wake it.
    synchronized (this) {
        Message prev = null;
        Message p = mMessages;
        // 1. 在 mMessages 链表中寻找同步屏障消息  ❶
        while (p != null && (p.target != null || p.arg1 != token)) {
            prev = p;
            p = p.next;
        }
        // 2. 未找到  ❷
        // 要么是同步屏障消息还没有发送，要么是同步屏障消息已经被移除了
        if (p == null) {
            throw new IllegalStateException("The specified message queue synchronization "
                    + " barrier token has not been posted or has already been removed.");
        }
        final boolean needWake;
        if (prev != null) {
            // 3. 如果同步屏障消息在 mMessages 链表的中间，把同步屏障消息的前一个节点的 next 指针指向同步屏障消息的下一个节点  ❸
            prev.next = p.next;
            needWake = false;
        } else {
            // 4. 如果同步屏障消息在 mMessages 链表的头部，同步屏障消息的下一个节点更新为 mMessages 链表的头节点  ❹
            mMessages = p.next;
            needWake = mMessages == null || mMessages.target != null;
        }
        // 5. 回收用完的同步屏障消息  ❺
        p.recycleUnchecked();

        // If the loop is quitting then it is already awake.
        // We can assume mPtr != 0 when mQuitting is false.
        if (needWake && !mQuitting) {
            nativeWake(mPtr);
        }
    }
}
```


### 4. Android 系统是如何应用同步屏障消息的？

Activity 在显示的时候，经过一系列的方法调用，最终会调用 ViewRootImpl 的 requestLayout() 方法。在 ViewRootImpl 的 requestLayout() 方法里面，又会调用 ViewRootImpl 的 scheduleTraversals() 方法。在 ViewRootImpl 的 scheduleTraversals() 方法里面，会先添加同步屏障消息，再发送异步消息，最后移除同步屏障消息。

```java

类名：android.view.ViewRootImpl

public final class ViewRootImpl implements ViewParent, View.AttachInfo.Callbacks, ThreadedRenderer.DrawCallbacks, AttachedSurfaceControl {

    ...
    
    @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }
    
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            // 1. 添加同步屏障消息
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            // 2. 发送异步消息
            // Choreographer post 的 TraversalRunnable 会在下一个 VSYNC 信号到来的时候执行
            // Posts a callback to run on the next frame.
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
    
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
    final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
    
    void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            // 3. 移除同步屏障消息
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            // 4. 执行 View 三大流程：测量，布局，绘制
            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
    
    private void performTraversals() {
    
        ...
        
        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
        
        ...
        
        performLayout(lp, mWidth, mHeight);
        
        ...
        
        performDraw();
        
        ...
    
    }
    
    ...
    
}
```


<h2 id="6"><del>六、 Android 中为什么非 UI 线程不能更新 UI ？</del></h2>

<h2 id="7">七、 参考链接</h2>

1. [Android Java Looper 机制](https://yuandaimaahao.github.io/AndroidFrameworkTutorialPages/004.基础组件篇/007.Android%20Java%20Looper%20机制.html)
2. [Handler 同步屏障机制](https://yuandaimaahao.github.io/AndroidFrameworkTutorialPages/004.基础组件篇/008.Handler%20同步屏障机制.html)
3. [终于搞明白了什么是同步屏障](https://juejin.cn/post/7258850748150104120)
4. [“终于懂了” 系列：Android屏幕刷新机制—VSync、Choreographer 全面理解！](https://cloud.tencent.com/developer/article/1685247)

