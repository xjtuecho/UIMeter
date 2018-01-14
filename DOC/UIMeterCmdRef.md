# UIMeter命令行手册

UIMeter内置一个命令解释器，可以通过超级终端（或者Putty、SecrueCRT）等软件来连接。
连接以后可以通过串口使用UIMeter全部功能。使用串口命令之前，需要切换通讯协议到
TERM协议，切换方法请参考UIMeter用户手册。

串口参数如下所示：

![MODBUS串口参数](image/51-MODBUS串口参数.png "MODBUS串口参数")

波特率115200、8位数据、1位停止、无校验、无流控。
 
本文档基于UIMeter固件v17.7.1，其余固件版本仅供参考。

随着新固件发布，相关命令可能会有调整，恕不另行通知。

## getui
获取当前电压、电流、时间、功率、电量等信息。

命令格式：`getui`

命令输出如下所示：
```
getui
 U: PGA=8 AD=0x000003  0.0000V 0.0000W      1uV
 I: PGA=8 AD=0x000000  0.0000A 9999.9R      0uV
 T: RAW=0x1600  22.0C   22.0C
 P: 0.0000Ah  0.0000Wh     32s
```
第三方工具可以使用该命令采集当前数据。

第三方软件可以通过执行该命令查询设备实时数据，可以关闭命令回显，降低第三方软件编写难度。

## clear
清除设备当前时间和电量信息。

命令格式：`clear`。

设备上电以后，运行时间和Ah电量、Wh电量会一直累计，执行该命令以后，运行时间、Ah电量、Wh电量全部归零。
其余数据为实时更新，该命令无影响。

## log
操作离线数据。

命令格式：`log [dump|max|int|ring|auto|uh|ul|ih|il] Operate data logs.`

不带参数的log命令输出当前设置，依次显示数据最大记录条数、记录间隔、RING模式开关、
AUTO模式开关，UH、UL、IH、IL四个参数。具体意义见以下子命令说明。
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

### log dump
log dump子命令用来导出离线数据。

命令格式：`log dump [数据条数]`。

数据条数应该小于设备最大记录条数。导出10条数据如下：
```
log dump 10
    i,    t(s),    U(V),    I(A), Tself, Tprob
    0,      11,  0.0001,  0.0000,  26.5,  26.5
    1,      12,  0.0000,  0.0000,  26.5,  26.5
    2,      13,  0.0001,  0.0000,  26.5,  26.5
    3,      14,  0.0000,  0.0000,  26.5,  26.5
    4,      15,  0.0000,  0.0000,  26.5,  26.5
    5,      16,  0.0000,  0.0000,  26.5,  26.5
    6,      17,  0.0000,  0.0000,  26.5,  26.5
    7,      18,  0.0000,  0.0000,  26.5,  26.5
    8,      19,  0.0000,  0.0000,  26.5,  26.5
    9,      20,  0.0000,  0.0000,  26.5,  26.5
```

第一列为数据索引，
第二列为设备记录数据时的相对时间（单位秒），
第三列为电压（启用分流器测量功能以后为分流器电流），
第四列为电流，
第五列为冷端温度（环温），
第六列为探头温度（K型热电偶测温时有意义）
 
可以借助超级终端捕获文字功能保存离线数据。菜单：传送(T)->捕获文字(C)…
打开超级终端的捕获文字功能，启动捕获文字，执行完log dump命令，然后停止。
保存离线记录数据。
 
![超级终端捕获文字](image/55-超级终端捕获文字.png "超级终端捕获文字")

离线数据格式为CSV文件，可以使用Excel或者任意一款文本编辑器编辑。

v17.5.11固件增加分流器电流测量功能，启用分流器电流测量功能以后，原电压档记录数据为分流器电流。

### log max
设置最大记录条数。

UIMeter具备2048和4096两种离线记录条数。可以使用log max命令设置最大记录数据。
```
log max 2
 Set Max data log to  2048
log max 4
 Set Max data log to  4096
```
最大记录条数需要和实际设备配置的EEPROM容量匹配，32kB对应2048条，64kB对应4096条。

设置最大记录条数为2048：`log max 2`

设置最大记录条数为4096：`log max 4`

设置以后立即生效，保存参数需要执行`param save`命令。

