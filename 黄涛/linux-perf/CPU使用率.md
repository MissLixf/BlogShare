# CPU使用率

## 概念

将CPU的时间划分为时间片，轮流执行任务，就达到了多任务同时运行的效果。CPU时间通过节拍率和节拍数来维护。节拍率单位是HZ，分为内核节拍率和用户节拍率，其中用户节拍率的值是固定的100（即1/100秒，10ms），每秒钟触发100次时间中断，每发生一次时间中断，节拍数加1

通过/proc虚拟文件系统，用户可以查看系统内部状态的信息，其中/proc/stat，可以查看系统CPU和任务统计信息，下面我们看一下CPU的：

```bash
$ cat /proc/stat | grep ^cpu

cpu  29246472 436 7061293 1189010031 844048 0 460919 0 0 0
cpu0 14567277 219 3544818 594761835 380172 0 228573 0 0 0
cpu1 14679194 217 3516475 594248196 463875 0 232346 0 0 0
```

其中，第一行表示总体情况。二三行分别是单个CPU的情况。从第2列到第11列表示不同场景下CPU的累计节拍数。

第一列表示CPU，剩余各列含义如下：

* user (us) 代表用户态CPU时间，不包括nice时间
* nice (ni) 好心值，表示低优先级的用户态CPU时间，可取范围是-20~19，数值越大，优先级越低
* system (sys)，内核态CPU时间
* idle (id)，空闲时间，不包括I/O时间
* iowait (wa) 等待I/O的CPU时间
* irq (hi) 处理硬件中断的CPU时间
* softirq (si) 处理软件中断的CPU时间
* steal (st) 偷走的时间，运行虚拟机时被占用的CPU时间
* guest (guest) 运行虚拟机的CPU时间
* guest_nice (gnice) 低优先级运行虚拟机的时间

通常所说的CPU使用率，就是CPU非空闲时间占总时间的百分比即：`(CPU总时间 - CPU空闲时间) / CPU总时间`，各种性能工具基本也是这么计算的，不过它们取的是一段时间的平均值。比如，top命令采用每3秒间隔的平均值，而ps采用进程的整个生命周期。

## 如何查看CPU使用率

常用的命令：

* top  显示系统整体的CPU和内存使用情况，以及各个进程的资源使用情况
  * top 结果默认显示所有CPU的平均值，按1，即可分别显示各CPU的值
* ps 只显示每个进程的资源使用情况

```bash
top - 18:39:24 up 74 days, 55 min,  6 users,  load average: 0.01, 0.24, 0.32
Tasks: 166 total,   2 running, 164 sleeping,   0 stopped,   0 zombie
%Cpu0  :  1.3 us,  0.7 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  1.0 us,  0.3 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  3881688 total,   376992 free,  3028316 used,   476380 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   592924 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 7145 systemd+  20   0   74048  17504    616 S   0.7  0.5  76:41.16 redis-server
 1286 root      20   0 2516200  70764   2280 S   0.3  1.8 506:00.57 java
18684 root      20   0  436544  94692   1380 S   0.3  2.4   0:06.54 celery
...
```

top命令的%CPU列，是用户态CPU和内核态CPU使用的总和，较为笼统。如果要查看每个进程的详细信息，使用pidstat即可：

```bash
$ pidstat 1 2
# 间隔1秒输出2组CPU使用率
Linux 3.10.0-693.11.1.el7.x86_64 (iz2zeap40j01vg100ifsf4z)      01/15/2019      _x86_64_    (2 CPU)

06:44:29 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
06:44:30 PM     0      3871    0.99    0.00    0.00    0.99     0  AliYunDun
06:44:30 PM     0     19097    0.99    0.00    0.00    0.99     1  celery
06:44:30 PM     0     19099    0.99    0.00    0.00    0.99     1  celery
06:44:30 PM     0     19437    0.00    0.99    0.00    0.99     1  pidstat

06:44:30 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
06:44:31 PM     0      3871    0.00    1.00    0.00    1.00     0  AliYunDun
06:44:31 PM     0      7125    1.00    0.00    0.00    1.00     1  docker-proxy
06:44:31 PM     0     26056    0.00    1.00    0.00    1.00     0  kworker/0:2
...
Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0      3871    0.50    0.50    0.00    1.00     -  AliYunDun
Average:        0      7125    0.50    0.00    0.00    0.50     -  docker-proxy
Average:        0     18681    0.50    0.00    0.00    0.50     -  celery
Average:        0     19437    0.00    0.50    0.00    0.50     -  pidstat
Average:        0     26056    0.00    0.50    0.00    0.50     -  kworker/0:2
```

* %usr  用户态CPU使用率
* %system  内核态...
* %guest  运行虚拟机...
* %CPU 总使用率
* 最后Average，计算两组数据的平均值

## perf 性能分析工具

通过以上工具找到CPU使用率较高的进程后，如何进一步定位是哪个函数造成的呢？

perf是linux内置的性能分析工具（如果没有，可以 yum 手动安装），下面我们看看如何用它解决上面的问题。

perf 常用的3种用法：

* perf top  系统画像工具，用来实时显示占用CPU时钟最多的函数或指令
* perf record  采样性能数据并保存
* perf report  解析perf record保存的数据

### perf  top

