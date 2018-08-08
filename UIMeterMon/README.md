# UIMeterMon操作说明

UIMeter目前支持二种通信协议：
1. TERM协议可以通过超级终端串口直接控制UIMeter。
2. MOSBUS协议为工业标准，可以使用组态软件编写监控工具。

鉴于很多用户对使用组态软件不习惯，UIMeter作者Echo使用VC编写了UIMeter专用上位机监控
软件：UIMeterMon，简称UIMM。UIMM主要功能为：
- 数据采集：将UIMeter采集到的数据存储到CSV文件，需要使用UIMeter硬件。
- 数据处理：对采集到的CSV数据进行分析处理，并且绘图，没有UIMeter硬件也可以使用，你甚至将
  其它软件的数据导成UIMM可以识别的CSV文件进行分析处理绘图

## 数据采集界面
数据采集界面如下所示：
 
![数据采集界面](image/01-数据采集界面.png "数据采集界面")

串口设置部分设置实际连接UIMeter的串口号，运行前先“打开串口”，然后勾上
“启动采集”，“实时数据”开始更新数据，勾上“记录数据”，可以把实时数据记录到CSV文件。

启动采集之前，勾上”快速采集”选项，UIMM将会以3Hz频率采样。

## 数据表现界面
使用UIMM打开CSV数据，点击“绘制曲线”按钮，可以绘制电压电流曲线：
 
![电压电流曲线](image/02-电压电流曲线.png "电压电流曲线")

使用鼠标可以对曲线进行局部放大：
 
![缩放以后的电压电流曲线](image/03-缩放以后的电压电流曲线.png "缩放以后的电压电流曲线")

同样可以查看温度曲线：
 
![温度曲线](image/04-温度曲线.png "温度曲线")

电量曲线：
 
![电量曲线](image/05-电量曲线.png "电量曲线")

## 操作视频
[UIMeterMon操作演示](http://v.youku.com/v_show/id_XODc2Nzc5NDI4.html?firsttime=72)

[红米5Plus充电数据分析](video/红米5Plus充电数据分析.wmv)

## 更多信息
[修订历史](ReleaseNotes.md)

[作者博客](http://blog.sina.com.cn/xjtuecho)

[作者微博](http://weibo.com/eth0)