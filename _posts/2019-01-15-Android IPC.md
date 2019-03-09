---
layout: post
title: Android IPC
tags: [ipc,android,service,binder,messager,aidl,eventbus]
comments: true
---

Service/Binder/Messager/AIDL/EventBus

## startService
### 启动
在其他组件中调用startService()方法后，服务即处于启动状态
### 停止
service中调用stopSelf()方法，或者其他组件调用stopService()方法后，service将停止运行
### 通讯方式
没有提供默认的通信方式，启动service后该service就处于独立运行状态
### 生命周期
一旦启动，service即可在后台无限期运行，即使启动service的组件已被销毁也不受其影响，直到其被停止
## bindService
### 启动
在其他组件中调用bindService()方法后，服务即处于启动状态。
### 停止
所有与service绑定的组件都被销毁，或者它们都调用了unbindService()方法后，service将停止运行
### 通讯方式
可以通过 ServiceConnection进行通信，组件可以与service进行交互、发送请求、获取结果，甚至是利用IPC跨进程执行这些操作。
### 生命周期
当所有与其绑定的组件都取消绑定(可能是组件被销毁也有可能是其调用了unbindService()方法)后，service将停止
## 什么时候用startService什么时候用bindService
它们之间的主要区别其实体现在两点，能否交互，以及生命周期。所以很显然的，startService适合那种启动之后不显式停止它就永远在后台运行，并且不需要客户端与服务端交互的service。
而bindService呢，就适合那种可以交互的，可以掌控它什么时候停什么时候开始的。另外，如果有IPC的需求，那当然bindService是必不可少的了。
***==startService()与bindService()并不冲突，同一个service可能既有组件调用了startService()启动它，又有组件与它进行了绑定。当同一个service与其他组件同时存在这两种联系时，其生命周期会发生变化，必须从两种方法的角度看service均停止才能真正停止。==***

