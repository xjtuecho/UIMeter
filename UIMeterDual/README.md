UIMeterDual为UIMeter的双通道版本，继承了UIMeter强大的电压电流监控记录功能，
测试通道增加到2路，前端采样采用德州仪器INA226方案，可以方便测试DCDC电路效率。

主要特性如下：

- 测量范围宽，电压高达36V，电流正负10A，两个通道。
- 使用8MB FLASH记录离线数据，支持`16384*8`条离线数据记录与导出。
- 独特的串口命令行界面，可以查看设置各种参数。
- 电流支持高端、低端测量，检流电阻共模电压高达36V，支持真四线测量。
- 可外接蓝牙模块，支持无线联机。
- 电压电流全软件校准，提供校准命令。

## 资料汇总：

- UIMeterDual用户手册PDF下载：[UIMeterDual_UserManual](DOC/UIMeterDual_UserManual_v19.6.18.pdf)
- UIMeterDual命令行手册PDF下载：[UIMeterDual_CmdRef](DOC/UIMeterDual_CmdRef_v19.6.19.pdf)
- UIMeterDual命令行手册在线阅读：[UIMeterDual_CmdRef](DOC/UIMeterDual_CmdRef.md)
- UIMeterDual固件发布日志：[ReleaseNotes](FW/ReleaseNotes.md)
- UIMeterDual固件下载：[UIMeterDual](FW/UIMeterDual_v19.6.19.hex)
