# UIMeterDual Command Line Reference

UIMeterDual内置一个命令解释器，可以通过超级终端（或者Putty、SecrueCRT）等软件来连接。

串口参数：`波特率115200、8位数据、1位停止、无校验、无流控`。

本文档基于UIMeter固件v19.6.19，其余固件版本仅供参考。

## 更新历史

### v19.6.19

初始发布。

## getui

获取两个通道电压电流信息。

命令格式：`getui` 

```
getui
 CHA:  0.0000V  0.0000A  0.0000W U:0x0000 I:0x0000
 CHB:  0.0000V  0.0000A  0.0000W U:0x0000 I:0x0000
```

数据分两行：

- 第一行为通道A电压、电流、功率、AD采样原始值
- 第二行为通道B电压、电流、功率、AD采样原始值

第三方工具可以使用该命令采集当前数据。

## clear

清零两个通道Ah，Wh和时间信息，无参数，无输出。

## log

操作离线记录数据。

命令格式：`log [dump|cha|chb|file|max|int|ring|auto|cross] Operate data logs.`

不带参数的log命令输出当前离线记录参数设置:

```
log [dump|cha|chb|file|max|int|ring|auto|cross] Operate data logs.
 Log FILE=0 MAX=8 INT=0 RING=0 AUTO=0 CROSS=0
```

参数意义见下表：

|  参数  |  意义             | 范围    | 复位值   |  备注                        |
|:------:|:-----------------:|:-------:|:--------:|:----------------------------:|
| FILE   | 记录文件编号      | 0-7     |   0      |                              |
| MAX    | 最大记录文件数    | 2,4,8,16|   8      | 需要与FLASH尺寸匹配          |
| INT    | 数据记录周期      | 0-65535 |   1      | 单位秒 0表示0.25秒           |
| RING   | 是否循环记录      | 0-1     |   0      | 文件7写满以后是否写文件0     |
| AUTO   | 是否上电自动记录  | 0-1     |   0      | 0:关闭 1:开启                |
| CROSS  | 是否跨文件记录    | 0-1     |   0      | 文件x写满后是否继续写文件x+1 |

UIMeterDual内置8MB FLASH，离线记录数据分为8个文件，编号0-7，每个文件内部最大可以记录16384条数据。

默认行为是记录到当前文件，用户可以指定文件编号，由于FLASH写入寿命有限，建议不要一直写同一个文件。

当前文件写满16384条记录以后，如果CROSS=0，记录结束，如果CROSS=1，继续写入下一个文件。

文件7写入完以后，如果RING=0，记录结束，如果RING=1，继续写文件1，如此往复，循环记录。

常见记录场景如下：

- 单次记录不超过16384条：设置FILE=x(0-7)，CROSS=0，RING=0。
- 单次记录不超过16384*8条：设置FILE=0，CROSS=1，RING=0。
- 最大容量16384*8循环记录：设置FILE=0，CROSS=1，RING=1。

### log dump

`log dump`子命令用来导出离线数据。

命令格式：`log dump [dec start] [dec len]`。

两个参数分别为导出开始位置和导出长度，导出长度如果不指定默认10条。

导出从第5条开始5条记录，如下所示：

```
log dump 5 5
       i,    t(s),   UA(V),   IA(A),   UB(V),   IB(A)
       5,    2023,  0.0000,  0.0000,  0.0000,  0.0000
       6,    2023,  0.0000,  0.0000,  0.0000, -0.0001
       7,    2023,  0.0000,  0.0000,  0.0000,  0.0000
       8,    2024,  0.0000,  0.0000,  0.0000,  0.0000
       9,    2024,  0.0000,  0.0000,  0.0000,  0.0000
```

- 第一列为文件内部记录编号
- 第二列为设备记录数据时的相对时间（单位秒）
- 第三列为通道A电压
- 第四列为通道A电流
- 第五列为通道B电压
- 第六列为通道B电流
 
可以借助超级终端捕获文字功能保存离线数据。菜单：传送(T)->捕获文字(C)…
打开超级终端的捕获文字功能，启动捕获文字，执行完log dump命令，然后停止。
保存离线记录数据。

离线数据格式为CSV文件，可以使用Excel或者任意一款文本编辑器编辑。

### log cha

`log cha`子命令用来查看通道A更多信息。

命令格式：`log cha [dec start] [dec len]`。

两个参数分别为导出开始位置和导出长度，导出长度如果不指定默认10条。

导出从第3条开始5条记录，如下所示：