## 创建Service
在Manifests文件里进行声明的时候，只有android:name属性是必须要有的，其他的属性都可以没有。
```xml
<service android:name=".ServiceDemo"/>
```
但是有的时候适当的配置可以让我们的开发进行地更加顺利，所以了解一下注册一个service可以声明哪些属性也是很有必要的。
```
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
### 继承Service
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

	START_NOT_STICKY : 如果系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。
	这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

	START_STICKY : 如果系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但绝对不会重新传递最后一个 Intent。
	相反，除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。
	这适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。
	
	START_REDELIVER_INTENT : 如果系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。
	任何挂起 Intent 均依次传递。这适用于主动执行应该立即恢复的作业（例如下载文件）的服务。

除此之外，继承service来实现的service，一定要记得，停止这个service。因为除非系统必须回收内存资源，否则系统不会停止或销毁service。事实上，如果这个service是与可见组件绑定，其几乎永远不会被停止或销毁。在这种情况下，如果你忘记了在其工作完成之后将其停止，势必会造成系统资源的浪费以及电池电量的消耗，而这应当是我们要极力避免的。

### 继承IntentService
如果确定了使用这种方式启动service并且不希望这个service被绑定的话，那么也许除了传统的创建一个类继承service之外我们有一个更好的选择==继承IntentService==。

只需要实现onHandleIntent()方法来完成具体的功能逻辑就可以了。

创建默认的工作线程，用于在应用的主线程外执行传递给 onStartCommand() 的所有 Intent。

创建工作队列，用于将一个 Intent 逐一传递给 onHandleIntent() 实现，这样的话就永远不必担心多线程问题了。

在处理完所有启动请求后停止服务，再也不用担心我忘记调用 stopSelf() 了。

提供 onBind() 的默认实现（返回 null）。

提供 onStartCommand() 的默认实现，可将 Intent 依次发送到工作队列和 onHandleIntent() 实现。

==但是要注意的是，如果需要重写其他的方法，比如onDestroy()方法，一定不要删掉它的超类实现！因为它的超类实现里面也许包括了对工作线程还有工作队列的初始化以及销毁等操作。==


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
### 区别
如果是继承Service类的话，通常情况下我们需要新建一个用于执行工作的新线程，因为默认情况下service将工作于应用的主线程，而这将会降低所有正在运行的Activity的性能。而IntentService就不同了。它是Service的子类，它使用工作线程来注意的处理所有的startService请求。
# Service生命周期
当服务与所有客户端之间的绑定全部取消时，Android 系统便会销毁这个服务（除非还使用 onStartCommand() 启动了该服务）。

因此，如果服务是纯粹的绑定服务，原则上我们是无需对其生命周期进行管理的，Android 系统会根据它是否绑定到任何客户端帮我们管理。

但实际上，我们应该始终在完成与服务的交互时或 Activity 暂停时取消绑定，以便服务能够在未被占用时关闭。

如果你只需要在 Activity 可见时与服务交互，则可以在 onStart() 期间绑定，在 onStop() 期间取消绑定。

如果你希望 Activity 在后台停止运行状态下仍可接收响应，则可在 onCreate() 期间绑定，在 onDestroy() 期间取消绑定。

但是注意，这意味着你的 Activity 在其整个运行过程中（甚至包括后台运行期间）都需要使用服务，因此如果服务位于其他进程内，那么当你提高该进程的权重时，系统终止该进程的可能性会增加。

通常情况下，切勿在 Activity 的 onResume() 和 onPause() 期间绑定和取消绑定，因为每一次生命周期转换都会发生这些回调，我们应该使发生在这些转换期间的处理保持在最低水平。

此外，如果我们的应用内的多个 Activity 绑定到同一服务，并且其中两个 Activity 之间发生了转换，则如果当前 Activity 在下一次绑定（恢复期间）之前取消绑定（暂停期间），系统可能会销毁服务并重建服务。

此外，如果我们的服务已启动并接受绑定，则当系统调用 onUnbind() 方法时，如果我们想在客户端下一次绑定到服务时接收 onRebind() 调用（而不是接收 onBind() 调用），则可选择返回 true。onRebind() 返回空值，但客户端仍在其 onServiceConnected() 回调中接收 IBinder。
# bindService
这是一种比startService更复杂的启动方式，同时使用这种方式启动的service也能完成更多的事情，比如其他组件可向其发送请求，接受来自它的响应，甚至通过它来进行IPC等等。我们通常将绑定它的组件成为客户端，而称它为服务器。

如果要创建一个支持绑定的service，我们必须要重写它的onBind()方法。这个方法会返回一个IBinder对象，它是客户端用来和服务器进行交互的接口。而要得到IBinder接口，我们通常有三种方式：继承Binder类，使用Messenger类，使用AIDL。

要完成客户端与服务端的绑定，有两件事要做。一是在客户端完成bindService的调用以及相关配置，二是在服务端里面实现onBind()方法的重写，返回一个用做信息交互的IBinder接口。

## 客户端的配置

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
## 服务端的配置
那么这个IBinder接口是什么呢？它是一个在整个Android系统中都非常重要的东西，是为高性能而设计的轻量级远程调用机制的核心部分。当然，它不仅仅可以用于远程调用，也可以用于进程内调用——事实上，我们现在所说的service这里的IBinder既有可能出现远程调用的场景，比如用它来进行IPC，也有可能出现进程内调用的场景，比如用它来进行同进程内客户端与服务器的交互。在这里只需要知道它是客户端用来和服务器进行交互的接口，并且知道可以怎样通过IBinder来实现它们的交互就可以了。
一般来讲，我们有三种方式可以获得IBinder的对象：继承Binder类，使用Messenger类，使用AIDL。
### 继承Binder类
Binder又是什么？它实现了IBinder接口，通过实现Binder类，我们的客户端可以直接通过这个类调用服务端的公有方法。另外，虽然从IPC的角度来讲，Binder是Android中的一种跨进程通信方式，但是其实一般service里面的Binder是不会涉及进程间通信的，所以其在这种情况下显得较为简单

	通过继承Binder类实现客户端与服务端通信应该怎样做：
	在service类中，创建一个满足以下任一要求的Binder实例
		包含客户端可调用的公共方法
		返回当前Service实例，其中包含客户端可调用的公共方法
		返回由当前service承载的其他类的实例，其中包含客户端可调用的公共方
	在onBind()方法中返回这个Binder实例
	在客户端中通过onServiceDisconnected()方法接收传过去的Binder实例，并通过它提供的方法进行后续操作

可以看到，在使用这种方法进行客户端与服务端之间的交互是需要有一个强制类型转换的——在onServiceDisconnected()中获得一个经过转换的IBinder对象，我们必须将其转换为service类中的Binder实例的类型才能正确的调用其方法。而这强制类型转换其实就隐含了一个使用这种方法的条件：客户端和服务端应当在同一个进程中！不然在类型转换的时候也许会出现问题——在另一个进程中一定有这个Binder实例么？没有的话就不能完成强制类型转换。
#### Google官方的例子

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
### 使用Messenger
大家看到Messenger可能会很轻易的联想到Message，然后很自然的进一步联想到Handler——没错，Messenger的核心其实就是Message以及Handler来进行线程间的通信。下面讲一下通过这种方式实现IPC的步骤：

	服务端实现一个Handler，由其接受来自客户端的每个调用的回调
	使用实现的Handler创建Messenger对象
	通过Messenger得到一个IBinder对象，并将其通过onBind()返回给客户端
	客户端使用 IBinder 将 Messenger（引用服务的 Handler）实例化，然后使用后者将 Message 对象发送给服务
	服务端在其 Handler 中（具体地讲，是在 handleMessage() 方法中）接收每个 Message

用这种方式，客户端并没有像扩展Binder类那样直接调用服务端的方法，而是采用了用Message来传递信息的方式达到交互的目的。接下来是一个简单的例子：
```java
//服务端
public class MessengerServiceDemo extends Service {

