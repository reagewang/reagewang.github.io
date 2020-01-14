---
layout: post
title: "ThreadLocal的使用"
tags: [thread,threadlocal]
comments: true
---

ThreadLocal的使用场景

# 场景1
每个线程需要一个独享对象,每个Thread内有自己的实例副本,不共享.
## 实践场景1
```java
/**
 * 1000个线程打印日期，用线程池来执行，出现线程安全问题
 * 多个线程打印自己的时间（如果线程超级多就会产生性能问题），所以要使用线程池。
 * 但是使用线程池时就会发现每个线程都有一个自己的SimpleDateFormat对象，没有必要，所以将SimpleDateFormat声明为静态，保证只有一个
 */
public class ThreadLocalNormalUsage03 {
    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    //只创建一次 SimpleDateFormat 对象，避免不必要的资源消耗
    static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            //提交任务
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage03().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT 开始计时
        Date date = new Date(1000 * seconds);
        return dateFormat.format(date);
    }
}
```
运行结果

`出现了秒数相同的打印结果，这显然是不正确的。`

## 出现问题的原因
`多个线程的task指向了同一个SimpleDateFormat对象，SimpleDateFormat是非线程安全的。`
## 解决问题的方案
### 加锁
`格式化代码是在最后一句return dateFormat.format(date);,所以可以为最后一句代码添加synchronized锁`
```java
public String date(int seconds) {
    //参数的单位是毫秒，从1970.1.1 00:00:00 GMT 开始计时
    Date date = new Date(1000 * seconds);
    String s;
    synchronized (ThreadLocalNormalUsage03.class) {
        s = dateFormat.format(date);
    }
    return s;
}
```
运行结果

`运行结果中没有发现相同的时间，达到了线程安全的目的`

`缺点：因为添加了synchronized，所以会保证同一时间只有一条线程可以执行，这在高并发场景下肯定不是一个好的选择，所以看看其他方案吧。`
### 使用ThreadLocal
```java
/**
 * 利用 ThreadLocal 给每个线程分配自己的 dateFormat 对象
 * 不但保证了线程安全，还高效的利用了内存
 */
public class ThreadLocalNormalUsage05 {
    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            //提交任务
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage05().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT 开始计时
        Date date = new Date(1000 * seconds);
        //获取 SimpleDateFormat 对象
        SimpleDateFormat dateFormat = ThreadSafeFormatter.dateFormatThreadLocal.get();
        return dateFormat.format(date);
    }
}

class ThreadSafeFormatter {
    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new
            ThreadLocal<SimpleDateFormat>(){
        //创建一份 SimpleDateFormat 对象
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        }
    };
}
```
运行结果

`使用了ThreadLocal后不同的线程不会有共享的 SimpleDateFormat 对象，所以也就不会有线程安全问题`

# 场景2
每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦
## 实践场景2
当前用户信息需要被线程内的所有方法共享
## 解决问题的方案
### 传递参数
可以作为参数在每个方法中进行传递

`缺点：但是这样做会产生代码冗余问题，并且可维护性差。`
### 使用Map
对此进行改进的方案是使用一个Map，在第一个方法中存储信息，后续需要使用直接get()即可

`缺点：如果在单线程环境下可以保证安全，但是在多线程环境下是不可以的。如果使用加锁和ConcurrentHashMap都会产生性能问题。`
### 使用ThreadLocal，实现不同方法间的资源共享
使用 ThreadLocal 可以避免加锁产生的性能问题，也可以避免层层传递参数来实现业务需求，就可以实现不同线程中存储不同信息的要求。
```java
public class ThreadLocalNormalUsage06 {
    public static void main(String[] args) {
        new Service1().process();
    }
}

class Service1 {
    public void process() {
        User user = new User("鲁毅");
        //将User对象存储到 holder 中
        UserContextHolder.holder.set(user);
        new Service2().process();
    }
}

class Service2 {
    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service2拿到用户名: " + user.name);
        new Service3().process();
    }
}

class Service3 {
    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service3拿到用户名: " + user.name);
    }
}

class UserContextHolder {
    public static ThreadLocal<User> holder = new ThreadLocal<>();
}

class User {
    String name;
    public User(String name) {
        this.name = name;
    }
}
```
# 对ThreadLocal的总结
* 让某个需要用到的对象实现线程之间的隔离（每个线程都有自己独立的对象）
* 可以在任何方法中轻松的获取到该对象
* 根据共享对象生成的时机选择使用initialValue方法还是set方法
* 对象初始化的时机由我们控制的时候使用initialValue 方式
* 如果对象生成的时机不由我们控制的时候使用 set 方式