### log int
设置离线记录间隔，单位秒，如：`log int 10`，设置离线记录间隔为10秒每次。最大间隔65535秒。

设置以后立即生效，保存参数需要执行`param save`命令。

### log ring
打开或者关闭RING模式。

默认模式下，记录数据达到最大以后停止记录，通过打开RING模式，可以使数据达到最大
以后自动从0开始记录，覆盖旧数据。该方法可以用于循环记录，跟踪最新的测试数据。

使用`log ring 1`命令打开RING模式，使用`log ring 0`命令关闭RING模式。

设置以后立即生效，保存参数需要执行`param save`命令。

### log auto
打开或者关闭AUTO模式。

默认情况下，数据离线记录功能需要用户来手动启动。如果用户需要同步采集某些量，
多台UIMeter之间很难完成同步启动采集， 因此可以打开AUTO模式，UIMeter上电以后
自动开始记录数据。通过给多台UIMeter同时上电来完成数据同步采集。

使用`log auto 1`命令打开自动记录功能，使用`log auto 0`命令关闭自动记录功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### log [uh|ul|ih|il]
设定UH、UL、IH、IL四个参数。

四个参数决定离线记录的条件。UH和UL设定电压上限和下限，IH和IL设定电流上限和下限。

记录规则如下：

 1. UH>UL。电压上限高于电压下限，实际电压高于下限并且低于上限时记录数据。
 2. UH<UL。电压上限低于电压下限，实际电压高于下限或者低于上限时记录数据。
 3. UH=UL。电压上限等于电压下限，离线记录数据和电压无关，只与电流有关。
 4. IH>IL。电流上限高于电流下限，实际电流高于下限并且低于上限时记录数据。
 5. IH<IL。电流上限低于电流下限，实际电流高于下限或者低于上限时记录数据。
 6. IH=IL。电流上限等于电流下限，离线记录数据和电压电流均无关，即全部记录。

注意，UH、UL、IH、IL为在原来手动记录基础上增加的四个条件，设置以后仍然需要手动
启停记录，如果不使用需要全部设置为0.

设置命令举例参考如下：
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

设置以后立即生效，保存参数需要执行“param save”命令。

## info
查看或者设置设备参数。

命令格式：`info [dev|lcd|probe|addr|alarm|baud] Display/Set system Info.`

不带参数的info命令获取设备参数。
```
info
 info [dev|lcd|probe|addr|alarm|baud] Display/Set system Info.
 DeviceType: STD_V20
 LcdType: LCD1602
 ProbeType: TYPEK
 ModbusAddr:   1 Baud: 115200
 ALARM: 1
 UART: TERM
 ECHO:1
 BackLight: 0x8
```
设置参数的命令格式为：`info [子命令] [参数]`

子命令如下表所示：

| 命令名 | 意义                 | 取值                          |
|:------:|:--------------------:|:-----------------------------:|
| dev    | 设置设备类型         | 1:标准版 2:高分辨率版本       |
| lcd    | 设置屏幕类型         | 0:1602屏 1:LCD彩屏            |
| probe  | 设置温度探头类型     | 0:K型热电偶 1:PT100 2:5k欧NTC |
| addr   | 设置串口地址         | 1-247                         |
| alarm  | 设置ALARM开关        | 0:关闭 1:开启                 |
| baud   | 设置MODBUS协议波特率 | 115200bps-2400bps             |

### info dev
UIMeter所有版本使用相同的固件，用info dev命令来进行区分。
- 使用`info dev 2`命令设置为高分辨率版本，配合1uA电流分辨率。
- 其它版本使用`info dev 1`命令设置，配合0.1mA电流分辨率。

设置以后立即生效，保存参数需要执行`param save`命令。

### info lcd
UIMeter兼容1602屏幕和TFT彩屏。
- `info lcd 0`设置为1602屏幕。
- `info lcd 1`设置为TFT彩屏。

设置以后执行`param save`命令保存参数，重启生效。

### info probe
UIMeter兼容三种温度探头。
- `info probe 0`设置为K型热电偶。
- `info probe 1`设置为PT100。
- `info probe 2`设置为5k欧NTC电阻。

设置以后立即生效，保存参数需要执行`param save`命令。

