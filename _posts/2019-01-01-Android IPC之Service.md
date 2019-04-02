---
layout: post
title: Android IPC Service
tags: [ipc,android,service]
comments: true
---

StartService/BinderService

# startService
---
## 启动
在其他组件中调用startService()方法后，服务即处于启动状态
## 停止
service中调用stopSelf()方法，或者其他组件调用stopService()方法后，service将停止运行
## 通讯方式
没有提供默认的通信方式，启动service后该service就处于独立运行状态
## 生命周期
一旦启动，service即可在后台无限期运行，即使启动service的组件已被销毁也不受其影响，直到其被停止

# bindService
---
## 启动
在其他组件中调用bindService()方法后，服务即处于启动状态。
## 停止
所有与service绑定的组件都被销毁，或者它们都调用了unbindService()方法后，service将停止运行
## 通讯方式
可以通过 ServiceConnection进行通信，组件可以与service进行交互、发送请求、获取结果，甚至是利用IPC跨进程执行这些操作。
## 生命周期
当所有与其绑定的组件都取消绑定(可能是组件被销毁也有可能是其调用了unbindService()方法)后，service将停止

# 什么时候用startService什么时候用bindService
---
它们之间的主要区别其实体现在两点，能否交互，以及生命周期。所以很显然的，startService适合那种启动之后不显式停止它就永远在后台运行，并且不需要客户端与服务端交互的service。而bindService呢，就适合那种可以交互的，可以掌控它什么时候停什么时候开始的。另外，如果有IPC的需求，那当然bindService是必不可少的了。

**`startService()与bindService()并不冲突，同一个service可能既有组件调用了startService()启动它，又有组件与它进行了绑定。当同一个service与其他组件同时存在这两种联系时，其生命周期会发生变化，必须从两种方法的角度看service均停止才能真正停止。`**

# 创建Service
---
在Manifests文件里进行声明的时候，只有android:name属性是必须要有的，其他的属性都可以没有。
```xml
    <service android:name=".ServiceDemo"/>
```
但是有的时候适当的配置可以让我们的开发进行地更加顺利，所以了解一下注册一个service可以声明哪些属性也是很有必要的。
```xml
<service
    android:enabled=["true" | "false"]
    //如果为true，则这个service可以被系统实例化，如果为false，则不行。默认为true

    android:exported=["true" | "false"]
    //如果为true，则其他应用的组件也可以调用这个service并且可以与它进行互动，如果为false，则只有与service同一个应用或者相同user ID的应用可以开启或绑定此service。它的默认值取决于service是否有intent filters。如果一个filter都没有，就意味着只有指定了service的准确的类名才能调用，也就是说这个service只能应用内部使用——其他的应用不知道它的类名。这种情况下exported的默认值就为false。反之，只要有了一个filter，就意味着service是考虑到外界使用的情况的，这时exported的默认值就为true

    android:icon="drawable resource"
    //一个象征着这个service的icon

    android:isolatedProcess=["true" | "false"]
    //如果设置为true，这个service将运行在一个从系统中其他部分分离出来的特殊进程中，我们只能通过Service API来与它进行交流。默认为false。

    android:label="string resource"
    //显示给用户的这个service的名字。如果不设置，将会默认使用<application>的label属性。

    android:name="string"
    //这个service的路径名，这个属性是唯一一个必须填的属性。

    android:permission="string"
    //其他组件必须具有所填的权限才能启动这个service。
    
    android:process="string" 
    //service运行的进程的name。默认启动的service是运行在主进程中的。
</service>
```
## 继承Service
另一个组件通过调用startService()方法，就可以启动一个特定的service，并且这将导致service中的onStartCommand()方法被调用。在调用startService()方法的时候，其他组件需要在方法中传递一个intent参数，然后service将会在onStartCommand()中接收这个intent，并获取一些数据。

当一个service通过这种方式启动之后，它的生命周期就已经不受启动它的组件影响了，它可以在后台无限期的运行下去，只要service自身没有调用stopSelf()并且其他的组件没有调用针对它的stopService()。

如果要创建一个支持绑定的service，我们必须要重写它的onBind()方法。这个方法会返回一个IBinder对象，它是客户端用来和服务器进行交互的接口。而要得到IBinder接口，我们通常有三种方式：继承Binder类，使用Messenger类，使用AIDL。

