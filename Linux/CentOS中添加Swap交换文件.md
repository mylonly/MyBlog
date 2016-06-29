# CentOS中添加Swap交换文件
1.先检查是否存在Swap文件

```Bash Shell
swapon -s #返回的信息为空，说明没有其他交换文件
```
2.创建一个Swap文件，使用dd命令

```Bash Shell
##of后面是swap文件的路径，可以自定义，count是swap文件的大小，推荐为内存的2倍
dd if=/dev/zero of=/home/swap bs=1024 count=1024000
```
3.将swap文件转换成swap格式

```Bash Shell
mkswap /home/swap
```
4.将swap文件挂载成swap分区

```Bash Shell
swapon /home/swap #创建完成后可以用`free -m`看看是否成功
```
5.让swap文件自动挂载

```Bash Shell
vim /etc/fstab 
#在文件末尾加上下面的命令
/home/swap swap swap default 0 0 
```


