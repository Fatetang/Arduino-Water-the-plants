## 一款迷你型的植物浇水系统，包括代码、电路板原理图PCB图和适用3D打印的外壳
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/v8.0-003.jpg)
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/v8.0-002.jpg)
#### 注意！所有版本PCB未加到电池低压保护，电池过放会导致电池寿命下降，请使用带保护板的电池，不要让电池电压低于3.0V。
#### 2020-07-19 将土壤湿度的电路集成到主控板上，外接自制的土壤湿度探测器。重新设计了盒子。
#### 2020-04-19,程序更新V1.1.8，稳定版
* 修复1.1.5-1.1.7的休眠不唤醒BUG
* 修复了L9110驱动芯片使用PWM死机的BUG，只限D版本，C版本暂未修复
* 具体做法为修改pin10引脚的pwm输出模式为相位和频率校正模式，频率为31250HZ
* 修复土壤湿度传感器初始化太久会自动重启的BUG
* 修改唤醒策略只有水泵auto模式下才会唤醒，水泵off模式下不会唤醒，永久休眠，直至电池没电

#### 2020-04-15,程序更新V1.1.6
* 将土壤湿度数值改为百分比模式 
* 更换看着顺眼的字体
* 加入看门狗超时复位，以免系统卡死导致电机不会停止
* 增加电池低压保护，低于等于3V，水泵和传感器断电，并提示 
* 电池低压时休眠时间强制设为24小时，下一次唤醒恢复原来设置
* 唤醒界面加入电机超时提示的次数
* L9110芯片对大功率电机使用平滑启动功能有几率使系统卡死，慎用（1.2A的电机）

#### 2020-03-20，超小尺寸版本V7.1正式发布<br>
* 添加100uf电容等多个滤波电容至5V电路，增加电流供应能力和稳定性，防止电机启动对电路造成过多的压降和干扰
* 添加一组π形滤波器至ADCVCC，增强ADC采样的抗干扰能力。
* 电压采样引脚串联10K电阻，以防sx1308升压芯片损坏时烧毁单片机
* 欲更换电机驱动失败，现还是L9110
* 欲增加低于3.0V时关断sx1306的功能失败，所以电池低压保护未成功
* 注意！从旧版本升级的需要重新刷入EERPOM,即烧录两次程序才可使用。
* 优化代码结构,去掉多余的代码，提升运行效率，添加更多注释
* 修复设置界面的参数会保存两次EEPROM的BUG
* 主界面“浇水”上面的英文提示由ON换成AUTO
* 设置界面的电压校准改为每次加减0.01V
* 修改设置界面的数值排序

## 特点
* 根据土壤的湿度进行浇水，而不是简单定时浇水，避免不必要的浇水，更加科学的管理植物。
* 可自动休眠，节省电量，并内置750mah电池（5.0），满足数周充一次电（小型花盆），配合太阳能充电器可实现长期无人监管。
* 丰富的设置选项，应对不同的植物。
* 两种电机驱动芯片选择，L9110和DRV8832。
* L9110支持pwm调节水泵速度，带热断电保护。
* DRV8832不支持手动调节水泵速度，芯片会自己调节到合适工作电压，带水泵短路、堵转监控功能。
* 土壤较松的话不适合垂直放置湿度传感器，建议水平放置在花盆底部
* 2种不同的PCB结构，5.0带16340电池座，7.0仅引出电池接口但板子更小。
* V5.1带电池座、制作简单。V7.1高度集成、体积最小、制作难度中等。
* 观看视频 https://space.bilibili.com/16758526/favlist?fid=769297826&ftype=create
* 3D打印件下载 https://www.thingiverse.com/thing:4025691
## 烧录说明
<br>1.使用ArduinoIDE编译上传，需要下载两次程序才可以使用在setup()找到这段程序看说明下载程序。EEPROM.put第一次写入去掉注释，第二次以后注释上，EEPROM.get第一次写入注释上，第二次以后写入去掉注释
<br>2.必须将arduino pro mini的电源指示LED和LED旁边的限流电阻焊下来，否则电量会很快耗尽，若只焊下LED没焊下电阻就读取电压会烧坏板子！