    static final int MSG_SAY_HELLO = 1;

    class ServiceHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_SAY_HELLO:
                    //当收到客户端的message时，显示hello
                    Toast.makeText(getApplicationContext(), "hello!", Toast.LENGTH_SHORT).show();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    final Messenger mMessenger = new Messenger(new ServiceHandler());

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Toast.makeText(getApplicationContext(), "binding", Toast.LENGTH_SHORT).show();
        //返回给客户端一个IBinder实例
        return mMessenger.getBinder();
    }
}
```
服务端主要是返给客户端一个IBinder实例，以供服务端构造Messenger，并且处理客户端发送过来的Message。当然，不要忘了要在Manifests文件里面注册：
```xml
<service
    android:name=".ActivityMessenger"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.lypeer.messenger"></action>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</service>
```
可以看到，这里注册的就和我们原先注册的有一些区别了，主要是因为我们在这里要跨进程通信，所以在另外一个进程里面并没有我们的service的实例，此时必须要给其他的进程一个标志，这样才能让其他的进程找到我们的service。讲道理，其实这里的android:exported属性不设置也可以的，因为在有intent-filter的情况下这个属性默认就是true。
```java
//客户端
public class ActivityMessenger extends Activity {

    static final int MSG_SAY_HELLO = 1;

    Messenger mService = null;
    boolean mBound;

    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            //接收onBind()传回来的IBinder，并用它构造Messenger
            mService = new Messenger(service);
            mBound = true;
        }

        public void onServiceDisconnected(ComponentName className) {
            mService = null;
            mBound = false;
        }
    };

    //调用此方法时会发送信息给服务端
    public void sayHello(View v) {
        if (!mBound) return;
        //发送一条信息给服务端
        Message msg = Message.obtain(null, MSG_SAY_HELLO, 0, 0);
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        //绑定服务端的服务，此处的action是service在Manifests文件里面声明的
        Intent intent = new Intent();
        intent.setAction("com.lypeer.messenger");
        //不要忘记了包名，不写会报错
        intent.setPackage("com.lypeer.ipcserver");
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
}
```
客户端就主要是发起与服务端的绑定，以及通过onServiceConnected()方法来过去服务端返回来的IBinder，借此构造Messenger，从而可以通过发送Message的方式与服务端进行交互。上面的例子其实并不完整，因为它只有客户端对服务端单方面的通信，而服务端没有发信息给客户端的功能——这显然是不合理的。而要实现这个其实也很简单，只要客户端里也创建一个Handler实例，让它接收来自服务端的信息，同时让服务端在客户端给它发的请求完成了之后再给客户端发送一条信息即可。用Messenger来进行IPC的话整体的流程是非常清晰的，Message在其中起到了一个信使的作用，通过它客户端与服务端的信息得以互通。
### 使用AIDL
AIDL，即Android Interface Definition Language，Android接口定义语言。它是一种IDL语言，可以拿来生成用于IPC的代码。在我看来，它其实就是一个模板。为什么这样说呢？在我们的使用中，实际上起作用的并不是我们写的AIDL代码，而是系统根据它生成的一个IInterface实例的代码。而如果大家多生成几个这样的实例，然后把它们拿来比较，你会发现它们都是有套路的——都是一样的流程，一样的结构，只是根据具体的AIDL文件的不同有细微的变动。所以其实AIDL就是为了避免我们一遍遍的写一些千篇一律的代码而出现的一个模板。

首先我们知道的第一点就是：AIDL是一种语言。既然是一种语言，那么相应的就很自然的衍生出了一些问题：

	为什么要设计出这么一门语言？
	它有哪些语法？
	我们应该如何使用它？
	再深入一点，我们可以思考，我们是如何通过它来达到我们的目的的？
	更深入一点，为什么要这么设计这门语言？会不会有更好的方式来实现我们的目的？

那么如何使用AIDL来通过bindService()进行线程间通信呢？基本上有下面这些步骤：

	服务端创建一个AIDL文件，将暴露给客户端的接口在里面声明
	在service中实现这些接口
	客户端绑定服务端，并将onServiceConnected()得到的IBinder转为AIDL生成的IInterface实例
	通过得到的实例调用其暴露的方法

#### 为什么要设计这门语言？
设计这门语言的目的是为了实现进程间通信，尤其是在涉及多进程并发情况下的进程间通信。

每一个进程都有自己的Dalvik VM实例，都有自己的一块独立的内存，都在自己的内存上存储自己的数据，执行着自己的操作，都在自己的那片狭小的空间里过完自己的一生。每个进程之间都你不知我，我不知你，就像是隔江相望的两座小岛一样，都在同一个世界里，但又各自有着自己的世界。而AIDL，就是两座小岛之间沟通的桥梁。相对于它们而言，我们就好像造物主一样，我们可以通过AIDL来制定一些规则，规定它们能进行哪些交流——比如，它们可以在我们制定的规则下传输一些特定规格的数据。

总之，通过这门语言，我们可以愉快的在一个进程访问另一个进程的数据，甚至调用它的一些方法，当然，只能是特定的方法。

#### 它有哪些语法？
其实AIDL这门语言非常的简单，基本上它的语法和 Java 是一样的，只是在一些细微处有些许差别——毕竟它只是被创造出来简化Android程序员工作的，太复杂不好——所以在这里我就着重的说一下它和 Java 不一样的地方。主要有下面这些点：
##### 文件类型：
用AIDL书写的文件的后缀是 .aidl，而不是 .java。
##### 数据类型：
AIDL默认支持一些数据类型，在使用这些数据类型的时候是不需要导包的，但是除了这些类型之外的数据类型，在使用之前必须导包，就算目标文件与当前正在编写的 .aidl 文件在同一个包下——在 Java 中，这种情况是不需要导包的。比如，现在我们编写了两个文件，一个叫做 Book.java ，另一个叫做 BookManager.aidl，它们都在 com.lypeer.aidldemo 包下 ，现在我们需要在 .aidl 文件里使用 Book 对象，那么我们就必须在 .aidl 文件里面写上 import com.lypeer.aidldemo.Book; 哪怕 .java 文件和 .aidl 文件就在一个包下。 
##### 默认支持的数据类型包括： 
Java中的八种基本数据类型，包括 byte，short，int，long，float，double，boolean，char。
String 类型。
CharSequence类型。
List类型：List中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable（下文关于这个会有详解）。List可以使用泛型。
Map类型：Map中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。Map是不支持泛型的。
##### 定向tag：
这是一个极易被忽略的点——这里的“被忽略”指的不是大家都不知道，而是很少人会正确的使用它。在我的理解里，定向 tag 是这样的：AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。另外，Java 中的基本类型和 String ，CharSequence 的定向 tag 默认且只能是 in 。还有，请注意，请不要滥用定向 tag ，而是要根据需要选取合适的——要是不管三七二十一，全都一上来就用 inout ，等工程大了系统的开销就会大很多——因为排列整理参数的开销是很昂贵的。
##### 两种AIDL文件：
在我的理解里，所有的AIDL文件大致可以分为两类。一类是用来定义parcelable对象，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。一类是用来定义方法接口，以供系统使用来完成跨进程通信的。可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。 
注：所有的非默认支持数据类型必须通过第一类AIDL文件定义才能被使用。

```java
// Book.aidl
//第一类AIDL文件的例子
//这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
//注意：Book.aidl与Book.java的包名应当是一样的
package com.lypeer.ipcclient;