```java
public class ServiceDemo extends Service {

    private static final String TAG = "ServiceDome";

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate");
        //在每个service的生命周期中这个方法会且仅会调用一次，并且它的调用在onStartCommand()以及onBond()之前，我们可以在这个方法中进行一些一次性的初始化工作。
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand");
        //当其他组件调用startService()方法时，此方法将会被调用
        //在这里进行这个service主要的操作
        return super.onStartCommand(intent, flags, startId);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind");
        //当其他组件调用bindService()方法时，此方法将会被调用
        //如果不想让这个service被绑定，在此返回null即可     
		//当其他组件通过bindService()方法与service相绑定之后，此方法将会被调用。这个方法有一个IBinder的返回值，这意味着在重写它的时候必须返回一个IBinder对象，它是用来支撑其他组件与service之间的通信的——另外，如果你不想让这个service被其他组件所绑定，可以通过在这个方法返回一个null值来实现。
        return null;
    }

    @Override
    public void onDestroy() {
        Log.d(TAG, "onDestroy");
        //service调用的最后一个方法
        //在此进行资源的回收       
		//这是service一生中调用的最后一个方法，当这个方法被调用之后，service就会被销毁。所以我们应当在这个方法里面进行一些资源的清理，比如注册的一些监听器什么的。
        super.onDestroy();
    }
}
```
我们注意到onStartCommand()的返回值是一个很奇怪的值，这是个什么呢？或者说这个方法的返回值是用来干嘛的呢？事实上，它的返回值是用来指定系统对当前线程的行为的。它的返回值必须是以下常量之一：

* START_NOT_STICKY : 如果系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

* START_STICKY : 如果系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但绝对不会重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。这适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。
	
* START_REDELIVER_INTENT : 如果系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。任何挂起 Intent 均依次传递。这适用于主动执行应该立即恢复的作业（例如下载文件）的服务。

除此之外，继承service来实现的service，一定要记得，停止这个service。因为除非系统必须回收内存资源，否则系统不会停止或销毁service。事实上，如果这个service是与可见组件绑定，其几乎永远不会被停止或销毁。在这种情况下，如果你忘记了在其工作完成之后将其停止，势必会造成系统资源的浪费以及电池电量的消耗，而这应当是我们要极力避免的。

## 继承IntentService
如果确定了使用这种方式启动service并且不希望这个service被绑定的话，那么也许除了传统的创建一个类继承service之外我们有一个更好的选择`继承IntentService`。只需要实现onHandleIntent()方法来完成具体的功能逻辑就可以了。创建默认的工作线程，用于在应用的主线程外执行传递给 onStartCommand() 的所有 Intent。创建工作队列，用于将一个 Intent 逐一传递给 onHandleIntent() 实现，这样的话就永远不必担心多线程问题了。在处理完所有启动请求后停止服务，再也不用担心我忘记调用 stopSelf() 了。提供 onBind() 的默认实现（返回 null）。提供 onStartCommand() 的默认实现，可将 Intent 依次发送到工作队列和 onHandleIntent() 实现。

`但是要注意的是，如果需要重写其他的方法，比如onDestroy()方法，一定不要删掉它的超类实现！因为它的超类实现里面也许包括了对工作线程还有工作队列的初始化以及销毁等操作。`

```java
public class IntentServiceDemo extends IntentService {

    public IntentServiceDemo(String name) {
        super(name);
        //构造方法
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        //在这里根据intent进行操作
    }
}
```
## 区别
如果是继承Service类的话，通常情况下我们需要新建一个用于执行工作的新线程，因为默认情况下service将工作于应用的主线程，而这将会降低所有正在运行的Activity的性能。而IntentService就不同了。它是Service的子类，它使用工作线程来注意的处理所有的startService请求。