## 原理<br>
1.基本原理：浇水，休眠，唤醒，判断土壤湿度是否达到设置的值，没达到继续休眠，达到就开始浇水，浇到设定的值就休眠，不断循环。比市面上的定时浇水多了一个土壤湿度检测功能，不再是盲目的浇水。<br><br>
2.休眠时电流低至0.8ma，水泵接口输出电压5v电流800ma，可根据需求选用水泵。<br><br>
3.上电前插上电容式土壤湿度传感器，当然没插也可以开机但会报错。<br>
每次上电需要手动开启浇水功能（主界面第三项），开启浇水功能不代表水泵就会运转，水泵运转要达到浇水下限的值。<br><br>
4.写有传感器和水泵的保护代码，一旦传感器或水泵故障即可触发保护机制，让浇水系统停止工作，并显示故障原因。<br>
传感器保护机制就比较简单，若传感器没工作，读取传感器数值的引脚就会受到干扰，数值会变得非常大，只要判断数值超过一定值就触发保护机制。<br>
水泵保护则是使用水泵超时时间来设定，若输出接口打开，10秒后记录当前的土壤湿度值，过一段时间在将旧的土壤湿度值跟现在的对比,若变化小于5，即会触发保护使浇水系统停止，即可判断水泵没接或水泵没工作或水管没插到土里。<br><br>

## 主要功能介绍<br>
1.自定义浇水上限和下限<br>
2.自定义浇水系统休眠时间<br>
3.水泵短路堵转保护和传感器异常保护<br>
4.强制关闭传感器<br>
5.强制开启关闭浇水功能<br>
6.自定义水泵超时保护时间<br>
7.驱动芯片过热停机<br>
8.读取电池、电压校准<br>
9.电池充电时的动画<br>
10.水泵运转时间显示<br>
11.水泵运转时的动画<br>
12.自定义屏幕亮度<br>
13.数据断电保存 <br><br>
A.自定义浇水上下限：浇水的下限就是土壤干到什么程度才开始浇水，上限就是土壤要浇到多湿才停止。<br><br>
B.自定义休眠时间：没有浇水任务会自动休眠,休眠能大大节省电量。<br><br>
C.水泵超时保护：水泵超时时间内土壤湿度是否有变化，如设定60即是每隔60秒内土壤湿度变化需要超过3%，否者会触发保护机制，以免没浇到水而无止境的运转水泵而使电量耗尽。<br><br>
D.水泵短路堵转保护和传感器异常保护：拔出传感器停止浇水功能、水泵短路堵转检测（只限DRV8832）<br><br>
E.电池电压校准：电池电压数值校准，因为每块电路板的基准电压都不会是5.00V，有时候会高低那么零点零几伏所以需要手动校准，使用万用表测量电池的电压进行对比校准。<br><br>
F.自定义屏幕亮度：主界面下长上下按键可以调节OELD的亮度。<br><br>
G.水泵运转时和充电时会有动画提示。<br><br>
## 驱动芯片说明
#### 请使用带保护板的锂离子电池，不管哪种芯片，电池的放电电流需要大于水泵的峰值电流，不然会掉电重启。
### L9110
* 支持普通直流有刷水泵
* 支持带控制板的无刷水泵
* 支持PWM输入
* 芯片自带热断保护，控制器不会提示但会断电重启
* 最大电流1000ma，使用超过1A电流的水泵会掉电重启。
### DRV8832
* 支持普通直流有刷水泵
* 不支持带控制板的无刷水泵
* 不支持PWM输入
* 芯片带短路和堵转保护，控制器会提示不会断电重启
* 最大电流1000ma，使用超过1A电流的水泵会被芯片自动限流，任然可以运转。