//注意parcelable是小写
parcelable Book;
```

```java
// BookManager.aidl
//第二类AIDL文件的例子
package com.lypeer.ipcclient;
//导入所需要使用的非默认支持数据类型的包
import com.lypeer.ipcclient.Book;

interface BookManager {

    //所有的返回值前都不需要加任何东西，不管是什么数据类型
    List<Book> getBooks();
    Book getBook();
    int getBookCount();

    //传参时除了Java基本类型以及String，CharSequence之外的类型
    //都需要在前面加上定向tag，具体加什么量需而定
    void setBookPrice(in Book book , int price)
    void setBookName(in Book book , String name)
    void addBookIn(in Book book);
    void addBookOut(out Book book);
    void addBookInout(inout Book book);
}
```
#### 如何使用AIDL文件来完成跨进程通信？
在进行跨进程通信的时候，在AIDL中定义的方法里包含非默认支持的数据类型与否，我们要进行的操作是不一样的。如果不包含，那么我们只需要编写一个AIDL文件，如果包含，那么我们通常需要写 n+1 个AIDL文件（ n 为非默认支持的数据类型的种类数）——显然，包含的情况要复杂一些。所以我接下来将只介绍AIDL文件中包含非默认支持的数据类型的情况，至于另一种简单些的情况相信大家是很容易从中触类旁通的。
##### 使数据类实现 Parcelable 接口
由于不同的进程有着不同的内存区域，并且它们只能访问自己的那一块内存区域，所以我们不能像平时那样，传一个句柄过去就完事了——句柄指向的是一个内存区域，现在目标进程根本不能访问源进程的内存，那把它传过去又有什么用呢？所以我们必须将要传输的数据转化为能够在内存之间流通的形式。这个转化的过程就叫做序列化与反序列化。简单来说是这样的：比如现在我们要将一个对象的数据从客户端传到服务端去，我们就可以在客户端对这个对象进行序列化的操作，将其中包含的数据转化为序列化流，然后将这个序列化流传输到服务端的内存中去，再在服务端对这个数据流进行反序列化的操作，从而还原其中包含的数据——通过这种方式，我们就达到了在一个进程中访问另一个进程的数据的目的。

而通常，在我们通过AIDL进行跨进程通信的时候，选择的序列化方式是实现 Parcelable 接口。关于实现 Parcelable 接口之后里面具体有那些方法啦，每个方法是干嘛的啦，这些我就不展开来讲了，那並非这篇文章的重点，我下面主要讲一下如何快速的生成一个合格的可序列化的类（以Book.java为例）。

注：若AIDL文件中涉及到的所有数据类型均为默认支持的数据类型，则无此步骤。因为默认支持的那些数据类型都是可序列化的。
###### 编译器自动生成
首先，创建一个类，正常的书写其成员变量，建立getter和setter并添加一个无参构造，然后 implements Parcelable ，解决完错误之后就不会报错了，这个 Book 类也基本上实现了 Parcelable 接口，可以执行序列化操作了。
但是请注意，这里有一个坑：默认生成的模板类的对象只支持为 in 的定向 tag 。为什么呢？因为默认生成的类里面只有 writeToParcel() 方法，而如果要支持为 out 或者 inout 的定向 tag 的话，还需要实现 readFromParcel() 方法——而这个方法其实并没有在 Parcelable 接口里面，所以需要我们从头写。

```java
/**
 * 参数是一个Parcel,用它来存储与传输数据
 * @param dest
 */