### info addr
设置串口地址，地址范围1-247.
- `info addr 2`设置串口地址为2，该地址与MODBUS协议地址相同。

设置以后立即生效，保存参数需要执行`param save`命令。

### info alarm
UIMeter的ALARM功能包括一个输出MOS管和一个LED指示灯，上电默认由UIMeter控制。
如果用户需要手动控制MOS管和LED指示灯，需要首先关闭ALARM功能。
- `info alarm 0`关闭ALARM功能。
- `info alarm 1`打开ALARM功能。

设置以后立即生效，掉电不保存。

### info baud
设置MODBUS协议串口波特率，支持波特率：115200、57600、38400、19200、9600、4800、2400。
- `info baud 9600`设置波特率为9600bps。
设置以后执行`param save`命令保存，然后切换为MODBUS协议生效。

**该命令仅仅对MODBUS协议生效。TERM协议仍然使用115200固定波特率**。

## param
操作用户参数。

命令格式：`param [load|save|restore] Operate parameters.`

param命令带三个子命令：load、save、restore。
- `param load`命令从内置EEPROM加载保存的参数。
- `param save`命令将参数保存到内置EEPROM。
- `param restore`命令恢复默认参数。同时按住左右键上电也可以恢复默认参数。

## uset
电压通道参数设置。

命令格式：`uset [adj|zero|max|min|cali] [adj 100000x|U 10000x] set U param.`

不带参数的uset命令输出当前电压通道和电流通道的所有参数。
```
uset
uset [adj|zero|max|min|cali] [adj 100000x|U 10000x] set U param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 20.0000V   U Min:  0.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```

### uset adj
设置电压增益校正系数。

电压增益校正系数是一个1附近的数值，用来校正分压电阻、基准初始值等带来的误差。
范围0-100，如果UIMeter显示电压数值小于实际电压值，需要增大电压增益校正系数，
反之减小电压增益校正系数。

使用uset adj命令将电压增益校正系数设置为1.00234，如下所示，设定数值需要扩大100000倍去掉小数点。
```
uset adj 100234
uset
uset [adj|zero|max|min|cali] [adj 100000x|U 10000x] set U param.
 U Adj:  1.00234   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 20.0000V   U Min:  0.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```
 
### uset zero
设置电压零偏校正系数。

如果电压通道在短接测量端时不为0，需要校正电压零偏。单位为一个电压分辨率。

如果电压显示0.0003V，使用`uset zero 3`命令校正。

如果电压显示-0.0002V，使用`uset zero -2`命令校正。

### uset [max|min]
设置电压上限、电压下限。

UIMeter使用电压上限、电压下限控制输出MOS管和LED。测量电压高于下限并且低于下限
时打开输出MOS管熄灭LED；测量电压高于上限或者低于下限关闭输出MOS管点亮LED。

设置方法举例如下：
```
uset max 100000
uset min 20000
uset
uset [adj|zero|max|min|cali] [adj 100000x|U 10000x] set U param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 10.0000V   U Min:  2.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```
电压数值需要扩大10000倍去掉小数。

### uset cali
电压快速校准。

首先使用`uset adj 100000`命令将电压增益系数设置为1.
将UIMeter电压通道与基准电压源并联，读取基准源电压值，执行命令 `uset cali [基准电压值]` 
UIMeter自动计算校准系数，保证电压显示值与基准电压值相等，基准电压值需要扩大10000倍去掉小数。

设置以后立即生效，保存参数需要执行`param save`命令。

## iset
电流通道参数设置。

命令格式：`iset [adj|zero|cali] [adj 100000x|I 10000x] set I param.`

不带参数的iset命令输出当前电压通道和电流通道的所有参数。
```
iset
iset [adj|zero|cali|shunt|gain] [adj 100000x|I 10000x] set I param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 10.0000V   U Min:  2.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```

### iset adj
设置电流增益校正系数。

电流增益校正系数是一个1附近的数值，用来校正检流电阻、基准初始值等带来的误差，
范围0-100，如果UIMeter显示电流数值小于实际电流值，需要增大电流增益校正系数，
反之减小电流增益校正系数。

使用iset adj命令将电流增益校正系数设置为1.00234，设定数值需要扩大100000倍去掉小数点。
```
iset adj 100234
iset
iset [adj|zero|cali|shunt|gain] [adj 100000x|I 10000x] set I param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  1.00234   I Zero:       0
 U Max: 10.0000V   U Min:  2.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```

