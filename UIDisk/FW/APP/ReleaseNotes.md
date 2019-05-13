## 修订历史

### v19.3.1

- 增加128x32分辨率OLED显示功能，控制器SSD1306接口I2C。
- SDA:PA13(SWDIO)
- SCL:PA14(SWCLK)

### v18.5.23

- 发布CMSIS-DAP固件，需要配合UIDisk_SYS v18.5.23使用。
- SWCLK:PA9
- SWDIO:PA10
- nRESET:PA13

### v16.12.31
- 初始发布版本。支持UIMeter TERM协议下数据采集功能，数据格式与离线记录数据相同，
- 可以使用UIMeterMon软件进行分析绘图。