```
log cha 3 5
       i,    t(s),   UA(V),   IA(A),   PA(W),    EffA,   mAh_A,   mWh_A
       3,    2022,  0.0000,  0.0001,  0.0000,  0.0000,       0,       0
       4,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       5,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       6,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       7,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
```

- 第一列为文件内部记录编号
- 第二列为设备记录数据时的相对时间（单位秒）
- 第三列为通道A电压
- 第四列为通道A电流
- 第五列为通道A功率
- 第六列为通道A效率（A/B）
- 第七列为通道mAh电量
- 第八列为通道mWh电量

### log chb

`log chb`子命令用来查看通道A更多信息。

命令格式：`log chb [dec start] [dec len]`。

两个参数分别为导出开始位置和导出长度，导出长度如果不指定默认10条。

导出从第3条开始5条记录，如下所示：

```
log chb 3 5
       i,    t(s),   UB(V),   IB(A),   PB(W),    EffB,   mAh_B,   mWh_B
       3,    2022,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       4,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       5,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
       6,    2023,  0.0000, -0.0001,  0.0000,  0.0000,       0,       0
       7,    2023,  0.0000,  0.0000,  0.0000,  0.0000,       0,       0
```

- 第一列为文件内部记录编号
- 第二列为设备记录数据时的相对时间（单位秒）
- 第三列为通道A电压
- 第四列为通道A电流
- 第五列为通道A功率
- 第六列为通道A效率（A/B）
- 第七列为通道mAh电量
- 第八列为通道mWh电量

### log file

`log file`子命令用来查看、设置当前文件编号。

命令格式：`log file [dec file index] Set log file index(0~7).`。

查看当前文件编号，并把文件编号设置为1，命令如下：

```
log file
 log file [dec file index] Set log file index(0~7).
 current log file index is 0
log file 1
 Set log file index to 1
```

### log max

`log max`子命令用来查看、设置最大文件数量，不能大于FLASH的MB容量。

命令格式：`log max [dec file max] Set log file max.`。

查看当前最大文件数量，并把最大文件数量设置为4，命令如下：

```
log max
 log max [dec file max] Set log file max.
 current log file max is 8
log max 4
 Set log file max to 4
```

### log int

`log int`子命令用来查看、设置离线记录周期。

命令格式：`log int [dec interval] Set log interval.`。

```
log int
 log int [dec interval] Set log interval.
 current log interval is 0
log int 1
 Set log interval to 1
```

### log ring

`log ring`子命令用来查看、设置循环记录模式。

命令格式：`log ring [0|1] Turn On/Off ring mode.`。

```
log ring
 log ring [0|1] Turn On/Off ring mode.
 current ring mode is Off
log ring 1
 Set Ring Mode to On
```

### log auto

`log auto`子命令用来查看、设置上电自动记录模式。

命令格式：`log auto [0|1] Turn On/Off auto start log mode.`。

```
log auto
 log auto [0|1] Turn On/Off auto start log mode.
 current auto start log mode is Off
log auto 1
 set auto start log mode to On
```

### log cross

`log cross`子命令用来查看、设置跨文件记录模式。

命令格式：`log cross [0|1] Turn On/Off cross file log mode.`。

```
log cross
 log cross [0|1] Turn On/Off cross file log mode.
 current cross file log mode is Off
log cross 1
 set cross file log mode to On
```

## info

查看设置系统信息。

命令格式：`info [baud|echo|bklt|lcd|time] Operate parameters.`

不带参数的info命令显示当前系统信息：

```
info [baud|echo|bklt|lcd|time] Operate parameters.
 BAUD=115200 ECHO=1 BKLT=0xA0 LCD=LCD1602 TIME=316s
```
参数意义见下表：

|  参数 |  意义       | 范围        | 复位值     |  备注              |
|:-----:|:-----------:|:-----------:|:----------:|:------------------:|
| BAUD  | 串口波特率  | 9600-115200 | 115200     |                    |
| ECHO  | 命令回显开关| 0-1         | 1          |                    |
| BKLT  | LCD背光     | 0x00-0xFF   | 0x80       | 0x00最亮0xFF最暗   |
| LCD   | LCD类型     | 0-1         | 0(LCD1602) | 暂时只支持1602屏幕 |
| TIME  | 运行时间    | 0-2^31      | 0          |                    |

### info baud

查看设置串口波特率。

命令格式：`info baud [baud]`。

不带参数的`info baud`命令显示当前波特率，支持的波特率有：

- 115200
- 57600
- 38400
- 19200
- 9600

```
info baud
 UART baud = 115200
info baud 57600
 set UART baud to 57600, save & reboot to apply new baud.
```

设置以后执行`param save`命令保存参数，重启生效。

