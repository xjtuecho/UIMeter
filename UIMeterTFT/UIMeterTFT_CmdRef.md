# UIMeterTFT Command Line Reference

UIMeterTFT内置一个命令解释器，可以通过超级终端（或者Putty、SecrueCRT）等软件来连接。

串口参数：`波特率115200、8位数据、1位停止、无校验、无流控`。

本文档基于UIMeterTFT固件v18.8.30，其余固件版本仅供参考。

# 更新历史

## v18.8.30 

增加高分辨率版本支持。

## v18.8.9 

初始发布。

## getui

获取当前电压、电流、时间、功率、电量等信息。

命令格式：`getui` 

```
getui
 U:   5.157V 0.1459W AD=0x317A
 I: -0.0283A 182.25R PGA=8 AD=0xFFFF52   -340uV
 P:-0.0044Ah -0.0230Wh    569s
 Vd+:0.252V AD=0x147F  Vdd:3.287V AD=0x5CE7
 Vd-:0.256V AD=0x1463   Tj:  32oC AD=0x6C7B
```

第三方软件可以通过执行该命令查询设备实时数据，可以关闭命令回显，降低第三方软件编写难度。

## clear

清除设备当前时间和电量信息。

命令格式：`clear`

设备上电以后，运行时间和Ah电量、Wh电量会一直累计，执行该命令以后，运行时间、Ah电量、Wh电量全部归零。
其余数据为实时更新，该命令无影响。

**可在测试数据界面下长按左键清除当前时间和电量信息**。

## log

操作离线数据。

命令格式：`log [dump|max|int|ring|auto|uh|ul|ih|il] Operate data logs.`

不带参数的log命令输出当前设置，如下所示，依次显示数据最大记录条数、记录间隔、
RING模式开关、AUTO模式开关，UH、UL、IH、IL四个参数。
```
log
log [dump|max|int|ring|auto|uh|ul|ih|il] Operate data logs.
 log data length is  4096
 log interval is   1
 ring mode is Off
 auto start log mode is Off
 UH= 0.0000V UL= 0.0000V
 IH= 0.0000A IL= 0.0000A
```
具体意义见以下子命令说明。

### log dump

log dump子命令用来导出离线数据。

命令格式：`log dump [数据条数]`.

数据条数应该小于设备最大记录条数。导出10条数据如下所示：
```
log dump 10
    i,    t(s),    U(V),    I(A),   Vd+,   Vd-
    0,      15,  4.9812,  0.0000, 0.017, 0.018
    1,      16,  4.9731,  0.0000, 0.017, 0.017
    2,      17,  4.9731,  0.0000, 0.017, 0.017
    3,      18,  4.9731,  0.0000, 0.017, 0.018
    4,      19,  4.9731,  0.0000, 0.018, 0.018
    5,      20,  4.9731,  0.0000, 0.017, 0.018
    6,      21,  4.9731,  0.0000, 0.017, 0.018
    7,      22,  4.9812,  0.0000, 0.017, 0.017
    8,      23,  4.9731,  0.0000, 0.017, 0.017
    9,      24,  4.9731,  0.0000, 0.018, 0.018
```
第一列为数据索引，第二列为设备记录数据时的相对时间（单位秒），第三列为电压，
第四列为电流，第五列为USB D+电压，第六列为USB D-电压。
 
可以借助超级终端捕获文字功能保存离线数据。菜单：传送(T)->捕获文字(C)…打开
超级终端的捕获文字功能，启动捕获文字，执行完log dump命令，然后停止。保存离线记录数据。

离线数据格式为CSV文件，可以使用Excel或者任意一款文本编辑器编辑。
也可以用UIMeterMon软件来分析处理。

### log max

设置最大记录条数。

最大记录条数需要和实际设备配置的EEPROM容量匹配，32kB对应2048条，64kB对应4096条。

设置最大记录条数为2048：log max 2

设置最大记录条数为4096：log max 4
```
log max 2
 Set Max data log to  2048
log max 4
 Set Max data log to  4096
```
设置以后立即生效，保存参数需要执行`param save`命令。

