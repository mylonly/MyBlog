# 树莓派3安装Node.js
(树莓派官方系统)
![2016-06-29_14634631824044.jpg](http://pic.mylonly.com/2016-06-29_14634631824044.jpg)
```Bash Shell
wget https://nodejs.org/dist/v5.2.0/node-v5.2.0-linux-armv7l.tar.gz
tar zxvf node-v5.2.0-linux-armv7l.tar.gz
cd node-v5.2.0-linux-armv7l
sudo cp bin/* /usr/bin/ -r
sudo cp include/ /usr/include/ -r
sudo cp lib/ /usr/lib/ -r
sudo cp share/ /usr/share/ -r
cd ..
node --version
```