### iset zero
设置电流零偏校正系数。

如果电流通道在测量端悬空时不为0，需要校正电流零偏。单位为一个电流分辨率。

如果电流显示0.0003A，使用`iset zero 3`命令校正。

如果电流显示-0.0002A，使用`iset zero -2`命令校正。

### iset cali
电流快速校准。

首先使用`iset adj 100000`命令将电流增益校正系数设置为1.
将UIMeter电流通道与基准电流源串联，读取基准源电流值，执行命令 `iset cali [基准电流值]`
UIMeter自动计算校准系数，保证电流显示值与基准电流值相等，基准电流值需要扩大10000倍去掉小数。

设置以后立即生效，保存参数需要执行`param save`命令。

### iset [shunt|gain]
设置分流器量程和增益校正系数。该命令从v17.5.11固件开始支持。

UIMeter从v17.5.11固件开始支持使用电压档测量分流器。短接J4跳线右侧两位，
然后设置分流器量程以后即可使用电压档测试分流器电流。

一般分流器输出满量程电压均为75mV。75mV电压除以量程即为分流器电阻。
如75mV 150A分流器，电阻为`75mV/150A=0.5mR`。

UIMeter采样分流器微小电阻上的压降，然后根据用户设置的分流器量程
计算出流过分流器上的电流。

命令格式：`iset shunt [分流器量程]`

命令格式：`iset gain [扩大100000倍后的增益校正系数]`

分流器量程设置为100A校正系数设置为1.00111，命令如下：
```
iset shunt 100
iset gain 100111
iset
iset [adj|zero|cali|shunt|gain] [adj 100000x|I 10000x] set I param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 20.0000V   U Min:  0.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:   100A Gain:  1.00111
```
分流器量程设置为0A校正系数设置为1，命令如下：
```
iset shunt 0
iset gain 100000
iset
iset [adj|zero|cali|shunt|gain] [adj 100000x|I 10000x] set I param.
 U Adj:  0.99597   U Zero:       2
 I Adj:  0.99602   I Zero:       0
 U Max: 20.0000V   U Min:  0.0000V
 U Hys:  0.5000V   ChkNum:       4
 75mV SHUNT Range:     0A Gain:  1.00000
```
分流器量程设置范围1A-65534A，可兼容市面上绝大多数分流器。

分流器量程设置为0A或者65535A时，关闭分流器电流测量功能。
  
一般用户设置分流器量程以后即可准确测量大电流，如果用户具备大电流校准条件，
可以使用iset gain命令进一步提高测量精度。

设置以后立即生效，保存参数需要执行`param save`命令。

## ctrl
设备控制命令。

命令格式：`ctrl [echo|bklt|led|mos|time] [param] Device Control.`

通过设备控制命令可以控制设备的运行参数。

### ctrl echo
开关命令行回显。
- 关闭命令行回显：`ctrl echo 0`。
- 打开命令行回显：`ctrl echo 1`。

UIMeter默认回显用户输入的字符，可以关闭命令行回显。
```
version
 UIMeter v17.12.6 SN:FF56066E7875524828586719
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
ctrl echo 0
 ECHO=0
 UIMeter v17.12.6 SN:FF56066E7875524828586719
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
 ECHO=1
version
 UIMeter v17.12.6 SN:FF56066E7875524828586719
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
 设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl bklt
设置背光亮度。

命令格式：`ctrl bklt [00-FF]`

参数为十六进制，0x00到0xFF，0x00最亮，0xFF最暗关闭背光。
可以通过info命令查看当前背光亮度。命令举例如下：
- 设置背光最亮：`ctrl bklt 00`
- 完全关闭背光：`ctrl bklt FF`
- 设置背光一半亮度：`ctrl bklt 80`

设置以后立即生效，保存参数需要执行`param save`命令。

### ctrl led
开关指示LED。
- 关闭指示LED：`ctrl led 0`。
- 打开指示LED：`ctrl led 1`。

UIMeter指示LED默认由软件自动控制，用户也可以手动控制。
用户使用`info alarm 0`命令关闭软件控制LED，可以使用`ctrl led`命令控制LED。
```
ctrl led 0
 set ALARM to 0 first, aborting...
