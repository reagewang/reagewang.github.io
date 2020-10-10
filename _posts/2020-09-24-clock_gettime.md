```
函数"clock_gettime"是基于Linux C语言的时间函数,可以用于计算时间，有秒和纳秒两种精度。

函数原型：

int clock_gettime(clockid_t clk_id, struct timespec *tp);
其中，cld_id类型四种：   

a CLOCK_REALTIME:系统实时时间,随系统实时时间改变而改变
b CLOCK_MONOTONIC,从系统启动这一刻起开始计时,不受系统时间被用户改变的影响
c CLOCK_PROCESS_CPUTIME_ID,本进程到当前代码系统CPU花费的时间
d CLOCK_THREAD_CPUTIME_ID,本线程到当前代码系统CPU花费的时间

本文默认采用CLOCK_REALTIME，即可实现并行程序的准确计时。

其中，timespec结构包括：

struct timespec {
time_t tv_sec; /* 秒*/
long tv_nsec; /* 纳秒*/
};
```

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <time.h>

// GetDate - 获取当前系统日期
/**
 *  函数名称：GetDate
 *  功能描述：取当前系统日期
 *
 *  输出参数：char * psDate  - 系统日期，格式为yyymmdd
 *  返回结果：0 -> 成功
 */
int 
GetDate(char * psDate){
    time_t nSeconds;
    struct tm * pTM;
    
    time(&nSeconds); // 同 nSeconds = time(NULL);
    pTM = localtime(&nSeconds);
    
    /* 系统日期,格式:YYYMMDD */
    sprintf(psDate,"%04d-%02d-%02d", 
            pTM->tm_year + 1900, pTM->tm_mon + 1, pTM->tm_mday);
    
    return 0;
}

// GetTime  - 获取当前系统时间
/**
 *  函数名称：GetTime
 *  功能描述：取当前系统时间
 *
 *  输出参数：char * psTime -- 系统时间，格式为HHMMSS
 *  返回结果：0 -> 成功
 */
int 
GetTime(char * psTime) {
    time_t nSeconds;
    struct tm * pTM;
    
    time(&nSeconds);
    pTM = localtime(&nSeconds);
    
    /* 系统时间，格式: HHMMSS */
    sprintf(psTime, "%02d:%02d:%02d",
            pTM->tm_hour, pTM->tm_min, pTM->tm_sec);
           
    return 0;       
}

// GetDateTime - 取当前系统日期和时间
/**
 *  函数名称：GetDateTime
 *  功能描述：取当前系统日期和时间
 *
 *  输出参数：char * psDateTime -- 系统日期时间,格式为yyymmddHHMMSS
 *  返回结果：0 -> 成功
 */
int 
GetDateTime(char * psDateTime) {
    time_t nSeconds;
    struct tm * pTM;
    
    time(&nSeconds);
    pTM = localtime(&nSeconds);

    /* 系统日期和时间,格式: yyyymmddHHMMSS */
    sprintf(psDateTime, "%04d-%02d-%02d %02d:%02d:%02d",
            pTM->tm_year + 1900, pTM->tm_mon + 1, pTM->tm_mday,
            pTM->tm_hour, pTM->tm_min, pTM->tm_sec);
            
    return 0;
}
```

```
#include<stdio.h>  
#include<string.h>  
#include<errno.h>  
  
#include<unistd.h>  
  
#include<dirent.h>  
#include<sys/types.h>  
#include<sys/stat.h>  
  
#define MODE (S_IRWXU | S_IRWXG | S_IRWXO)  
  
int mk_dir(char *dir)  
{  
    DIR *mydir = NULL;  
    if((mydir= opendir(dir))==NULL)//判断目录   
    {  
      int ret = mkdir(dir, MODE);//创建目录  
      if (ret != 0)  
      {  
          return -1;  
      }  
      printf("%s created sucess!/n", dir);  
    }  
    else  
    {  
        printf("%s exist!/n", dir);  
    }  
  
    return 0;  
}  
```

```
	double diff = difftime(start,end);
    if(diff < 0) {
    	diff = 0 - diff;
	}
    int count_gap_day = (int)diff/(60*60*24);
```

```
time_t stringTotime_t(std::string str)
{
    struct tm tm_;
    int year, month, day, hour, minute,second;
    sscanf(str.c_str(),"%04d-%02d-%02d %02d:%02d:%02d", &year, &month, &day, &hour, &minute, &second);
    tm_.tm_year  = year-1900;
    tm_.tm_mon   = month-1;
    tm_.tm_mday  = day;
    tm_.tm_hour  = hour;
    tm_.tm_min   = minute;
    tm_.tm_sec   = second;
    tm_.tm_isdst = 0;
    time_t t_ = mktime(&tm_);
    return t_;
}
```

```
    std::vector<std::string> s = {"5", "7", "4", "2", "8", "6", "1", "9", "0", "3"};
    // sort using a lambda expression
    std::sort(s.begin(), s.end(), [](std::string a, std::string b) {
        return std::stoi(a) < std::stoi(b);
    });
    for (auto a : s) {
        std::cout << a << " ";
    }
    std::cout << '\n';