public void readFromParcel(Parcel dest) {
    //注意，此处的读值顺序应当是和writeToParcel()方法中一致的
}
```
像上面这样添加了 readFromParcel() 方法之后，我们的 Book 类的对象在AIDL文件里就可以用 out 或者 inout 来作为它的定向 tag 了。此时，完整的 Book 类的代码是这样的：

```java
package com.lypeer.ipcclient;

import android.os.Parcel;
import android.os.Parcelable;

/**
 * Book.java
 *
 * Created by lypeer on 2016/7/16.
 */
public class Book implements Parcelable{
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    private String name;
    private int price;
    public Book(){}

    public Book(Parcel in) {
        name = in.readString();
        price = in.readInt();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(price);
    }

    /**
     * 参数是一个Parcel,用它来存储与传输数据
     * @param dest
     */
    public void readFromParcel(Parcel dest) {
        //注意，此处的读值顺序应当是和writeToParcel()方法中一致的
        name = dest.readString();
        price = dest.readInt();
    }

    //方便打印数据
    @Override
    public String toString() {
        return "name : " + name + " , price : " + price;
    }
}
```
至此，关于AIDL中非默认支持数据类型的序列化操作就完成了。
##### 书写AIDL文件
首先我们需要一个 Book.aidl 文件来将 Book 类引入使得其他的 AIDL 文件其中可以使用 Book 对象。那么第一步，如何新建一个 AIDL 文件呢？Android Studio已经帮我们把这个集成进去了： new->AIDL->AIDL File，按下鼠标左键就会弹出一个框提示生成AIDL文件了。生成AIDL文件之后，项目的目录会变成这样的：比起以前多了一个叫做 aidl 的包，而且他的层级是和 java 包相同的，并且 aidl 包里默认有着和 java 包里默认的包结构。如何新建AIDL文件说的差不多了，接下来就该写AIDL文件的内容了。

```java
// Book.aidl
//第一类AIDL文件
//这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
//注意：Book.aidl与Book.java的包名应当是一样的
package com.lypeer.ipcclient;