info alarm 0
 ALARM: 0
ctrl led 0
ctrl led 1
```

### ctrl mos
开关输出MOS管。
- 关闭输出MOS管：`ctrl mos 0`。
- 打开输出MOS管：`ctrl mos 1`。

UIMeter输出MOS管默认由软件自动控制，用户也可以手动控制。用户使用`info alarm 0`
命令关闭软件控制输出MOS管，可以使用`ctrl mos`命令控制输出MOS管。
```
ctrl mos 0
 set ALARM=0 first, aborting...
info alarm 0
 ALARM: 0
ctrl mos 0
ctrl mos 1
```

### ctrl time
设置设备运行时间。

命令格式：`ctrl time [设备运行秒数]`

命令举例如下：
- 复位设备运行时间：`ctrl time 0`
- 设置设备运行时间为1小时：`ctrl time 3600`
- 设置设备运行时间为1天：`ctrl time 86400`

UIMeter上电以后运行时间从0开始自动增加，用户可通过ctrl time命令手动设置运行时间。

### ctrl dir
设置TFT屏幕显示方向。

命令格式：`ctrl dir [0|1|2|3]`

共TFT屏幕共4个显示方向0、1、2、3。默认显示方向为0。

设为默认显示方向：`ctrl dir 0`

该命令只对TFT屏幕设备有效，设置以后需要保存参数重启生效。

### ctrl menu
设置设备上电初始菜单。

命令格式：`ctrl menu [0|1|2|3|4|5|6|7]`

菜单编号意义见下表。

| 序号 | 菜单       | 屏幕 | 备注 |
|:----:|:----------:|:----:|:----:|
| 0    | 电压电流   | 1602 | 默认 |
| 1    | 电量时间   | 1602 |      |
| 2    | 温度测量   | 1602 |      |
| 3    | 离线记录   | 1602 |      |
| 4    | 关于信息   | 1602 |      |
| 5    | 主界面     | TFT  | 默认 |
| 6    | 大字体界面 | TFT  |      |
| 7    | 设备信息   | TFT　|      |

举例：TFT屏幕设置上电初始菜单为大字体界面：`ctrl menu 6`

设置以后需要保存参数重启生效。

## sethys
设置电压检测滞环与检测次数。

命令格式：`sethys  [dec hys 10000x] [dec chkNum] set Hysteresis.`

UIMeter默认通过用户设置的最高电压最低电压控制输出MOS管和指示LED，
检测到实际电压高于最高电压或者低于最低电压时断开输出MOS管，点亮指示LED。

锂电池放电为例，设置放电截止电压3V，放电到3V时断开输出MOS管，由于电池内阻和
线路电阻的存在，电池电压会升高到3.3V以上，导致重新打开输出MOS管再次放电。
假如设置3V时断开MOS管，3.5V打开MOS管，0.5V电压差定义为检测滞环。

同样以锂电池放电为例，设置放电截止电压3V，UIMeter连续检测N次电压低于3V以后，
断开输出MOS管，N定义为检测次数。

注意：如果没有特殊需要，请使用默认值。

## reboot
重启系统。

可以带一个延时参数，单位ms，如果`reboot 900`延时900ms以后重启。

命令输出如下：
```
reboot
 rebooting...
 UIMeter v17.12.6 SN:FF56066E7875524828586719
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
reboot 900
 rebooting...
 UIMeter v17.12.6 SN:FF56066E7875524828586719
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
 info -> info [dev|lcd|probe|addr|alarm|baud] Display/Set system Info.
 param -> param [load|save|restore] Operate parameters.
 uset -> uset [adj|zero|max|min|cali] [adj 100000x|U 10000x] set U param.
 iset -> iset [adj|zero|cali|shunt|gain] [adj 100000x|I 10000x] set I param.
 ctrl -> ctrl [echo|bklt|led|mos|time|dir|menu] [param] Device Control.
 reboot -> reboot [delay ms] Restart system.
 help -> help Info.
 version -> display SW version and SN.
```

## version
获取固件和设备序列号等信息。

命令输出如下：
```
version
 UIMeter v17.12.6 SN:FF56066E7875524828586719
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```