# 使用ThreadLocal的好处
* 达到线程安全的目的
* 不需要加锁，执行效率高
* 更加节省内存，节省开销
* 免去传参的繁琐，降低代码耦合度

# ThreadLocal原理
在Thread类内部有有ThreadLocal.ThreadLocalMap threadLocals = null;这个变量，它用于存储ThreadLocal，因为在同一个线程当中可以有多个ThreadLocal，并且多次调用get()所以需要在内部维护一个ThreadLocalMap用来存储多个ThreadLocal
## ThreadLocal相关方法
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);

    //获取到了值直接返回resule
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //没有获取到才会进行初始化
    return setInitialValue();
}

private T setInitialValue() {
    //获取initialValue生成的值，并在后续操作中进行set，最后将值返回
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}

public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

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
T initialValue()  该方法用于设置初始值，并且在调用get()方法时才会被触发，所以是懒加载。但是如果在get()之前进行了set()操作，这样就不会调用initialValue()。通常每个线程只能调用一次本方法，但是调用了remove()后就能再次调用

void set(T t) 为这个线程设置一个新值

T get() 获取线程对应的value

void remove() 删除对应这个线程的值

# ThreadLocal注意点
## 内存泄漏
某个对象不会再被使用，但是该对象的内存却无法被收回

强引用：当内存不足时触发GC，宁愿抛出OOM也不会回收强引用的内存

弱引用：触发GC后便会回收弱引用的内存

### 正常情况
当Thread运行结束后，ThreadLocal中的value会被回收，因为没有任何强引用了

### 非正常情况
当Thread一直在运行始终不结束，强引用就不会被回收，存在以下调用链 Thread-->ThreadLocalMap-->Entry(key为null)-->value因为调用链中的 value 和 Thread 存在强引用，所以value无法被回收，就有可能出现OOM。

JDK的设计已经考虑到了这个问题，所以在set()、remove()、resize()方法中会扫描到key为null的Entry，并且把对应的value设置为null，这样value对象就可以被回收。
```java
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            //当ThreadLocal为空时，将ThreadLocal对应的value也设置为null
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }

    setThreshold(newLen);
    size = count;
    table = newTab;
}
```
> 但是只有在调用set()、remove()、resize()这些方法时才会进行这些操作，如果没有调用这些方法并且线程不停止，那么调用链就会一直存在，所以可能会发生内存泄漏。

## 如何避免内存泄漏（阿里规约）
调用remove()方法，就会删除对应的Entry对象，可以避免内存泄漏，所以使用完ThreadLocal后，要调用remove()方法。
```java
class Service1 {

    public void process() {
        User user = new User("鲁毅");
        //将User对象存储到 holder 中
        UserContextHolder.holder.set(user);
        new Service2().process();
    }
}

class Service2 {

    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service2拿到用户名: " + user.name);
        new Service3().process();
    }
}

class Service3 {

    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service3拿到用户名: " + user.name);
        //手动释放内存，从而避免内存泄漏
        UserContextHolder.holder.remove();
    }
}
```
## ThreadLocal的空指针异常问题
```java
/**
 * ThreadLocal的空指针异常问题
 */
public class ThreadLocalNPE {

    ThreadLocal<Long> longThreadLocal = new ThreadLocal<>();

    public void set() {
        longThreadLocal.set(Thread.currentThread().getId());
    }

    public Long get() {
        return longThreadLocal.get();
    }

    public static void main(String[] args) {

        ThreadLocalNPE threadLocalNPE = new ThreadLocalNPE();

        //如果get方法返回值为基本类型，则会报空指针异常，如果是包装类型就不会出错
        System.out.println(threadLocalNPE.get());

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocalNPE.set();
                System.out.println(threadLocalNPE.get());
            }
        });
        thread1.start();
    }
}
```
## 空指针异常问题的解决
如果get方法返回值为基本类型，则会报空指针异常，如果是包装类型就不会出错。这是因为基本类型和包装类型存在装箱和拆箱的关系，造成空指针问题的原因在于使用者。

## 共享对象问题
如果在每个线程中ThreadLocal.set()进去的东西本来就是多个线程共享的同一对象，比如static对象，那么多个线程调用ThreadLocal.get()获取的内容还是同一个对象，还是会发生线程安全问题。

## 可以不使用ThreadLocal就不要强行使用
如果在任务数很少的时候，在局部方法中创建对象就可以解决问题，这样就不需要使用ThreadLocal。

## 优先使用框架的支持，而不是自己创造
例如在Spring框架中，如果可以使用RequestContextHolder，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄漏。


