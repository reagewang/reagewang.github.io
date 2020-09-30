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