**UIMeterTFT支持4096条离线记录数据，无需使用该命令设置**。

### log int

设置离线记录间隔，单位秒，如：`log int 10`，设置离线记录间隔为10秒每次。

设置为0时以最快3Hz记录数据，最大间隔65535秒。

设置以后立即生效，保存参数需要执行`param save`命令。

### log ring

打开或者关闭RING模式（循环记录）。

默认模式下，记录数据达到最大以后停止记录，通过打开RING模式，可以使数据达到最大
以后自动从0开始记录，覆盖旧数据。该方法可以用于循环记录，跟踪最新的测试数据。

使用`log ring 1`命令打开RING模式。

使用`log ring 0`命令关闭RING模式。

设置以后立即生效，保存参数需要执行`param save`命令。

### log auto

打开或者关闭AUTO模式（自动记录）。

默认情况下，数据离线记录功能需要用户来手动启动。如果用户需要同步采集某些量，
多台设备之间很难完成同步启动采集，此时可以打开AUTO模式，设备上电以后自动开始
记录数据。通过给多台设备同时上电来完成数据同步采集。

使用`log auto 1`命令打开自动记录功能。

使用`log auto 0`命令关闭自动记录功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### log [uh|ul|ih|il]

设定UH、UL、IH、IL四个参数。
四个参数决定离线记录的条件。UH和UL设定电压上限和下限，IH和IL设定电流上限和下限，

记录规则如下：
  1. UH>UL。电压上限高于电压下限，实际电压高于下限并且低于上限时记录数据。
  2. UH<UL。电压上限低于电压下限，实际电压高于下限或者低于上限时记录数据。
  3. UH=UL。电压上限等于电压下限，离线记录数据和电压无关，只与电流有关。
  4. IH>IL。电流上限高于电流下限，实际电流高于下限并且低于上限时记录数据。
  5. IH<IL。电流上限低于电流下限，实际电流高于下限或者低于上限时记录数据。
  6. IH=IL。电流上限等于电流下限，离线记录数据和电压电流均无关，即全部记录。

注意，UH、UL、IH、IL为在原来手动记录基础上增加的四个条件，设置以后仍然需要手动
启停记录，如果不使用需要全部设置为0.

设置命令举例如下：

```
log uh 0
 set UH= 0.0000V
log ul 0
 set UL= 0.0000V
log ih 20000
 set IH= 2.0000A
log il 10000
 set IL= 1.0000A
log
log [dump|max|int|ring|auto|uh|ul|ih|il] Operate data logs.
 log data length is  4096
 log interval is   1
 ring mode is Off
 auto start log mode is Off
 UH= 0.0000V UL= 0.0000V
 IH= 2.0000A IL= 1.0000A
```

电流大于1A小于2A时记录离线数据：UH=UL=0V，IH=2A，IL=1A。

电压高于10V时记录离线数据：UH=0V，UL=10V，IH=0A，IL=0A。

设置以后立即生效，保存参数需要执行`param save`命令。

## param

操作用户参数。

命令格式：`param [load|save|restore] Operate parameters.`

param命令带三个子命令：load、save、restore。

`param load`命令从内置EEPROM加载保存的参数。

`param save`命令将参数保存到内置EEPROM。

`param restore`命令恢复默认参数。

## uset

电压通道参数设置。

命令格式：`uset [adj|zero|cali] [adj 100000x|U 10000x] set U param.`

不带参数的uset命令输出当前电压通道和电流通道的所有参数。如下所示：

```
uset
uset [adj|zero|cali] [adj 100000x|U 10000x] set U param.
 U Adj:  1.00652   U Zero:       0
 I Adj:  1.00179   I Zero:       0
```

### uset adj

设置电压增益校正系数。

电压增益校正系数是一个1附近的数值，用来校正分压电阻、基准初始值等带来的误差。
范围0-100，如果UIMeterTFT显示电压数值小于实际电压值，需要增大电压增益校正
系数，反之减小电压增益校正系数。

