# Linux常用命令
1. 磁盘挂载

```
fdisk -l #查看磁盘信息
mount /dev/xvdb1 /mnt #挂载磁盘
```
2. 设置开机自动加载磁盘

```
vim /etc/fstab
写入 /dev/xvdb1 /mnt ext4 default 1 1
```
3. 修改主机名

```
vim /etc/sysconfig/network #修改里面的HOSTNAME值
```

4. 设置ssh自动认证

```
ssh-keygen -t rsa #在客户机生成秘钥，
scp ~/.ssh/id_rsa.pub root@xxx.com:/home/xxx/ #将客户端生成的公钥文件发送到服务器上
#将id_rsa.pub文件写入服务器的.ssh/authorized_keys中，
最好用cat命令写入,手动创建authorized_keys文件会出现各种各样的权限认证问题
cat id_rsa.pub >> ~/.ssh/authorized_keys
```



