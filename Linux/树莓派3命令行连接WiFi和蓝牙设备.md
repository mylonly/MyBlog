# 树莓派3命令行连接WiFi和蓝牙设备
转载[原文链接](https://www.embbnux.com/2016/04/10/raspberry_pi_3_wifi_and_bluetooth_setting_on_console/)

![2016-06-29_14634631824044.jpg](http://pic.mylonly.com/2016-06-29_14634631824044.jpg)

###WiFi连接

```
pi@raspi3:~ $ iwlist scan
wlan0 Scan completed :
Cell 01 - Address: 00:1E:20:50:AA:BB
Channel:8
Frequency:2.447 GHz (Channel 8)
Quality=70/70 Signal level=-32 dBm
Encryption key:on
ESSID:"WIFINAME"
Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
9 Mb/s; 12 Mb/s; 18 Mb/s
Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
Mode:Master
Extra:tsf=0000000000000000
Extra: Last beacon: 2157000ms ago
IE: Unknown: 000546616E6379
IE: Unknown: 010882848B960C121824
IE: Unknown: 030108
IE: Unknown: 050401020000
IE: Unknown: 0706303020010B14
IE: Unknown: 2A0100
IE: Unknown: 32043048606C
IE: IEEE 802.11i/WPA2 Version 1
Group Cipher : TKIP
Pairwise Ciphers (2) : CCMP TKIP
Authentication Suites (1) : PSK
IE: Unknown: 7F080000000000000040
IE: Unknown: DD180050F2020101000003A4000027A4000042435E0062322F00
 
可以看到周围的wifi热点信息
配置连接到某个热点:
 
# 编辑wifi文件
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
# 在该文件最后添加下面的话
network={
  ssid="WIFINAME"
  psk="password"
}
# 引号部分分别为wifi的名字和密码
# 保存文件后几秒钟应该就会自动连接到该wifi
# 查看是否连接成功
ifconfig wlan0

```

### 蓝牙连接

``` 
pi@raspi3:~ $ sudo bluetoothctl
[NEW] Controller BB:27:EB:0D:9D:DD raspi3 [default]
[bluetooth]# list
Controller BB:27:EB:0D:9D:DD raspi3 [default]
[bluetooth]# power on
Changing power on succeeded
[bluetooth]# scan on
Discovery started
[CHG] Controller BB:27:EB:0D:9D:DD Discovering: yes
[NEW] Device E8:07:BF:3A:25:AA NDZ-03-GA
[CHG] Device E8:07:BF:3A:25:AA RSSI: -66
[bluetooth]# agent on
Agent registered
[CHG] Device E8:07:BF:3A:25:AA RSSI: -56
[bluetooth]# pair E8:07:BF:3A:25:AA
Attempting to pair with E8:07:BF:3A:25:AA
[CHG] Device E8:07:BF:3A:25:AA Connected: yes
[CHG] Device E8:07:BF:3A:25:AA UUIDs:
	00001108-0000-1000-8000-00805f9b34ff
[CHG] Device E8:07:BF:3A:25:AA Paired: yes
Pairing successful
[CHG] Device E8:07:BF:3A:25:AA Connected: no
[bluetooth]# trust E8:07:BF:3A:25:AA
[CHG] Device E8:07:BF:3A:25:AA Trusted: yes
Changing E8:07:BF:3A:25:AA trust succeeded
[bluetooth]# connect E8:07:BF:3A:25:AA

```

这样就连上蓝牙设备了，如果是蓝牙音响的话还得装下支持软件:
 ```
sudo apt-get install pulseaudio pulseaudio-module-bluetooth
```


