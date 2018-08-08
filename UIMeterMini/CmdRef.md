# UIMeterMini命令行手册

UIMeter内置一个命令解释器，可以通过超级终端（或者Putty、SecrueCRT）等软件来连接。
连接以后可以设置设备参数，访问或者导出离线记录数据。

串口参数如图 1所示。波特率115200、8位数据、1位停止、无校验、无流控。
 
![MODBUS串口参数](image/51-MODBUS串口参数.png "MODBUS串口参数")

连接以后输入help回车，可以查看支持的命令，及对应帮助信息。

## getui

getui命令可以获取当前时间、电压、电流、功率、电量等信息。

```
getui
 T=105s U=1mV I=-3mA P=0mW 0mAh 0mWh
```
第三方软件可以借助getui命令采集设备数据。

为简化第三方软件编写难度，可以采用`info echo 0`命令关闭UIMeterMini的命令回显功能。
 
## log
log命令用来操作离线数据。不带参数的log命令输出当前设置。
```
log
log [dump|max|int|ring|auto] Operate data logs.
 current log data length is 4096
 current log interval is 1
 current ring mode is Off
 current auto start log mode is Off
```

### log dump
log dump子命令用来导出离线数据，格式：`log dump [数据条数]`。
数据条数应该小于设备最大记录条数。导出10条数据如下所示：
```
log dump 10
     i,  t(s), U(mV), I(mA)
     0,    17,     1,    -3
     1,    18,     0,    -3
     2,    19,     1,    -3
     3,    20,     0,    -3
     4,    21,     1,    -3
     5,    22,     0,    -3
     6,    23,     1,    -2
     7,    24,     1,    -3
     8,    25,     1,    -2
     9,    26,     1,    -3
```

第一列为数据序号，第二列为采样时间（单位秒），
第三列为电压（单位毫伏），第四列为电流（单位毫安）。
 
可以借助超级终端捕获文字功能保存离线数据。菜单：传送(T)->捕获文字(C)…打开超级终端的捕获文字功能，

![超级终端捕获文字](image/52-超级终端捕获文字.png "超级终端捕获文字")

启动捕获文字，执行完log dump命令，然后停止。保存离线记录数据。
 
离线数据格式为CSV文件，可以使用Excel或者任意一款文本编辑器编辑。

### log max
设置最大记录条数，需要和实际设备配置的EEPROM容量匹配。
UIMeterMini默认配置32kB EEPROM，最大支持4096条离线数据。如果更换更小的EEPROM，
需要使用log max命令设置最大记录数据，如更换16kB EEPROM可以支持2048条离线数据，
需要执行`log max 2048`命令。

2.3	log int
设置离线记录间隔，单位秒，如：`log int 10`，设置离线记录间隔为10秒每次。最大间隔65535秒。

2.4	log ring
默认模式下，记录数据达到最大以后停止记录，通过打开ring模式，可以使数据达到最大
以后自动从0开始记录，覆盖旧数据。该方法可以用于循环记录，跟踪最新的测试数据。

使用`log ring 1`命令打开ring模式。

使用`log ring 0`命令关闭ring模式。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

具体请参考help命令在线帮助。

2.5	log auto
默认情况下，数据离线记录功能需要用户来手动启动。如果用户需要同步采集某些量，
多台UIMeterMini 之间很难完成同步启动采集，因此可以设定UIMeterMini上电启动
以后自动记录数据，通过给多台UIMeterMini同时上电来完成数据同步采集。

使用`log auto 1`命令打开自动记录功能。

使用`log auto 0`命令关闭自动记录功能。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

具体请参考help命令在线帮助。

## info
info命令查看或者设置设备参数。不带参数的info命令获取设备参数。
```
info
info [echo|baud|menu|bklt|halt|coma] Operate parameters.
 BAUD=115200 ECHO=1 MENU=0 BKLT=3 HALT=0s COMA=1
 LOGM=4096 ITVL=1 RING=0 AUTO=0
```
设置参数的命令格式为：`info [子命令] [参数]`
如设置数码管亮度为3：命令为`info bklt 3`
子命令如下表所示：

| 命令名 |  意义           | 取值               |
|:------:|:---------------:|:------------------:|
| echo   | 命令回显标志    | 0/1                |
| baud   | 串口波特率      | 115200             |   
| menu   | 上电默认菜单    | 0-4                |
| bklt   | 数码管亮度      | 0-3                |
| halt   | 自动休眠间隔    | 0-65535秒，0不休眠 |
| coma   | 是否共阳极数码管| 1共阳极，0共阴极   |