# Service生命周期
---
当服务与所有客户端之间的绑定全部取消时，Android 系统便会销毁这个服务（除非还使用 onStartCommand() 启动了该服务）。因此，如果服务是纯粹的绑定服务，原则上我们是无需对其生命周期进行管理的，Android 系统会根据它是否绑定到任何客户端帮我们管理。但实际上，我们应该始终在完成与服务的交互时或 Activity 暂停时取消绑定，以便服务能够在未被占用时关闭。如果你只需要在 Activity 可见时与服务交互，则可以在 onStart() 期间绑定，在 onStop() 期间取消绑定。如果你希望 Activity 在后台停止运行状态下仍可接收响应，则可在 onCreate() 期间绑定，在 onDestroy() 期间取消绑定。但是注意，这意味着你的 Activity 在其整个运行过程中（甚至包括后台运行期间）都需要使用服务，因此如果服务位于其他进程内，那么当你提高该进程的权重时，系统终止该进程的可能性会增加。通常情况下，切勿在 Activity 的 onResume() 和 onPause() 期间绑定和取消绑定，因为每一次生命周期转换都会发生这些回调，我们应该使发生在这些转换期间的处理保持在最低水平。此外，如果我们的应用内的多个 Activity 绑定到同一服务，并且其中两个 Activity 之间发生了转换，则如果当前 Activity 在下一次绑定（恢复期间）之前取消绑定（暂停期间），系统可能会销毁服务并重建服务。此外，如果我们的服务已启动并接受绑定，则当系统调用 onUnbind() 方法时，如果我们想在客户端下一次绑定到服务时接收 onRebind() 调用（而不是接收 onBind() 调用），则可选择返回 true。onRebind() 返回空值，但客户端仍在其 onServiceConnected() 回调中接收 IBinder。

# bindService
---
这是一种比startService更复杂的启动方式，同时使用这种方式启动的service也能完成更多的事情，比如其他组件可向其发送请求，接受来自它的响应，甚至通过它来进行IPC等等。我们通常将绑定它的组件成为客户端，而称它为服务器。如果要创建一个支持绑定的service，我们必须要重写它的onBind()方法。这个方法会返回一个IBinder对象，它是客户端用来和服务器进行交互的接口。而要得到IBinder接口，我们通常有三种方式：继承Binder类，使用Messenger类，使用AIDL。要完成客户端与服务端的绑定，有两件事要做。一是在客户端完成bindService的调用以及相关配置，二是在服务端里面实现onBind()方法的重写，返回一个用做信息交互的IBinder接口。

# 客户端的配置
---
客户端原则上来讲调用bindService()方法就可以了，然而事实并没有这么简单。原因就出在bindService()这个方法身上。
```java
public boolean bindService(Intent service, ServiceConnection conn, int flags) {
    return mBase.bindService(service, conn, flags);
}
```
可以看到，bindService()方法需要三个参数，第一个是一个intent，我们都很熟悉，用来指定启动哪一个service以及传递一些数据过去。第二个参数可能就有点陌生了，这是个啥？这是实现客户端与服务端通信的一个关键类。要想实现它，就必须重写两个回调方法：onServiceConnected()以及onServiceDisconnected()，而我们可以通过这两个回调方法得到服务端里面的IBinder对象，从而达到通信的目的。第三个参数是一个int值，它是用来做什么的呢？它是一个指示绑定选项的标志，通常应该是 BIND_AUTO_CREATE，以便创建尚未激活的服务。 其他可能的值为 BIND_DEBUG_UNBIND 和 BIND_NOT_FOREGROUND，或 0（表示无）。
```java
ServiceDemo mService;
//BinderDemo是在ServiceDemo里面的一个继承了Binder的内部类，这是一种得到IBinder接口的方式
//下文会有详述
ServiceDemo.BinderDemo mBinder;

/**
 * Class used for the client Binder.  Because we know this service always
 * runs in the same process as its clients, we don't need to deal with IPC.
 */
public class BinderDemo extends Binder {
    ServiceDemo getService() {
        // Return this instance of LocalService so clients can call public methods
        return ServiceDemo.this;
    }
}

private ServiceConnection mConnection = new ServiceConnection() {

    //系统会调用该方法以传递服务的　onBind() 方法返回的 IBinder。
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        //当系统调用 onServiceConnected() 回调方法时，我们可以使用接口定义的方法开始调用服务。
        mBinder = (ServiceDemo.BinderDemo) service;
        //getService()是BinderDemo中的一个方法
        mService = mBinder.getService();
        //在此处可以利用得到的ServiceDemo对象调用该类中的构造方法
        Log.d(this.getClass().getSimpleName(), "onServiceConnected");
    }

    //Android系统会在与服务的连接意外中断时（例如当服务崩溃或被终止时）调用该方法。当客户端取消绑定时，系统“绝对不会”调用该方法。
    @Override
    public void onServiceDisconnected(ComponentName name) {
        Log.d(this.getClass().getSimpleName(), "onServiceDisconnected");
    }
};
```
上面的例子实现了一个比较普通的ServiceConnection的主要功能，我们可以通过它得到目标service的对象，然后可以调用其内的共有方法，实现客户端与服务端交互的目的。
# 服务端的配置
---
那么这个IBinder接口是什么呢？它是一个在整个Android系统中都非常重要的东西，是为高性能而设计的轻量级远程调用机制的核心部分。当然，它不仅仅可以用于远程调用，也可以用于进程内调用——事实上，我们现在所说的service这里的IBinder既有可能出现远程调用的场景，比如用它来进行IPC，也有可能出现进程内调用的场景，比如用它来进行同进程内客户端与服务器的交互。在这里只需要知道它是客户端用来和服务器进行交互的接口，并且知道可以怎样通过IBinder来实现它们的交互就可以了。
一般来讲，我们有三种方式可以获得IBinder的对象：继承Binder类，使用Messenger类，使用AIDL。
## 继承Binder类
Binder又是什么？它实现了IBinder接口，通过实现Binder类，我们的客户端可以直接通过这个类调用服务端的公有方法。另外，虽然从IPC的角度来讲，Binder是Android中的一种跨进程通信方式，但是其实一般service里面的Binder是不会涉及进程间通信的，所以其在这种情况下显得较为简单

