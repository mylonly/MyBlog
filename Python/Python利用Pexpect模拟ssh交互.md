# Python利用Pexpect模拟ssh交互

> ==关于Pexpect==
> Pexpect 是 Don Libes 的 Expect 语言的一个 Python 实现，是一个用来启动子程序，并使用正则表达式对程序输出做出特定响应，以此实现与其自动交互的 Python 模块。 Pexpect 的使用范围很广，可以用来实现与 ssh、ftp 、telnet 等程序的自动交互；可以用来自动复制软件安装包并在不同机器自动安装；还可以用来实现软件测试中与命令行交互的自动化。

#####本文利用到的Pexpect的类和方法

1. `spawn()`类:

	```Python
	class spawn:
	    def __init__(self,command,args=[],timeout=30,maxread=2000,\
	    searchwindowsize=None, logfile=None, cwd=None, env=None)
	```
	spawn是Pexpect模块主要的类，用以实现启动子程序，它有丰富的方法与子程序交互从而实现用户对子程序的控制。它主要使用 pty.fork() 生成子进程，并调用 exec() 系列函数执行 command 参数的内容。

2. `spawn()`类中的`expect()`函数:

	```Python
	expect(self, pattern, timeout=-1, searchwindowsize=None)
	```
	在参数中： pattern 可以是正则表达式， pexpect.EOF ， pexpect.TIMEOUT ，或者由这些元素组成的列表。需要注意的是，当 pattern 的类型是一个列表时，且子程序输出结果中不止一个被匹配成功，则匹配返回的结果是缓冲区中最先出现的那个元素，或者是列表中最左边的元素。使用 timeout 可以指定等待结果的超时时间 ，该时间以秒为单位。当超过预订时间时， expect 匹配到pexpect.TIMEOUT。

3. `spawn()`类中的`before`和`after`属性:
	
	expect 不断从读入缓冲区中匹配目标正则表达式，当匹配结束时 pexpect 的 before 成员中保存了缓冲区中匹配成功处之前的内容， pexpect 的 after 成员保存的是缓冲区中与目标正则表达式相匹配的内容。
	
	```Python
	child = pexpect.spawn('/bin/ls /') 
	child.expect (pexpect.EOF) 
	print child.before
	```
	以上代码就是打印在根目录下面执行ls命令后的输出内容
4.	 `spawn()`类中的send系列函数:
	
	```Python
	send(self, s) 
	sendline(self, s='') 
	sendcontrol(self, char)
	```
	这些方法用来向子程序发送命令，模拟输入命令的行为。 与 send() 不同的是 sendline() 会额外输入一个回车符 ，更加适合用来模拟对子程序进行输入命令的操作。 当需要模拟发送 “Ctrl+c” 的行为时，还可以使用 sendcontrol() 发送控制字符。
	
	```Python
	child.sendcontrol('c')
	
	```
	
#####功能模块分解

1. 首先我们需要一个可以单独的session会话，可以由connect函数创建指定host,username和password的会话子进程

		```Python
		PROMPT = ['#','$','>','\$','>>>']
		def createChildSession(host,username,password):
		    command = 'ssh '+username+'@'+host
		    child = pexpect.spawn(command)
		    ret = child.expect([pexpect.TIMEOUT,'Are you sure you want to continue connecting','[P|p]assword']+PROMPT)
		    if ret == 0:
		        print('[-] Error Connecting')
		        return
		    if ret == 1:
		        child.sendline('yes')
		        ret = child.expect([pexpect.TIMEOUT,'[p|P]assword'])
		        if ret == 0:
		            print('[-] Error Connecting')
		            return
		        if ret == 1:
		            send_command(password)
		            return
		    if ret == 2:
		        send_command(password)
		        return
		    return child
		```
		利用spawn创建会话之后,利用expect匹配可能存在的返回结果,如果匹配'Are you sure you want to continue connecting' 说明需要确认认证信息，如果直接返回password或者Password`这里利用[p|P]assword正则来匹配`,说明需要输入密码,如果直接是PROMPT中存在的字符，说明直接登录上去了。
	
	
2. 一个单独的执行命令的函数:
	
	```Python
	def send_command(child,cmd):
	    child.sendline(cmd)
	    child.expect(PROMPT)
	    print(child.before)
	```
	一旦通过验证,我们就可以用上面的command函数在ssh会话中发送命令，然后等待命令提示符的出现，最后将命令的执行结果通过child.before打印出来。

3. 一个包含参数解析的main函数:

	```Python
	def main():
	    parse = optparse.OptionParser('Usage %prog -H <host> -u <username> -p <password> -c <command>')
	    parse.add_option('-H',dest='host',type='string',help='specify the host')
	    parse.add_option('-u',dest='username',type='string',help='specify the username')
	    parse.add_option('-p',dest='password',type='string',help='specify the password')    
	    parse.add_option('-c',dest='command',type='string',help='specify the command')
	
	    (options,args)=parse.parse_args()
	    host = options.host
	    username = options.username
	    password = options.password
	    command = options.command
	    
	    session = createChildSession(host,username,password)
	    send_command(session,command)
	```
	optparse是一个用来给你的代码添加各种命令参数的库，用其解析出输入的host,username,password已经command,然后调用创建session会话，最后利用send_command向此session发送命令

```Bash Shell
tianxianggendeiMac:Python-Study Apple$ python ssh.py -H pi.****.com -u root -p ***** -c pwd
```
输出:

```Bash Shell
 pwd
/root
root@raspberrypi:~
```
	
#####完整代码

```Python
#!/usr/bin/python
#-*-coding:utf-8-*-
# date:2016-6-21
# author:root
# 利用pexpect模拟ssh登陆

import pexpect
import optparse

PROMPT = ['#','$','>','\$','>>>']

def send_command(child,cmd):
    child.sendline(cmd)
    child.expect(PROMPT)
    print(child.before)

def createChildSession(host,username,password):
    command = 'ssh '+username+'@'+host
    child = pexpect.spawn(command)
    ret = child.expect([pexpect.TIMEOUT,'Are you sure you want to continue connecting','[P|p]assword']+PROMPT)
    if ret == 0:
        print('[-] Error Connecting')
        return
    if ret == 1:
        child.sendline('yes')
        ret = child.expect([pexpect.TIMEOUT,'[p|P]assword'])
        if ret == 0:
            print('[-] Error Connecting')
            return
        if ret == 1:
            send_command(password)
            return
    if ret == 2:
        send_command(password)
        return
    return child

def main():
    parse = optparse.OptionParser('Usage %prog -H <host> -u <username> -p <password> -c <command>')
    parse.add_option('-H',dest='host',type='string',help='specify the host')
    parse.add_option('-u',dest='username',type='string',help='specify the username')
    parse.add_option('-p',dest='password',type='string',help='specify the password')    
    parse.add_option('-c',dest='command',type='string',help='specify the command')

    (options,args)=parse.parse_args()
    host = options.host
    username = options.username
    password = options.password
    command = options.command
    
    session = createChildSession(host,username,password)
    send_command(session,command)

if __name__ == '__main__':
    main()
```