### info echo
设置超级终端下命令回显，`info echo 0`关闭回显，`info echo 1`打开回显。效果见图 7所示。
```
version
 UIMeterMini v17.8.5 Flash:16k SN:140036000A57334737373620
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
info echo 0
 Set ECHO status to 0...
 UIMeterMini v17.8.5 Flash:16k SN:140036000A57334737373620
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
 Set ECHO status to 1...
version
 UIMeterMini v17.8.5 Flash:16k SN:140036000A57334737373620
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
命令回显关闭以后，UIMeterMini将不会回显用户输入的命令，主要用于软件通过命令
查询UIMeterMini的数据场合。如：可以通过主机发送字符串`getui\r\n`获取设备信息，
UIMeterMini将会在收到整个字符串以后发送电压、电流、功率信息。

### info baud
设置设备波特率，支持波特率：9600、19200、38400、57600、115200（默认）。
设置以后保存参数并且重启生效。

如果忘记设置的波特率，上电时按住左键，强制使用默认115200波特率，
使用115200波特率连上设备以后设置新波特率。无特殊需求建议使用默认波特率。

### info menu
设置上电默认菜单。

| 编号| 名称     |备注   |
|:---:|:--------:|:-----:|
| 0   | 电压     | 默认  |
| 1   | 时间计数 |       |
| 2   | 电流     |       |   
| 3   | 功率     |       |
| 4   | 离线计数 |       |

UIMeterMini上电默认显示电压，可以通过`info menu`命令设置其它上电菜单。
如需要上电直接显示电流，输入命令`info menu 3`，然后使用`param save`命令保存。
reboot命令重启即可。

给命令仅仅影响上电初始界面，上电以后，可以通过按键切换界面。

### info bklt
设定数码管亮度，`info bklt 0/1/2/3`，数字越大，数码管越亮，功耗越大。
默认亮度为0，适合绝大多数应用场合。

### info halt
设定自动休眠时间。最大时间65535秒。设置为0时，关闭自动休眠功能。
如：`info halt 10`设置自动休眠时间为10秒，如果10秒之内没有按键操作，没有串口通讯数据，
设备将会进入休眠状态，消耗电流降低到170uA左右，可以长期连接电池使用。

### info coma
设置数码管类型。

不同批次产品使用的数码管类型可能不同，软件中需要设定。coma=1表示共阳极数码管，coma=0表示共阴极数码管。

`info coma 1`设置为共阳极数码管。

`info coma 0`设置为共阴极数码管。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

## uset
电压校准相关命令。uset命令带两个子命令：adj、cali。不带子命令的uset命令输出当前电压和电流相关校正系数。

|  命令名  |     意义                   | 取值       |
|:--------:|:--------------------------:|:----------:|
|   uadj   | 电压校正系数，扩大10000倍  | 0-65535    |
|   iadj   | 电流校正系数，扩大10000倍  | 0-65535    |
|   izro   | 电流零偏校正               | -1000~1000 |

### uset adj
uset adj命令设置电压校正系数。系数扩大10000倍，即10020表示1.0020，执行情况见图 8。
```
uset
 UADJ=10000 IADJ=10000 IZRO=0
uset adj 10020
 Set UADJ to 10020...
```
设定以后，如果需要掉电保存参数，需要执行`param save`命令。

### uset cali
电压自动校准命令。
先保证电压校正系数为10000（1.0000），如果不是，可以使用命令`uset adj 10000`设置电压校正系数为10000.

将设备接入已知基准电压，如5V，输入`uset cali 5000`，设备可以自动计算出电压校正系数。输入电压单位为mV。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

## iset
电流校准相关命令。iset命令带三个子命令：adj、cali、zro。不带子命令的uset命令输出当前电压和电流相关校正系数。

### iset adj
iset adj命令设置电流校正系数。系数扩大10000倍，即10030表示1.0030，执行情况如下。
```
iset
 UADJ=10000 IADJ=10000 IZRO=0
iset adj 10030
 Set IADJ to 10030...
```
设定以后，如果需要掉电保存参数，需要执行`param save`命令。

### iset cali
电流自动校准命令。
先保证电流校正系数为10000（1.0000），如果不是，可以使用命令`iset adj 10000`设置电压校正系数为10000.

将设备接入已知电流，如1A，输入`iset cali 1000`，设备可以自动计算出电流校正系数。输入电流单位为mA。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

### iset zro
设置电流零偏校正。
设备实际电流为0时显示的电流值为零偏电流，可以使用`iset zro`命令校正。
如实际电流为0，实际显示0.003，即电流3mA，使用命令`iset zro 3`设置零偏校正为3mA。

设定以后，如果需要掉电保存参数，需要执行`param save`命令。

## param

param命令带三个子命令：load、save、restore。

`param load`命令从内置EEPROM加载保存的参数。

`param save`命令将参数保存到内置EEPROM。

`param restore`命令恢复默认参数。同时按住左右键上电也可以恢复默认参数。
```
param
param [load|save|restore] Operate parameters.
param restore
 Restore default parameters...
param load
 Load parameters from EEPROM...
param save
 Save parameters to EEPROM...
```

## reboot
重启UIMeterMini。
可以带一个延时参数，单位ms，如果`reboot 900`延时900ms以后重启。
```
reboot 900
 reboot xboot ...
 UIMeterMini v17.8.5 Flash:16k SN:140036000A57334737373620
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
 
## help
获取在线帮助。
 
```
help
 getui -> get voltage current and power etc.
 log -> log [dump|max|int|ring|auto] Operate data logs.
 info -> info [echo|baud|menu|bklt|halt|coma] Set/Show params.
 uset -> uset [adj|cali] Set/Show Voltage params.
 iset -> iset [adj|cali|zro] Set/Show Current params.
 param -> param [load|save|restore] Operate parameters.
 reboot -> reboot [delay ms] Restart system.
 help -> help Info.
 version -> display SW version and SN.
```

## version
获取固件和设备序列号等信息。
```
version
 UIMeterMini v17.8.5 Flash:16k SN:140036000A57334737373620
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```