## 引脚定义<br>
6-电机驱动输入引脚1<br>
5-电机驱动输入引脚2<br>
12-传感器电源正极<br>
13-led指示灯（暂时未启用）<br>
2-下按键<br>
3-上按键<br>
4-确认按键<br>
A3-读取电压的引脚<br>
A0-读取土壤湿度的引脚<br>
A2-读取充电状态的引脚<br>
A4-OLED,SDA<br>
A5-OLED,SCL<br>

## 程序更新说明<br>
    v1.1.80
    a.修复1.1.5-1.1.7的休眠不唤醒BUG
    b.修复了L9110驱动芯片使用PWM死机的BUG
      具体做法为修改pin10引脚的pwm输出模式为相位和频率校正模式，频率为31250HZ
    c.修复土壤湿度传感器初始化太久会自动重启的BUG
    d.修改唤醒策略只有水泵auto模式下才会唤醒，水泵off模式下不会唤醒，永久休眠，直至电池没电
    v1.1.71-D版本
    a.修复1.1.5-1.1.7的休眠不唤醒BUG
    b.C版本暂未修复L9110使用pwm死机问题，请谨慎使用PWM
     具体做法为修改pin10引脚的pwm输出模式为相位和频率校正模式，频率为31250HZ
    c.修复土壤湿度传感器初始化太久会自动重启的BUG
    d.修改唤醒策略只有水泵auto模式下才会唤醒，水泵off模式下不会唤醒，永久休眠，直至电池没电
    v1.1.70
    修改上下限每次步进数为2%
    缩短看门狗溢出时间为2秒，即系统卡死会在4-2秒内复位
    增加软复位，先按下“下键”不放再按“确认键”即可软复位
    修复休眠时间太大时休眠不了的BUG
    新增电机堵转短路保护，仅限drv8832版本（程序代号E）
    修改充电指示的判断程序
    v1.1.6(需更新u8g2_fonts.c)
    将土壤湿度数值改为百分比模式
    v1.1.5(需更新u8g2_fonts.c)
    更换看着顺眼的字体
    加入看门狗超时复位，以免系统卡死导致电机不会停止
    增加电池低压保护，低于3V，水泵和传感器断电，并提示
    低压时休眠时间强制设为24小时
    L9110水泵平滑启动对大功率电机有几率使系统卡死，慎用
    v1.1.4
    修复充电误判的BUG
    新增真正的“永不休眠”模式，休眠时间为0开启
    v1.1.3(需重刷eeprom)
    a.水泵浇水模式改为 永久关闭-自动运行-强制启动 3个模式
    主界面的“浇水”按下为强制启动，只有OFF下有效
    主界面上的“AUTO/OFF”可以选择，按下为“自动模式”“永久关闭”之间切换
    注意，浇水模式可断电保存，所以需要重新刷入eeprom
    修改模式不会立即保存到eeprom，休眠时才会保存到eeprom
    b.再次优化唤醒时检测土壤湿度传感器的算法，修复唤醒有几率会误判的BUG
    c.更换内存占用更小的字体，为后续升级节省空间
    d.更换获取数字位数的算法
    v1.1.2
    a.美化设置界面ui
    土壤传感器  开
    水泵接口    R+L-
    b.美化按调节亮度的ui(主界面长按加减键)
    c.更快的检测传感器是否拔出
    v1.1.1
    改善唤醒时读取土壤湿度的稳定性
    v1.1.0
    将动态内存从90%减少至55%
    更换选框样式
    v1.0.1(需重刷eeprom)
    注意！从旧版本升级的需要重新刷入EERPOM,即烧录两次程序才可使用。
    优化代码结构,去掉多余的代码，提升运行效率，添加更多注释
    修复设置界面的参数会保存两次EEPROM的BUG
    主界面“浇水”上面的英文提示由ON换成AUTO
    设置界面的电压校准改为每次加减0.01V
    修改设置界面的数值排序
![v7.0](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/v7.0.png)
![v7.1](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/V7.1.jpg)
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/js4056-3.jpg)
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/js4056-4.jpg)
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/js4056-5.jpg)
![](https://github.com/jie326513988/Arduino-Water-the-plants/blob/master/picture/js4056-6.jpg)