### info echo

开关命令行回显。

- 关闭命令行回显：`info echo 0`。
- 打开命令行回显：`info echo 1`。

### info bklt

显示设置背光亮度。

命令格式：`info bklt [00-FF]`

参数为十六进制，0x00到0xFF，0x00最亮，0xFF最暗关闭背光。

可以通过info命令查看当前背光亮度。命令举例如下：

- 设置背光最亮：`info bklt 00`
- 完全关闭背光：`info bklt FF`
- 设置背光一半亮度：`info bklt 80`

设置以后立即生效，保存参数需要执行`param save`命令。

### info lcd

查看设置LCD类型。**注意**本设备暂时只支持1602屏幕。

- `info lcd 0`设置为1602屏幕。
- `info lcd 1`设置为TFT彩屏。

### info time

显示设置设备运行时间。

命令格式：`ctrl time [设备运行秒数]`

命令举例如下：

- 复位设备运行时间：`info time 0`
- 设置设备运行时间为1小时：`info time 3600`
- 设置设备运行时间为1天：`info time 86400`

UIMeterDual上电以后运行时间从0开始自动增加，用户可通过`info time`命令手动设置运行时间。

## adj

查看设置ADC采样比例系数。

命令格式：`adj [ua|ia|ub|ib] [adj_100000x] set sample gain adjustment.`

- 第一个参数为要设置的通道，u表示电压，i表示电流，a表示A通道，b表示B通道。
- 第二个参数为要设置的参数，扩大100000倍。
- 不带参数输出当前的采样比例系数。

```
adj
adj [ua|ia|ub|ib] [adj_100000x] set sample gain adjustment.
 UadjA: 1.00000 UadjB: 1.00000
 IadjA: 1.00000 IadjB: 1.00000
adj ua 100234
 Set UadjA to  1.00234...
adj
adj [ua|ia|ub|ib] [adj_100000x] set sample gain adjustment.
 UadjA: 1.00234 UadjB: 1.00000
 IadjA: 1.00000 IadjB: 1.00000
```

## zero

查看设置ADC采样零偏系数。

命令格式：`zero [ua|ia|ub|ib] [LSB] set sample zero adjustment.`

- 第一个参数为要设置的通道，u表示电压，i表示电流，a表示A通道，b表示B通道。
- 第二个参数为要设置的参数，单位为LSB，可以为负值。
- 不带参数输出当前的采样零偏系数。

```
zero
zero [ua|ia|ub|ib] [LSB] set sample zero adjustment.
 UzeroA:     0 IzeroA:   -24
 UzeroB:     0 IzeroB:   -24
zero ua -10
 Set UzeroA to   -10 ...
zero
zero [ua|ia|ub|ib] [LSB] set sample zero adjustment.
 UzeroA:   -10 IzeroA:   -24
 UzeroB:     0 IzeroB:   -24
```

## cali

电压电流快速校准。

命令格式：`cali [ua|ia|ub|ib] [U_10000X|I_10000X] Voltage or Current calibration.`。

首先使用`adj`命令将增益系数设置为1。

将UIMeterDual电压或者电流连接进准源，读取基准源电压或者电流值。

执行命令 `cali [ua|ia|ub|ib] [基准值]`。

UIMeterDual自动计算校准系数，保证显示值与基准值相等，基准值需要扩大10000倍去掉小数。

## eeprom

查看、设置内部模拟EEPROM内容。

命令格式：`eeprom [load|save|read|write] [addr] [data] Operate Int. EEPROM.`

### eeprom load

加载eeprom：`eeprom load`，数据从FLASH加载到内部缓存。

### eeprom save

存储eeprom：`eeprom save`，eeprom数据从内部缓存写入FLASH，掉电保存。

### eeprom read

读eeprom：`eeprom read [地址] [长度]`，其中地址为必要参数，长度可不填，默认全部读取，从内部缓存读取数据。

读取全部EEPROM内容:

```
eeprom read 0

0x0000  0001 86A0 0000 0000 0001 86A0 FFE8 0000
0x0008  0001 86A0 0000 0000 0001 86A0 FFE8 0000
0x0010  0480 0000 00A0 0001 FFFF FFFF FFFF FFFF
0x0018  FFFF FFFF FFFF FFFF FFFF FFFF FFFF FFFF
0x0020  0000 0000 0000 0000 0008 0000 FFFF FFFF
0x0028  FFFF FFFF FFFF FFFF FFFF FFFF FFFF FFFF
0x0030  FFFF FFFF FFFF FFFF FFFF FFFF FFFF 12CA
0x0038  FFFF FFFF FFFF FFFF 2000 0800 275C
```

