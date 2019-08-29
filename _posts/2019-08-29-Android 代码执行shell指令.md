---
layout: post
title: "Android通过代码执行adb shell指令"
tags: [android,adb,shell,tinycap]
comments: true
---

tinycap通过adb shell指令直接抓取录音文件，现在转换为通过代码方式执行
```java
// exec指令
String command = "tinycap /sdcard/1111.wav -D 0 -d 0 -r 48000 -c 2 -b 16";
try {
    Process process = Runtime.getRuntime().exec(command);
} catch (IOException e) {
    e.printStackTrace();
}
printMessage(process.getInputStream());
printMessage(process.getErrorStream());
```
```java
// ctrl+c
try {
    Process p = Runtime.getRuntime().exec("kill -2 " + getTinyCapPID());
} catch (IOException e) {
    e.printStackTrace();
}
```
```java
private static void printMessage(final InputStream input) {
    new Thread(new Runnable() {
    @Override
    public void run() {
        Reader reader = new InputStreamReader(input);
        BufferedReader bf = new BufferedReader(reader);
        String line = null;
        try {
            while((line=bf.readLine())!=null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    }).start();
}
```
因为adb shell后，执行tinymix打开相关流，然后tinycap开始录音，然后ctrl+c，结束录音，录音文件是ok的，可以播放。

那么我们的解决思路是去提前停止tinycap所对应的进程(Runtime.getRuntime().exec执行这行命令时产生的进程)

尝试：直接拿到这个进程p.destroy().失败了，虽然tinycap停止了，但是生成的wav音频文件播放不了。

这就说明，ctrl+c这个操作实际并不是强杀进程。

上网查询了解到ctrl+c实际发送了SIGINT这个信号。那么我们尝试着去模拟这个信号，用kill命令可以。

kill-2 pid。注意是-2，正好对应SIGINT，kill命令的相关参数可以上网查询。

那么第一步，我们要先拿到tinycap的pid。

尝试了几种方法：

1. 反射，stackoverflow上找到的 fail
```java
public static int getPid(Process p) {
    int pid = -1;
 
    try {
        Field f = p.getClass().getDeclaredField("pid");
        f.setAccessible(true);
        pid = f.getInt(p);
        f.setAccessible(false);
    } catch (Throwable e) {
        pid = -1;
    }
    return pid;
}
```
链接：https://stackoverflow.com/questions/13055794/android-runtime-getruntime-exec-get-process-id
虽然用的字段是pid，但是实测实际获取的却是ppid，绝望。java.lang.Process这个类往上追，追到ProcessImpl再到UNIXProcess.除了pid这个看起来有用的字段，没有别的可以获取pid的字段了。所以fail

2. 用RunningTaskInfo fail

实测机器是8.0上只能拿到systemui，launcher和当前应用。所以fail

3. 用RunningAppProcessInfo fail

只要AndroidManifest申明权限android.permission.REAL_GET_TASKS，并且是系统应用，就可以拿到所有应用的pid。

但是注意了，是应用。都是类似com.android.settings的进程，但是tinycap这种Runtime.getRuntime().exec生成的进程不在此列。所以fail

4. 模拟ps | grep tinycap 去获取pid success
```java
public String getTinyCapPID() {

    java.lang.Process psProcess = null;
    try {
        psProcess = Runtime.getRuntime().exec("sh");
    } catch (IOException e) {
        e.printStackTrace();
    }
    DataOutputStream out = new DataOutputStream(psProcess.getOutputStream());
    InputStream is = psProcess.getInputStream();

    try {
        out.writeBytes("ps | grep 'tinycap' | cut -c 10-15\n");
        out.writeBytes("ps\n");
        out.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }

    try {
        out.writeBytes("exit\n");
        out.flush();
        psProcess.waitFor();
    } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    String re="";
    try {
        if (is.read() != 0) {
            byte firstByte = (byte) is.read();
            int available = is.available();
            byte[] characters = new byte[available + 1];
            characters[0] = firstByte;
            is.read(characters, 1, available);
            re = new String(characters);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return re;
}
```