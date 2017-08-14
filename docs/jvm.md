# jvm 内存分析

## jmap
通过jmap -heap pid,查看到的为JAVA堆内存(Young Gen+Old Gen)+Perm Gen

## jstat

* jstat命令命令格式：
> jstat [Options] vmid [interval] [count]

* 参数说明
```
Options，选项，我们一般使用 -gcutil 查看gc情况
vmid，VM的进程号，即当前运行的java进程号
interval，间隔时间，单位为秒或者毫秒
count，打印次数，如果缺省则打印无数次
```

## 示例
jstat -gc 12538 5000

即会每5秒一次显示进程号为12538的java进成的GC情况，
![](images/gc.md)
显示内容说明如下（部分结果是通过其他其他参数显示的，暂不说明）：
```
S0C：年轻代中第一个survivor（幸存区）的容量 (字节) 
         S1C：年轻代中第二个survivor（幸存区）的容量 (字节) 
         S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节) 
         S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节) 
         EC：年轻代中Eden（伊甸园）的容量 (字节) 
         EU：年轻代中Eden（伊甸园）目前已使用空间 (字节) 
         OC：Old代的容量 (字节) 
         OU：Old代目前已使用空间 (字节) 
         PC：Perm(持久代)的容量 (字节) 
         PU：Perm(持久代)目前已使用空间 (字节) 
         YGC：从应用程序启动到采样时年轻代中gc次数 
         YGCT：从应用程序启动到采样时年轻代中gc所用时间(s) 
         FGC：从应用程序启动到采样时old代(全gc)gc次数 
         FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s) 
         GCT：从应用程序启动到采样时gc用的总时间(s) 
         NGCMN：年轻代(young)中初始化(最小)的大小 (字节) 
         NGCMX：年轻代(young)的最大容量 (字节) 
         NGC：年轻代(young)中当前的容量 (字节) 
         OGCMN：old代中初始化(最小)的大小 (字节) 
         OGCMX：old代的最大容量 (字节) 
         OGC：old代当前新生成的容量 (字节) 
         PGCMN：perm代中初始化(最小)的大小 (字节) 
         PGCMX：perm代的最大容量 (字节)   
         PGC：perm代当前新生成的容量 (字节) 
         S0：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比 
         S1：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比 
         E：年轻代中Eden（伊甸园）已使用的占当前容量百分比 
         O：old代已使用的占当前容量百分比 
         P：perm代已使用的占当前容量百分比 
         S0CMX：年轻代中第一个survivor（幸存区）的最大容量 (字节) 
         S1CMX ：年轻代中第二个survivor（幸存区）的最大容量 (字节) 
         ECMX：年轻代中Eden（伊甸园）的最大容量 (字节) 
         DSS：当前需要survivor（幸存区）的容量 (字节)（Eden区已满） 
         TT： 持有次数限制 
         MTT ： 最大持有次数限制 
```

## 查看jvm进程中线程占用
* jps（查看java进程）
* top -Hp xxx(查看进程中线程的占用情况)
* printf "%x\n" xxx （查看16进制）
* jstack xxx |grep xxx（查看线程情况）
