---
layout: post
title: "Android时间格式转换"
tags: [time,date,long,string]
comments: true
---

date类型转换为String类型
```java
// formatType格式为yyyy-MM-dd HH:mm:ss//yyyy年MM月dd日 HH时mm分ss秒
// data Date类型的时间
public static String dateToString(Date data, String formatType) {
    return new SimpleDateFormat(formatType).format(data);
}
```

long类型转换为String类型
```java
// currentTime要转换的long类型的时间
// formatType要转换的string类型的时间格式
public static String longToString(long currentTime, String formatType)
        throws ParseException {
    Date date = longToDate(currentTime, formatType); // long类型转成Date类型
    String strTime = dateToString(date, formatType); // date类型转成String
    return strTime;
}
```

string类型转换为date类型
```java
// strTime要转换的string类型的时间，formatType要转换的格式yyyy-MM-dd HH:mm:ss//yyyy年MM月dd日
// HH时mm分ss秒，
// strTime的时间格式必须要与formatType的时间格式相同
public static Date stringToDate(String strTime, String formatType)
        throws ParseException {
    SimpleDateFormat formatter = new SimpleDateFormat(formatType);
    Date date = null;
    date = formatter.parse(strTime);
    return date;
}

private Date StringToDate(String s){
    Date time=null;
    SimpleDateFormat sd=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    try {
        time=sd.parse(s);
    } catch (java.text.ParseException e) {
        System.out.println("输入的日期格式有误！"); 
        e.printStackTrace();
    }
    return time;
}
```

long转换为Date类型
```java
// currentTime要转换的long类型的时间
// formatType要转换的时间格式yyyy-MM-dd HH:mm:ss//yyyy年MM月dd日 HH时mm分ss秒
public static Date longToDate(long currentTime, String formatType)
        throws ParseException {
    Date dateOld = new Date(currentTime); // 根据long类型的毫秒数生命一个date类型的时间
    String sDateTime = dateToString(dateOld, formatType); // 把date类型的时间转换为string
    Date date = stringToDate(sDateTime, formatType); // 把String类型转换为Date类型
    return date;
}
```

String类型转换为long类型
```java
// strTime要转换的String类型的时间
// formatType时间格式
// strTime的时间格式和formatType的时间格式必须相同
public static long stringToLong(String strTime, String formatType)
        throws ParseException {
    Date date = stringToDate(strTime, formatType); // String类型转成date类型
    if (date == null) {
        return 0;
    } else {
        long currentTime = dateToLong(date); // date类型转成long类型
        return currentTime;
    }
}
```

date类型转换为long类型
```java
// date要转换的date类型的时间
public static long dateToLong(Date date) {
    return date.getTime();
}

private long DateToLong(Date time){
    SimpleDateFormat st=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//yyyyMMddHHmmss
    return Long.parseLong(st.format(time));
}
```

将毫秒转换为小时：分钟：秒格式
```java
public static String ms2HMS(int _ms){
    String HMStime;
    _ms/=1000;
    int hour=_ms/3600;
    int mint=(_ms%3600)/60;
    int sed=_ms%60;
    String hourStr=String.valueOf(hour);
    if(hour<10){
        hourStr="0"+hourStr;
    }
    String mintStr=String.valueOf(mint);
    if(mint<10){
        mintStr="0"+mintStr;
    }
    String sedStr=String.valueOf(sed);
    if(sed<10){
        sedStr="0"+sedStr;
    }
    HMStime=hourStr+":"+mintStr+":"+sedStr;
    return HMStime;
}
```

将毫秒转换为标准日期格式
```java
public static String ms2Date(long _ms){
    Date date = new Date(_ms);
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());
    return format.format(date);
}

public static String ms2DateOnlyDay(long _ms){
    Date date = new Date(_ms);
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault());
    return format.format(date);
}
```

标准时间转换为时间戳
```java
public static long Date2ms(String _data){
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    try {
        Date date = format.parse(_data);
        return date.getTime();
    }catch(Exception e){
        return 0;
    }
}
```

