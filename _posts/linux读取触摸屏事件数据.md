# linux读取触摸屏事件数据

cat /proc/bus/input/devices

应该就能够看到触摸设备的相关信息

```shell
~ # cat /proc/bus/input/devices
I: Bus=0018 Vendor=0000 Product=0000 Version=0000
N: Name="gt9xx_ts"
P: Phys=
S: Sysfs=/devices/platform/soc/soc:amba/12119000.i2c/i2c-9/9-0014/input/input0
U: Uniq=
H: Handlers=event0
B: PROP=0
B: EV=b
B: KEY=0
B: ABS=660000000000000

I: Bus=0018 Vendor=0000 Product=0000 Version=0000
N: Name="gt9xx_ts"
P: Phys=
S: Sysfs=/devices/platform/soc/soc:amba/1211b000.i2c/i2c-11/11-0014/input/input1
U: Uniq=
H: Handlers=event1
B: PROP=0
B: EV=b
B: KEY=0
B: ABS=660000000000000

I: Bus=0003 Vendor=038f Product=0541 Version=0004
N: Name="USB 2.0 Camera"
P: Phys=usb-12300000.xhci_0-1/button
S: Sysfs=/devices/platform/soc/12300000.xhci_0/usb1/1-1/1-1:1.0/input/input2
U: Uniq=
H: Handlers=kbd event2
B: PROP=0
B: EV=3
B: KEY=100000 0 0 0

```

ABS表示触摸屏的绝对坐标掩码，掩码上面表示16进制
ABS=0x660 0000 0000 0000
=0110 0110 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000
其中为1的比特的位置就表示触摸屏会报告这一类型的事件，前面bit53,bit54,bit57,bit58为1，那么看linux/input.h文件就表示事件code码有
ABS_MT_POSITION_X=0x35=53  
ABS_MT_POSITION_Y=0x36=54
ABS_MT_TRACKING_ID=0x39=57
ABS_MT_PRESSURE=0x3a=58