//注意parcelable是小写
parcelable Book;
```

```java
// BookManager.aidl
//第二类AIDL文件
//作用是定义方法接口
package com.lypeer.ipcclient;
//导入所需要使用的非默认支持数据类型的包
import com.lypeer.ipcclient.Book;

interface BookManager {

    //所有的返回值前都不需要加任何东西，不管是什么数据类型
    List<Book> getBooks();

    //传参时除了Java基本类型以及String，CharSequence之外的类型
    //都需要在前面加上定向tag，具体加什么量需而定
    void addBook(in Book book);
}
```
##### 移植相关文件
注意：这里又有一个坑！大家可能注意到了，在 Book.aidl 文件中，我一直在强调：Book.aidl与Book.java的包名应当是一样的。这似乎理所当然的意味着这两个文件应当是在同一个包里面的——事实上，很多比较老的文章里就是这样说的，他们说最好都在 aidl 包里同一个包下，方便移植——然而在 Android Studio 里并不是这样。如果这样做的话，系统根本就找不到 Book.java 文件，从而在其他的AIDL文件里面使用 Book 对象的时候会报 Symbol not found 的错误。为什么会这样呢？因为 Gradle 。大家都知道，Android Studio 是默认使用 Gradle 来构建 Android 项目的，而 Gradle 在构建项目的时候会通过 sourceSets 来配置不同文件的访问路径，从而加快查找速度——问题就出在这里。Gradle 默认是将 java 代码的访问路径设置在 java 包下的，这样一来，如果 java 文件是放在 aidl 包下的话那么理所当然系统是找不到这个 java 文件的。那应该怎么办呢？

又要 java文件和 aidl 文件的包名是一样的，又要能找到这个 java 文件——那么仔细想一下的话，其实解决方法是很显而易见的。首先我们可以把问题转化成：如何在保证两个文件包名一样的情况下，让系统能够找到我们的 java 文件？这样一来思路就很明确了：要么让系统来 aidl 包里面来找 java 文件，要么把 java 文件放到系统能找到的地方去，也即放到 java 包里面去。接下来我详细的讲一下这两种方式具体应该怎么做：

	1:修改 build.gradle 文件：在 android{} 中间加上下面的内容：
	sourceSets {
	    main {
	        java.srcDirs = ['src/main/java', 'src/main/aidl']
	    }
	}
	也就是把 java 代码的访问路径设置成了 java 包和 aidl 包，这样一来系统就会到 aidl 包里面去查找 java 文件，也就达到了我们的目的。
	2:把 java 文件放到 java 包下去：把 Book.java 放到 java 包里任意一个包下，保持其包名不变，与 Book.aidl 一致。只要它的包名不变，Book.aidl 就能找到 Book.java ，而只要 Book.java 在 java 包下，那么系统也是能找到它的。但是这样做的话也有一个问题，就是在移植相关 .aidl 文件和 .java 文件的时候没那么方便，不能直接把整个 aidl 文件夹拿过去完事儿了，还要单独将 .java 文件放到 java 文件夹里去。

我们可以用上面两个方法之一来解决找不到 .java 文件的坑，具体用哪个就看大家怎么选了。
	
##### 编写服务端代码
通过上面几步，我们已经完成了AIDL及其相关文件的全部内容，那么我们究竟应该如何利用这些东西来进行跨进程通信呢？其实，在我们写完AIDL文件并 clean 或者 rebuild 项目之后，编译器会根据AIDL文件为我们生成一个与AIDL文件同名的 .java 文件，这个 .java 文件才是与我们的跨进程通信密切相关的东西。事实上，基本的操作流程就是：在服务端实现AIDL中定义的方法接口的具体逻辑，然后在客户端调用这些方法接口，从而达到跨进程通信的目的。

接下来我直接贴上我写的服务端代码：

```java
/**
 * 服务端的AIDLService.java
 * <p/>
 * Created by lypeer on 2016/7/17.
 */