使用uset adj命令将电压增益校正系数设置为1.00234，命令如下：
```
uset adj 100234
uset
uset [adj|zero|cali] [adj 100000x|U 10000x] set U param.
 U Adj:  1.00234   U Zero:       0
 I Adj:  1.00179   I Zero:       0
``` 
设定数值需要扩大100000倍去掉小数点。

### uset zero

设置电压零偏校正系数。

如果电压通道在短接测量端时不为0，需要校正电压零偏。单位为一个电压分辨率。

UIMeterTFT不支持零偏校正，该命令保留。

### uset cali

电压快速校准。

首先使用`uset adj 100000`命令将电压增益系数设置为1.

将UIMeterTFT电压通道与基准电压源并联，读取基准源电压值，执行命令`uset cali [基准电压值]`
UIMeterTFT自动计算校准系数，保证电压显示值与基准电压值相等。基准电压值需要扩大10000倍去掉小数。

设置以后立即生效，保存参数需要执行`param save`命令。

## iset

电流通道参数设置。

命令格式：`iset [adj|zero|cali] [adj 100000x|I 10000x] set I param.`

不带参数的iset命令输出当前电压通道和电流通道的所有参数。如下所示：
```
iset
iset [adj|zero|cali] [adj 100000x|I 10000x] set I param.
 U Adj:  1.00652   U Zero:       0
 I Adj:  1.00179   I Zero:       0
```

### iset adj

设置电流增益校正系数。

电流增益校正系数是一个1附近的数值，用来校正检流电阻、基准初始值等带来的误差，
范围0-100，如果UIMeterTFT显示电流数值小于实际电流值，需要增大电流增益校正
系数，反之减小电流增益校正系数。

使用iset adj命令将电流增益校正系数设置为1.00234，命令如下：
```
iset adj 100234
iset
iset [adj|zero|cali] [adj 100000x|I 10000x] set I param.
 U Adj:  1.01209   U Zero:       0
 I Adj:  1.00234   I Zero:       0
```
设定数值需要扩大100000倍去掉小数点。

### iset zero

设置电流零偏校正系数。

如果电流通道在测量端悬空时不为0，需要校正电流零偏。单位为一个电流分辨率。

如果电流显示0.0003A，使用`iset zero 3`命令校正。

如果电流显示-0.0002A，使用`iset zero -2`命令校正。

### iset cali

电流快速校准。

首先使用`iset adj 100000`命令将电流增益校正系数设置为1.

将UIMeter电流通道与基准电流源串联，读取基准源电流值，执行命令：`iset cali [基准电流值]`
UIMeterTFT自动计算校准系数，保证电流显示值与基准电流值相等。基准电流值需要扩大10000倍去掉小数。

设置以后立即生效，保存参数需要执行`param save`命令。

## tset

温度校正设置。

命令格式：`tset [v30|cali] [V30|T] set T param.`

UIMeterTFT通过MCU内部温度传感器测量温度，通过设置30℃对应的温度传感器输出电压，
可校正内部温度传感器输出，获得准确的温度测量值。

### tset v30

直接设置传感器校正电压。

如将传感器输出电压设置为1430mV，执行以下命令。

```
tset v30 1430
```

### tset cali

温度快速校准。

将UIMeterTFT放置在已知温度的环境中，执行以下命令，校正温度传感器。
```
tset cali [当前环境温度]
```

设备自动计算出校正电压。

设置以后立即生效，保存参数需要执行`param save`命令。

## ctrl

设备控制命令。

命令格式：`ctrl [echo|bklt|time|dir|stby|iwake|dev] [param] Device Control.`

控制或者显示设备的运行参数，不带参数的ctrl命令显示当前设置，如下所示：
```
ctrl
ctrl [echo|bklt|time|dir|stby|iwake|dev] [param] Device Control.
 ECHO=1 BKLT=0xF880 DIR=0x0002 STBY=7200  WAKE=1000  VINT=1193
 DEV=0 Standard
```

### ctrl echo

开关命令行回显。

