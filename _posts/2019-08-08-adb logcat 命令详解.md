---
layout: post
title: "adb logcat 命令详解"
tags: [adb,logcat]
comments: true
---

# 帮助信息
---
> adb logcat --help
```
Usage: logcat [options] [filterspecs]
options include:
  -s              Set default filter to silent.
                  Like specifying filterspec '*:s'
  -f <filename>   Log to file. Default to stdout
  -r [<kbytes>]   Rotate log every kbytes. (16 if unspecified). Requires -f
  -n <count>      Sets max number of rotated logs to <count>, default 4
  -v <format>     Sets the log print format, where <format> is one of:

                  brief process tag thread raw time threadtime long

  -c              clear (flush) the entire log and exit
  -o              for -f save file use orginal log no gz log
  -k              enable kernel log, this is enabled by default
  -d              dump the log and then exit (don't block)
  -t <count>      print only the most recent <count> lines (implies -d)
  -g              get the size of the log's ring buffer and exit
  -b <buffer>     Request alternate ring buffer, 'main', 'system', 'radio'
                  or 'events'. Multiple -b parameters are allowed and the
                  results are interleaved. The default is -b main -b system.
  -B              output the log in binary
filterspecs are a series of
  <tag>[:priority]

where <tag> is a log component tag (or * for all) and priority is:
  V    Verbose
  D    Debug
  I    Info
  W    Warn
  E    Error
  F    Fatal
  S    Silent (supress all output)

'*' means '*:d' and <tag> by itself means <tag>:v

If not specified on the commandline, filterspec is set from ANDROID_LOG_TAGS.
If no filterspec is found, filter defaults to '*:I'

If not specified with -v, format is set from ANDROID_PRINTF_LOG
or defaults to "brief"
```
## adb logcat 命令格式 : 
adb logcat [options] [filterspecs]

其中 options<选项> 和 filterspecs<过滤项> 在 中括号 [] 中, 说明这是可选的.

## 选项解析
* "-s"选项 : 设置输出日志的标签, 只显示该标签的日志;
* "-f"选项 : 将日志输出到文件, 默认输出到标准输出流中, -f 参数执行不成功;
* "-r"选项 : 按照每千字节输出日志, 需要 -f 参数, 不过这个命令没有执行成功;
* "-n"选项 : 设置日志输出的最大数目, 需要 -r 参数, 这个执行 感觉 跟 adb logcat 效果一样;
* "-v"选项 : 设置日志的输出格式, 注意只能设置一项;
* "-c"选项 : 清空所有的日志缓存信息;
* "-d"选项 : 将缓存的日志输出到屏幕上, 并且不会阻塞;
* "-t"选项 : 输出最近的几行日志, 输出完退出, 不阻塞;
* "-g"选项 : 查看日志缓冲区信息;
* "-b"选项 : 加载一个日志缓冲区, 默认是 main, 下面详解;
* "-B"选项 : 以二进制形式输出日志;

### 输出指定标签内容 "-s"选项
> adb logcat -s tag

### 输出日志信息到文件 "-f"选项或">"输出
> 该选向后面跟着输入日志的文件, 使用 adb logcat -f log 命令, 会出现错误, 这里我们不推荐使用该选项,此方式命令会出现couldn't open output file: Read-only file system 错误

> ">" 后面跟着要输出的日志文件, 可以将 logcat 日志输出到文件中, 使用 adb logcat > log 命令, 使用 more log 命令查看日志信息

### 指定 logcat 的日志输出格式 "-v"选项
* "brief"格式 : 这是默认的日志格式 "优先级/标签(进程ID):日志信息"
> adb logcat -v prief
* "process"格式 : " 优先级 (进程ID) : 日志信息 "
> adb logcat -v process
* "tag"格式 : " 优先级 / 标签 : 日志信息"
> adb logcat -v tag
* "thread"格式 : " 优先级 ( 进程ID : 线程ID) 标签 : 日志内容 "
> adb logcat -v thread
* "raw"格式 : 只输出日志信息, 不附加任何其他 信息, 如 优先级 标签等
> adb logcat -v raw
* "time"格式 : "日期 时间 优先级 / 标签 (进程ID) : 日志信息"
> adb logcat -v time 
* "threadtime"格式 : "日期 时间 优先级 / 标签 (进程ID) : 进程名称 : 日志信息 " 
> adb logcat -v threadtime
* "long"格式 : " [ 日期 时间 进程ID : 线程ID 优先级 / 标签] 日志信息 ", 输出以上提到的所有的头信息
> adb logcat -v long

### 清空日志缓存信息
> 使用 adb logcat -c 命令, 可以将之前的日志信息清空, 重新开始输出日志信息

### 将缓存日志输出
> 使用 adb logcat -d 命令, 输出命令, 之后推出命令, 不会进行阻塞

### 输出最近的日志
> 使用 adb logcat -t 5 命令, 可以输出最近的5行日志, 并且不会阻塞

### 查看日志缓冲区信息
> 使用 adb logcat -g 命令

### 加载日志缓冲区
> 使用 adb logcat -b 缓冲区类型 命令
* system缓冲区 - 与系统相关的日志信息
* radio缓冲区 - 广播电话相关的日志信息
* events缓冲区 - 事件相关的日志信息
* main缓冲区 - 默认的缓冲区

### 以二进制形式输出日志
> 使用 adb logcat -B 命令

## 过滤项解析
```
<tag>[:priority] 
标签:日志等级
默认的日志过滤项是 " *:I " ;
-- V : Verbose (明细);
-- D : Debug (调试);
-- I : Info (信息);
-- W : Warn (警告);
-- E : Error (错误);
-- F : Fatal (严重错误);
-- S : Silent(Super all output) (最高的优先级, 可能不会记载东西);
```

### 过滤指定等级日志

使用 adb logcat 10 *:E 命令, 显示 Error 以上级别的日志

### 过滤指定标签等级日志
使用 adb logcat WifiHW:D *:S 命令进行过滤

命令含义 : 输出10条日志, 日志是 标签为 WifiHW, 并且优先级 Debug(调试) 等级以上的级别的日志;

注意 *:S : 如果没有 *S 就会输出错误;

### 可以同时设置多个过滤器
使用 adb logcat WifiHW:D dalvikvm:I *:S 命令, 输出 WifiHW 标签 的 Debug 以上级别 和 dalvikvm 标签的 Info 以上级别的日志

## 使用管道过滤日志
### 过滤固定字符串
adb logcat | grep Wifi 只要命令行出现的日志都可以过滤, 不管是不是标签

### 过滤字符串忽略大小写
adb logcat | grep -i wifi

## 使用正则表达式匹配
### 正则表达式过滤日志
V/ActivityManager(  574): getTasks: max=1, flags=0, receiver=null  
分析日志 : 该日志开头两个字符是 "V/", 后面开始就是标签, 写一个正则表达式 "^..ActivityManager", 就可以匹配日志中的 "V/ActivityManager" 字符串;  
使用上面的正则表达式组成命令 adb logcat | grep "^..Activity"