public class AIDLService extends Service {

    public final String TAG = this.getClass().getSimpleName();

    //包含Book对象的list
    private List<Book> mBooks = new ArrayList<>();

    //由AIDL文件生成的BookManager
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
        public void addBook(Book book) throws RemoteException {
            synchronized (this) {
                if (mBooks == null) {
                    mBooks = new ArrayList<>();
                }
                if (book == null) {
                    Log.e(TAG, "Book is null in In");
                    book = new Book();
                }
                //尝试修改book的参数，主要是为了观察其到客户端的反馈
                book.setPrice(2333);
                if (!mBooks.contains(book)) {
                    mBooks.add(book);
                }
                //打印mBooks列表，观察客户端传过来的值
                Log.e(TAG, "invoking addBooks() method , now the list is : " + mBooks.toString());
            }
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        Book book = new Book();
        book.setName("Android开发艺术探索");
        book.setPrice(28);
        mBooks.add(book);   
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.e(getClass().getSimpleName(), String.format("on bind,intent = %s", intent.toString()));
        return mBookManager;
    }
}
```
整体的代码结构很清晰，大致可以分为三块：第一块是初始化。在 onCreate() 方法里面我进行了一些数据的初始化操作。第二块是重写 BookManager.Stub 中的方法。在这里面提供AIDL里面定义的方法接口的具体实现逻辑。第三块是重写 onBind() 方法。在里面返回写好的 BookManager.Stub 。

接下来在 Manefest 文件里面注册这个我们写好的 Service ，这个不写的话我们前面做的工作都是无用功：

```xml
<service
    android:name=".service.AIDLService"
    android:exported="true">
        <intent-filter>
            <action android:name="com.lypeer.aidl"/>
            <category android:name="android.intent.category.DEFAULT"/>
        </intent-filter>