关闭命令行回显：`ctrl echo 0`。

打开命令行回显：`ctrl echo 1`。

UIMeterTFT默认回显用户输入的字符，可以关闭命令行回显。举例如下：

```
version
 UIMeterTFT v18.8.9 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
ctrl echo 0
 set ECHO to 0...
 UIMeterTFT v18.8.9 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
 set ECHO to 1...
version
 UIMeterTFT v18.8.9 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```

设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl bklt

设置显示背光亮度。

命令格式：`ctrl bklt [hex背光亮度]`。

背光亮度控制字为16位，分高字节与低字节，高字节为待机背光亮度，低字节为工作背光亮度，
0x00背光最亮，0xFF背光最暗，默认0xF880，即正常工作亮度0x80，待机背光亮度0xF8。

设置正常工作背光全亮，待机背光熄灭，命令如下：

```
ctrl bklt ff00
 set BKLT to 0xFF00 and apply...
ctrl
ctrl [echo|bklt|time|dir|stby|iwake] [param] Device Control.
 ECHO=1 BKLT=0xFF00 DIR=0x0002 STBY=7200  WAKE=1000  VINT=1193

```

设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl time

设置设备运行时间。

命令格式：`ctrl time [设备运行秒数]`。

UIMeterTFT上电以后运行时间从0开始自动增加，用户可通过`ctrl time`命令手动设置运行时间。

复位设备运行时间：`ctrl time 0`。

设置设备运行时间为1小时：`ctrl time 3600`。

设置设备运行时间为1天：`ctrl time 86400`。

### ctrl dir

设置屏幕显示方向。

命令格式：`ctrl dir [0|1|2|3]`。

屏幕共4个显示方向：0、1、2、3。默认显示方向为2。

设为默认显示方向：`ctrl dir 2`。

设置以后立即生效，保存参数需要执行`param save`命令。

**可在大字体界面下长按按键来快速旋转屏幕。**

### ctrl stby

设置自动休眠时间。

命令格式：`ctrl stby [十进制秒数]`。

超过设定的秒数以后，设备降低屏幕亮度，降低功耗。

设置为0或者655350关闭该功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl iwake

设置待机唤醒电流。。

命令格式：`ctrl iwake [十进制电流值(单位一个LSB即0.1mA或者0.1uA)]`。

进入待机状态以后，如果电流超过设定值，退出待机状态。

如设置唤醒电流为200mA：`ctrl iwake 2000`

设置0或者65535关闭该功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl dev

设置设备类型。

命令格式：`ctrl dev [0|1]`。

标准版电流分辨率0.1mA，高分辨率版本电流分辨率1uA，使用该命令进行切换。

设置设备为标准版：`ctrl dev 0`。

设置设备为高分辨率版：`ctrl dev 1`。

设置以后立即生效，保存参数需要执行`param save`命令。

## reboot

重启系统。

可以带一个延时参数，单位ms，如果`reboot 900`延时900ms以后重启。

命令输出如下：

```
reboot
 rebooting...
 UIMeterTFT v18.8.9 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
reboot 900
 rebooting...
 UIMeterTFT v18.8.9 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```

## help

获取在线帮助。

命令输出如下：
```
help
 getui -> get U I P R Info.
 clear -> clear power and time Info.
 log -> log [dump|max|int|ring|auto|uh|ul|ih|il] Operate data logs.
 param -> param [load|save|restore] Operate parameters.
 uset -> uset [adj|zero|cali] [adj 100000x|U 10000x] set U param.
 iset -> iset [adj|zero|cali] [adj 100000x|I 10000x] set I param.
 tset -> tset [v30|cali] [V30|T] set T param.
 ctrl -> ctrl [echo|bklt|time|dir|stby|iwake|dev] [param] Device Control.
 reboot -> reboot [delay ms] Restart system.
 help -> help Info.
 version -> display SW version and SN.
```

## version

获取固件和设备序列号等信息。

命令输出如下：
```
version
 UIMeterTFT v18.8.30 SN:832238583632000636345253
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