### eeprom write

写eeprom：`eeprom write [地址] [数据]`，其中地址和数据均为必要参数，数据写入内部缓存。

将0xABCD写入EEPROM地址0x39:

```
eeprom write 39 ABCD
 write 0xABCD to eeprom address 0x0039 ...
eeprom read 0

0x0000  0001 86A0 0000 0000 0001 86A0 FFE8 0000
0x0008  0001 86A0 0000 0000 0001 86A0 FFE8 0000
0x0010  0480 0000 00A0 0001 FFFF FFFF FFFF FFFF
0x0018  FFFF FFFF FFFF FFFF FFFF FFFF FFFF FFFF
0x0020  0000 0000 0000 0000 0008 0000 FFFF FFFF
0x0028  FFFF FFFF FFFF FFFF FFFF FFFF FFFF FFFF
0x0030  FFFF FFFF FFFF FFFF FFFF FFFF FFFF 12CA
0x0038  FFFF ABCD FFFF FFFF 2000 0800 275C
```

## flash

读写、擦除SPI FLASH。

命令格式：`flash [read|write|erase] [addr] [data] Operate SPI Flash.`

SPI FLASH支持读取、编程、擦除操作，写入之前需要先擦除，操作举例如下：

```
flash read 0
0x00000000 00 00 00 00 00 00 07 E6 00 00 00 00 00 00 00 01 ................
0x00000010 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x00000020 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0x00000030 00 00 00 00 00 00 00 00 00 00 00 00 00 00 DE 6B ...............k
flash erase 0
 erase sector 0x0000...
flash read 0 40
0x00000000 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
0x00000010 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
0x00000020 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
0x00000030 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
flash write 2 33
 write 0x33 to flash address 0x00000002 ...
flash read 0 40
0x00000000 FF FF 33 FF FF FF FF FF FF FF FF FF FF FF FF FF ..3.............
0x00000010 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
0x00000020 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
0x00000030 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF ................
```

### flash read

读取SPI FLASH。

命令格式`flash read [hex addr] [hex len]`。

带两个参数：起始地址和长度，长度如不指定默认256字节。

### flash write

编程SPI FLASH。

命令格式`flash write [hex addr] [hex data]`。

带两个参数：起始地址和数据，都需要指定。擦除过的地址才可以编程。

### flash erase

擦除SPI FLASH。

命令格式`flash erase [hex sector | chip]`。

带一个参数：扇区地址或者chip。

每个扇区大小为4kB，8MB共2048个扇区，因此扇区号取值0-7FF。

参数指定chip时执行全片擦除，擦除时间会比较长。

命令举例如下：

```
flash erase 1
 erase sector 0x0001...
flash erase chip
 erase chip done.
```

## param

操作用户参数。

命令格式：`param [load|save|restore] Operate parameters.`

param命令带三个子命令：load、save、restore。

`param load`命令从内置EEPROM加载保存的参数。

`param save`命令将参数保存到内置EEPROM。

`param restore`命令恢复默认参数。同时按住左右键上电也可以恢复默认参数。

## reboot

重启系统。

可以带一个延时参数，单位ms，如`reboot 900`延时900ms以后重启。

命令输出如下：
```
reboot
 rebooting ...
 UIMeterDual v19.6.19 SN:0D8004000657334339353420
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
reboot 900
 rebooting ...
 UIMeterDual v19.6.19 SN:0D8004000657334339353420
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```

## help

获取在线帮助。

命令输出如下：
```
help
 getui -> get voltage current and power etc.
 clear -> clear power and time Info.
 log -> log [dump|cha|chb|file|max|int|ring|auto|cross] Operate data logs.
 info -> info [baud|echo|bklt|lcd|time] Show/Set system info.
 adj -> adj [ua|ia|ub|ib] [adj_100000x] set sample gain adjustment.
 zero -> zero [ua|ia|ub|ib] [LSB] set sample zero adjustment.
 cali -> cali [ua|ia|ub|ib] [U_10000x|I_10000x] Voltage or Current calibration.
 eeprom -> eeprom [load|save|read|write] [addr] [data] Operate Int. EEPROM.
 flash -> flash [read|write|erase] [addr] [data] Operate SPI Flash.
 param -> param [load|save|restore] Operate parameters.
 reboot -> reboot [delay ms] Restart system.
 help -> help Info.
 version -> display SW version and SN.
```

## version

获取固件和设备序列号等信息。

命令输出如下：
```
version
 UIMeterDual v19.6.19 SN:0D8004000657334339353420
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