* 通过继承Binder类实现客户端与服务端通信应该怎样做：
* 在service类中，创建一个满足以下任一要求的Binder实例
    * 包含客户端可调用的公共方法
	* 返回当前Service实例，其中包含客户端可调用的公共方法
	* 返回由当前service承载的其他类的实例，其中包含客户端可调用的公共方
* 在onBind()方法中返回这个Binder实例
* 在客户端中通过onServiceDisconnected()方法接收传过去的Binder实例，并通过它提供的方法进行后续操作

可以看到，在使用这种方法进行客户端与服务端之间的交互是需要有一个强制类型转换的——在onServiceDisconnected()中获得一个经过转换的IBinder对象，我们必须将其转换为service类中的Binder实例的类型才能正确的调用其方法。而这强制类型转换其实就隐含了一个使用这种方法的条件：客户端和服务端应当在同一个进程中！不然在类型转换的时候也许会出现问题——在另一个进程中一定有这个Binder实例么？没有的话就不能完成强制类型转换。
### Google官方的例子
```java
public class LocalService extends Service {
    // Binder given to clients
    private final IBinder mBinder = new LocalBinder();
    // Random number generator
    private final Random mGenerator = new Random();

    /**
     * Class used for the client Binder.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    public class LocalBinder extends Binder {
        LocalService getService() {
            // Return this instance of LocalService so clients can call public methods
            return LocalService.this;
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    /** method for clients */
    public int getRandomNumber() {
      return mGenerator.nextInt(100);
    }
}

public class BindingActivity extends Activity {
    LocalService mService;
    boolean mBound = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Bind to LocalService
        Intent intent = new Intent(this, LocalService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Unbind from the service
        if (mBound) {
            unbindService(mConnection);
            mBound = false;
        }
    }

    /** Called when a button is clicked (the button in the layout file attaches to
      * this method with the android:onClick attribute) */
    public void onButtonClick(View v) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call were something that might hang, then this request should
            // occur in a separate thread to avoid slowing down the activity performance.
            int num = mService.getRandomNumber();
            Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
        }
    }

    /** Defines callbacks for service binding, passed to bindService() */
    private ServiceConnection mConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            LocalBinder binder = (LocalBinder) service;
            mService = binder.getService();
            mBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            mBound = false;
        }
    };
}
```
LocalBinder 为客户端提供 getService() 方法，以检索 LocalService 的当前实例。这样，客户端便可调用服务中的公共方法。 例如，客户端可调用服务中的 getRandomNumber()。当Activity进入onStart()状态时，就会尝试与目标service绑定，而当点击按钮时，如果绑定已经完成，就会调用service中的方法getRandomNumber()，并将其输出。线程内通信基本上就是这样了，没什么复杂的地方。

# 参考链接
---
https://blog.csdn.net/luoyanglizi/article/details/51586437

https://blog.csdn.net/luoyanglizi/article/details/51594016

https://blog.csdn.net/luoyanglizi/article/details/51980630

https://blog.csdn.net/luoyanglizi/article/details/52029091

https://blog.csdn.net/luoyanglizi/article/details/51958091