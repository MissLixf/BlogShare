### 概念

中断是系统用来响应硬件设备请求的一种机制，它会打断进程的正常调度和执行，然后调用内核中的中断处理程序来响应设备的请求。以点外卖为例，点好后等送到的电话（中断）就行了，在此期间，你可以做其他的事情。

可以说，中断是一种异步的事件处理机制，可以提高系统的并发处理能力。

为了减少中断对正常进程的影响，中断处理程序必须尽可能快地运行。而且，中断处理程序在响应中断时，会临时关闭中断（中断禁止模式），导致新的中断无法被响应（类似于电话占线）。为了解决这两个问题，Linux将中断处理程序分成了两部分：

* 上半部（硬件中断）：快速处理中断，在中断禁止模式下运行，主要处理跟硬件紧密先关或时间敏感的工作（比如，键盘鼠标输入、磁盘读取、网卡收发等）
* 下半部（软中断）：异步处理上半部未完成的工作，通常以内核线程的方式运行。（主要包括：网络收发、定时、调度、RCU锁等）

下面以网卡接收数据包为例进行说明。

* 上半部：网卡接收到数据包后，通过硬件中断，通知内核有新的数据到了，将数据从网卡读入内存，更新下硬件寄存器状态(表示数据已经读好了)，然后发送软中断信号，通知下半部做进一步处理。
* 下半部：被软中断信号唤醒后，从内存找到网络数据，根据网络协议解析处理，发送给应用程序。

### 查看中断和内核线程

通过proc文件系统可以查看

* /proc/softirqs  查看软中断

  ```shell
  [root@iz2zeap40j01vg100ifsf4z ~]# cat /proc/softirqs
                      CPU0       CPU1
            HI:          1          1
         TIMER:    6315960    6406790
        NET_TX:          7          2
        NET_RX:    5811737    5761743
         BLOCK:      31495          0
  BLOCK_IOPOLL:          0          0
       TASKLET:         27          5
         SCHED:    2891351    2974106
       HRTIMER:          0          0
           RCU:    3569212    3581708
  ```

* /proc/interrupts  查看硬中断

  ```shell
  [root@iz2zeap40j01vg100ifsf4z ~]# cat /proc/interrupts
             CPU0       CPU1
    0:         25          0   IO-APIC-edge      timer
    1:         10          0   IO-APIC-edge      i8042
    6:          3          0   IO-APIC-edge      floppy
    8:          0          0   IO-APIC-edge      rtc0
    9:          0          0   IO-APIC-fasteoi   acpi
   10:          0          0   IO-APIC-fasteoi   virtio4
   11:         34          0   IO-APIC-fasteoi   uhci_hcd:usb1
   12:         15          0   IO-APIC-edge      i8042
   14:          0          0   IO-APIC-edge      ata_piix
   15:      67142          0   IO-APIC-edge      ata_piix
   24:          0          0   PCI-MSI-edge      virtio2-config
   25:      31883      67877   PCI-MSI-edge      virtio2-req.0
   26:          0          0   PCI-MSI-edge      virtio3-config
   27:        170          0   PCI-MSI-edge      virtio3-req.0
   28:          0          0   PCI-MSI-edge      virtio1-config
   29:          3          0   PCI-MSI-edge      virtio1-virtqueues
   30:          0          0   PCI-MSI-edge      virtio0-config
   31:          1     111167   PCI-MSI-edge      virtio0-input.0
   32:      16900     132582   PCI-MSI-edge      virtio0-output.0
   33:     137937      13965   PCI-MSI-edge      virtio0-input.1
   34:       1331     146336   PCI-MSI-edge      virtio0-output.1
  NMI:          0          0   Non-maskable interrupts
  LOC:   35228300   35407955   Local timer interrupts
  SPU:          0          0   Spurious interrupts
  PMI:          0          0   Performance monitoring interrupts
  IWI:     796912     786762   IRQ work interrupts
  RTR:          0          0   APIC ICR read retries
  RES:    3191412    3243228   Rescheduling interrupts
  CAL:      62023      67611   Function call interrupts
  TLB:      60923      59420   TLB shootdowns
  TRM:          0          0   Thermal event interrupts
  THR:          0          0   Threshold APIC interrupts
  DFR:          0          0   Deferred Error APIC interrupts
  MCE:          0          0   Machine check exceptions
  MCP:        229        229   Machine check polls
  ERR:          0
  MIS:          0
  PIN:          0          0   Posted-interrupt notification event
  PIW:          0          0   Posted-interrupt wakeup event
  ```

*  以上命令前加上`watch -d `即可动态查看数值的变化，比如：`watch -d cat /proc/softirqs`

软中断以内核线程的方式运行，每个CPU都有一个软中断的内核线程，名为`ksoftirqd/CPU编号`，可以通过ps命令查看：

```shell
[root@iz2zeap40j01vg100ifsf4z ~]# ps aux | grep ksoftirq
root         3  0.0  0.0      0     0 ?        S    Jun05   0:00 [ksoftirqd/0]
root        13  0.0  0.0      0     0 ?        S    Jun05   0:00 [ksoftirqd/1]
```



### 软中断与性能

软中断频率过高时（比如洪水攻击），频繁调用软中断内核线程，从而导致中断处理不及时，进而引发网络收发延迟、调度缓慢等性能问题。

注意，软中断不一定占用太多CPU资源，但是可能造成网络收发延迟，甚至是丢包，如果发生这种情况，通过终端远程登录操作就会很卡顿。