```bash
$ perf top
# 具体含义可以 man perf-top 查看
Samples: 260K of event 'cpu-clock', Event count (approx.): 1117764011

Overhead  Shared Object                 Symbol
  14.82%  [kernel]                      [k] _raw_spin_unlock_irqrestore
   5.45%  [kernel]                      [k] run_timer_softirq
   4.98%  [kernel]                      [k] finish_task_switch
   2.49%  [kernel]                      [k] __do_softirq
   1.21%  [kernel]                      [k] tick_nohz_idle_exit
   1.17%  perf                          [.] rb_next
   1.01%  [kernel]                      [k] sk_run_filter
   0.95%  [kernel]                      [k] rcu_process_callbacks
   0.94%  python2.7                     [.] 0x00000000001614bb
```

第一行有3个参数：

* samples 采样数
* event  事件类型
* event count 时间总数

这里的情况是，perf采集了260千个的CPU时钟时间，而总事件数是1326083091

下面具体列的含义：

* overhead 占用比例
* shared object  函数/指定所在的【动态共享对象】，比如内核，进程名，动态链接库名，内核模块等
* object 动态共享对象的类型：
  * [.] 用户空间的可执行程序或动态链接库
  * [k] 内核空间
* symbol 函数名（未知的用16进制地址表示）

### perf record & perf report

perf top 展示的系统实时性能信息，但是数据无法保存，也就不能用于离线或后续分析。perf record 提供了保存采样数据，然后使用perf report自动解析数据并展示：

```bash
$ perf record
# perf record 开始采样，ctrl + c 终止采样
^C[ perf record: Woken up 26 times to write data ]
[ perf record: Captured and wrote 7.034 MB perf.data (140390 samples) ]

$ perf report
# 展示形式同perf top
```

## 压测分析

这里我们使用ab （apache bench），来模拟客户端的HTTP请求。

服务器A运行web应用，服务器B使用ab进行模拟请求

```bash
$ ab -c 100 -n 1000 http://api.server_a.com/home/all/
# -c 100 并发100个请求  -n 1000 总共1000个请求

Benchmarking api.server_a.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        nginx/1.12.2
Server Hostname:        api.server_a.com
Server Port:            80

Document Path:          /home/all/
Document Length:        4988 bytes

Concurrency Level:      100
Time taken for tests:   8.260 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      5387293 bytes
HTML transferred:       4988000 bytes
Requests per second:    121.06 [#/sec] (mean)
# 每秒可以承受的平均请求
Time per request:       826.014 [ms] (mean)
Time per request:       8.260 [ms] (mean, across all concurrent requests)
# 平均每个请求的响应时间
Transfer rate:          636.92 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        2    3   3.8      3     122
Processing:    82  778 158.6    784    1473
Waiting:       80  778 158.6    784    1473
Total:         85  781 158.6    787    1476

Percentage of the requests served within a certain time (ms)
  50%    787
  66%    819
  75%    846
  80%    868
  90%    939
  95%    989
  98%   1062
  99%   1109
 100%   1476 (longest request)

```

运行以上测试期间，通过top查看服务器A的性能情况：

```bash
top - 16:21:45 up 74 days, 22:38,  4 users,  load average: 1.23, 0.50, 0.37
Tasks: 142 total,   8 running, 127 sleeping,   7 stopped,   0 zombie
%Cpu0  : 94.6 us,  3.0 sy,  0.0 ni,  0.7 id,  0.0 wa,  0.0 hi,  1.7 si,  0.0 st
%Cpu1  : 94.0 us,  2.3 sy,  0.0 ni,  1.0 id,  0.0 wa,  0.0 hi,  2.7 si,  0.0 st
KiB Mem :  3881688 total,   284400 free,  3304276 used,   293012 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   320356 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
11684 root      20   0  369276 104768   3656 S  20.3  2.7   0:12.24 uwsgi
11600 root      20   0  516668 107452   7028 S  19.3  2.8   0:22.65 uwsgi
11674 root      20   0  368764 103192   3616 R  19.3  2.7   0:10.44 uwsgi
11682 root      20   0  368228 102976   3648 R  19.3  2.7   0:11.95 uwsgi
11576 root      20   0   63224  18788   3696 S   2.7  0.5   0:12.14 supervisord

```

可以观察到两个CPU的使用率直线上升，接近饱和状态。

利用perf进行调用链分析，可以找到导致CPU使用率升高的函数：

```bash
perf top -g -p 1651
# -g 开启调用关系分析；-p 指定进程号
```

注意，如果是应用运行在docker容器中，以上输出结果无法显示具体函数名，而是一串16进制的字符。因此测试不建议在docker中进行。

#### 遇到的问题

* ab -c 1000 -n 10000 时，报错：apr_pollset_poll: The timeout specified has expired (70007)，无法完成所有请求。解决方案： 

  ```ini
  并发1K， 总共1W，超时120s, -k 保持连接keep-alive -r socket接收错误后，不退出测试
  ab -c 1000 -n 10000 -s 120 -k -r wwww.xxx.com
  ```

* 

## 短时进程

有时候CPU使用率很高，但是又找不到具体是哪个进程导致的。原因可能如下：

* 进程不停的崩溃重启（可能是配置错误的原因）
* 短时进程：应用内执行的命令调用，并且这些命令只运行很短的时间。

对于短时进程，常用的性能检测工具很难有效监测（在top中可能出现一下（状态是R），又消失了，又突然出现）。偶尔发现后，可以通过pstree 找到调用短时进程的父进程，进而排查问题：

```bash
pstree | grep 短时进程名
```

常用的工具难以监测短时进程，除了perf 工具外，`execsnoop`（https://github.com/brendangregg/perf-tools/blob/master/execsnoop）是专门监测短时进程的工具：

```bash
execsnoop
# 要结束监测，ctl + c
```

通过execsnoop找到父进程后，从父进程所在的应用入手，进而排查问题。