```

```
linux高精度struct timespec 和 struct timeval
一、struct timespec 定义：

typedef long time_t;
#ifndef _TIMESPEC
#define _TIMESPEC
struct timespec {
time_t tv_sec; // seconds 
long tv_nsec; // and nanoseconds 
};
#endif
struct timespec有两个成员，一个是秒，一个是纳秒, 所以最高精确度是纳秒。
一般由函数int clock_gettime(clockid_t, struct timespec *)获取特定时钟的时间，常用如下4种时钟：
CLOCK_REALTIME 统当前时间，从1970年1.1日算起
CLOCK_MONOTONIC 系统的启动时间，不能被设置
CLOCK_PROCESS_CPUTIME_ID 本进程运行时间
CLOCK_THREAD_CPUTIME_ID 本线程运行时间
struct tm *localtime(const time_t *clock);  //线程不安全
struct tm* localtime_r( const time_t* timer, struct tm* result );//线程安全
size_t strftime (char* ptr, size_t maxsize, const char* format,const struct tm* timeptr );

二、struct timeval 定义：

struct timeval {
time_t tv_sec; // seconds 
long tv_usec; // microseconds 
};
struct timezone{ 
int tz_minuteswest; //miniutes west of Greenwich 
int tz_dsttime; //type of DST correction 
};

struct timeval有两个成员，一个是秒，一个是微秒, 所以最高精确度是微秒。
一般由函数int gettimeofday(struct timeval *tv, struct timezone *tz)获取系统的时间 

#include<stdio.h>
#include<time.h>
#include<sys/time.h>

void nowtime_ns()
{
    printf("---------------------------struct timespec---------------------------------------\n");
    printf("[time(NULL)]     :     %ld\n", time(NULL));
    struct timespec ts;
    clock_gettime(CLOCK_REALTIME, &ts);
    printf("clock_gettime : tv_sec=%ld, tv_nsec=%ld\n", ts.tv_sec, ts.tv_nsec);

    struct tm t;
    char date_time[64];
    strftime(date_time, sizeof(date_time), "%Y-%m-%d %H:%M:%S", localtime_r(&ts.tv_sec, &t));
    printf("clock_gettime : date_time=%s, tv_nsec=%ld\n", date_time, ts.tv_nsec);
}
void nowtime_us()
{
    printf("---------------------------struct timeval----------------------------------------\n");
    printf("[time(NULL)]    :    %ld\n", time(NULL));
    struct timeval us;
    gettimeofday(&us,NULL);
    printf("gettimeofday: tv_sec=%ld, tv_usec=%ld\n", us.tv_sec, us.tv_usec);

    struct tm t;
    char date_time[64];
    strftime(date_time, sizeof(date_time), "%Y-%m-%d %H:%M:%S", localtime_r(&us.tv_sec, &t));
    printf("gettimeofday: date_time=%s, tv_usec=%ld\n", date_time, us.tv_usec);
}

int main(int argc, char* argv[])
{
    nowtime_ns();
    printf("\n");
    nowtime_us();
    printf("\n");
    return 0;
}

#include <time.h>

// 返回自系统开机以来的毫秒数（tick）
unsigned long GetTickCount()
{
    struct timespec ts;

    clock_gettime(CLOCK_MONOTONIC, &ts);

    return (ts.tv_sec * 1000 + ts.tv_nsec / 1000000);
}

int main()
{
    struct timespec time1 = { 0, 0 };

    clock_gettime(CLOCK_REALTIME, &time1);
    printf("CLOCK_REALTIME: %d, %d\n", time1.tv_sec, time1.tv_nsec);

    clock_gettime(CLOCK_MONOTONIC, &time1);
    printf("CLOCK_MONOTONIC: %d, %d\n", time1.tv_sec, time1.tv_nsec);

    clock_gettime(CLOCK_MONOTONIC_RAW, &time1);
    printf("CLOCK_MONOTONIC_RAW: %d, %d\n", time1.tv_sec, time1.tv_nsec);

    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &time1);
    printf("CLOCK_PROCESS_CPUTIME_ID: %d, %d\n", time1.tv_sec,
           time1.tv_nsec);

    clock_gettime(CLOCK_THREAD_CPUTIME_ID, &time1);
    printf("CLOCK_THREAD_CPUTIME_ID: %d, %d\n", time1.tv_sec,
           time1.tv_nsec);

    printf("\n%d\n", time(NULL));

    printf("tick count in ms: %ul\n", GetTickCount());

    return 0;
}
```