计算时间差
```java
public static  String DateDistance(Date startDate,Date endDate){
    if(startDate == null ||endDate == null){
        return null;
    }
    long timeLong = endDate.getTime() - startDate.getTime();
    if(timeLong<0){
        timeLong=0;
    }
    if (timeLong<60*1000)
        return timeLong/1000 + "秒前";
    else if (timeLong<60*60*1000){
        timeLong = timeLong/1000 /60;
        return timeLong + "分钟前";
    }
    else if (timeLong<60*60*24*1000){
        timeLong = timeLong/60/60/1000;
        return timeLong+"小时前";
    }
    else if ((timeLong/1000/60/60/24)<7){
        timeLong = timeLong/1000/ 60 / 60 / 24;
        return timeLong + "天前";
    }else{
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
        return formatter.format(startDate);
    }
}
```

计算与当前的时间差
```java
public static  String DateDistance2now(long _ms){
    SimpleDateFormat DateF = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    try {
        Long time=new Long(_ms);
        String d = DateF.format(time);
        Date startDate=DateF.parse(d);
        Date nowDate = Calendar.getInstance().getTime();
        return DateDistance(startDate, nowDate);
    }catch (Exception e){
    }
    return null;
}
```

日期比较
```java
public void compareToNowDate(Date date){
    Date nowdate=new Date();
    //format date pattern
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    //convert to millions seconds
    long time=DateToLong(StringToDate(formatter.format(nowdate)));
    long serverTime=DateToLong(date);
    //convert to seconds
    long minTime=Math.abs(time-serverTime)/1000;	Log.d(getLocalClassName(), "minTime= "+minTime);
}
```

计算日期之间相隔几天
```java
public long compareDataToNow(String date){
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");  
    Date passDate,nowDate;
    long diff=-100l,days=-100l;
    try {
        passDate = sdf.parse(date);
        String nowStr=sdf.format(new Date());
        nowDate=sdf.parse(nowStr);  
        
        diff = passDate.getTime() - nowDate.getTime();   
        days = diff / (1000 * 60 * 60 * 24); 
    } catch (ParseException e) {
        e.printStackTrace();
    }
    return diff;
}
```

Android中获取系统时间有多种方法
```java
void getTime1(){
    //long now = android.os.SystemClock.uptimeMillis();
    long time = System.currentTimeMillis();
    SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date d1=new Date(time);
    String t1=format.format(d1);
    Log.e("msg", t1);
}

void getTime2(){
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd-HH:mm:ss");
    String t=format.format(new Date());
    Log.e("msg", t);
}

void getTime3(){
    Calendar calendar = Calendar.getInstance();
    String created = calendar.get(Calendar.YEAR) + "年"
            + (calendar.get(Calendar.MONTH)+1) + "月"//从0计算
            + calendar.get(Calendar.DAY_OF_MONTH) + "日"
            + calendar.get(Calendar.HOUR_OF_DAY) + "时"
            + calendar.get(Calendar.MINUTE) + "分"+calendar.get(Calendar.SECOND)+"s";
    Log.e("msg", created);
}

void getTime4(){
    Time t=new Time(); // or Time t=new Time("GMT+8"); 加上Time Zone资料。
    t.setToNow(); // 取得系统时间
    String time = t.year + "年 " + (t.month+1) + "月 " + t.monthDay + "日 " + t.hour + "h " + t.minute + "m " + t.second;
    Log.e("msg", time);
}
```

获取星期日期
```java
void getWeek(){
    Calendar calendar = Calendar.getInstance();
    int day = calendar.get(Calendar.DAY_OF_WEEK);
    String today = null;
    if (day == 2) {
        today = "Monday";
    } else if (day == 3) {
        today = "Tuesday";
    } else if (day == 4) {
        today = "Wednesday";
    } else if (day == 5) {
        today = "Thursday";
    } else if (day == 6) {
        today = "Friday";
    } else if (day == 7) {
        today = "Saturday";
    } else if (day == 1) {
        today = "Sunday";
    }
    System.out.println("Today is:- " + today);
}
```