</service>
```
到这里我们的服务端代码就编写完毕了。
##### 编写客户端代码
前面说过，在客户端我们要完成的工作主要是调用服务端的方法，但是在那之前，我们首先要连接上服务端，完整的客户端代码是这样的：

```java
/**
 * 客户端的AIDLActivity.java
 * 由于测试机的无用debug信息太多，故log都是用的e
 * <p/>
 * Created by lypeer on 2016/7/17.
 */
public class AIDLActivity extends AppCompatActivity {

    //由AIDL文件生成的Java类
    private BookManager mBookManager = null;

    //标志当前与服务端连接状况的布尔值，false为未连接，true为连接中
    private boolean mBound = false;

    //包含Book对象的list
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
    public void addBook(View view) {
        //如果与服务端的连接处于未连接状态，则尝试连接
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
            mBookManager.addBook(book);
            Log.e(getLocalClassName(), book.toString());
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
同样很清晰，首先建立连接，然后在 ServiceConnection 里面获取 BookManager 对象，接着通过它来调用服务端的方法。

##### 测试
通过上面的步骤，我们已经完成了所有的前期工作，接下来就可以通过AIDL来进行跨进程通信了！将两个app同时运行在同一台手机上，然后调用客户端的 addBook() 方法，我们会看到服务端的 logcat 信息是这样的：

	//服务端的 log 信息，我把无用的信息头去掉了，然后给它编了个号
	1，on bind,intent = Intent { act=com.lypeer.aidl pkg=com.lypeer.ipcserver }
	2，invoking getBooks() method , now the list is : [name : Android开发艺术探索 , price : 28]
	3，invoking addBooks() method , now the list is : [name : Android开发艺术探索 , price : 28, name : APP研发录In , price : 2333]

客户端的信息是这样的：

	//客户端的 log 信息
	1，service connected
	2，[name : Android开发艺术探索 , price : 28]
	3，name : APP研发录In , price : 30

所有的 log 信息都很正常并且符合预期——这说明我们到这里为止的步骤都是正确的，按照上面说的来做是能够正确的使用AIDL来进行跨进程通信的。

### Messenger与AIDL的比较
首先，在实现的难度上，肯定是Messenger要简单的多，另外，使用Messenger还有一个显著的好处是它会把所有的请求排入队列，因此你几乎可以不用担心多线程可能会带来的问题。
但是如果项目中有并发处理问题的需求，或者会有大量的并发请求，这个时候Messenger就不适用了——它的特性让它只能串行的解决请求。另外，我们在使用Messenger的时候只能通过Message来传递信息实现交互，但是在有些时候也许我们需要直接跨进程调用服务端的方法，这个时候又怎么办呢？只能使用AIDL。所以，这两种IPC方式各有各的优点和缺点，具体使用哪种就看具体的需求。

# 参考链接
https://blog.csdn.net/luoyanglizi/article/details/51586437
https://blog.csdn.net/luoyanglizi/article/details/51594016
https://blog.csdn.net/luoyanglizi/article/details/51980630
https://blog.csdn.net/luoyanglizi/article/details/52029091
https://blog.csdn.net/luoyanglizi/article/details/51958091