# 利用Github的Webhook功能和Node.js完成项目的自动部署
_本文对任何提供Webhook的git仓库都适用_

![2016-06-29_14635623250348.jpg](http://pic.mylonly.com/2016-06-29_14635623250348.jpg)

### 首先完成Node.js服务器的代码构建，先上代码，再解释
	
``` Node.js
var http = require('http')
var createHandler = require('github-webhook-handler')
var handler = createHandler({ path: '/', secret: 'root' })
// 上面的 secret 保持和 GitHub 后台设置的一致

function run_cmd(cmd, args, callback) {
  var spawn = require('child_process').spawn;
  var child = spawn(cmd, args);
  var resp = "";

  child.stdout.on('data', function(buffer) { resp += buffer.toString(); });
  child.stdout.on('end', function() { callback (resp) });
}

http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(7777)

handler.on('error', function (err) {
  console.error('Error:', err.message)
})

handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);
    run_cmd('sh', ['./deploy.sh',event.payload.repository.name], function(text){ console.log(text) });
})
```
上面的代码中用到了一个`github-webhook-handler`的中间价，你可以用`npm install -g github-webhook-handler`来全局安装

还有代码这行:
```
var handler = createHandler({ path: '/', secret: 'root' }) 
```
其中secret后的参数是你在github的项目中添加webhook时设置的secret值，替换成自己的就行了

### 完成deploy.sh脚本
deploy.sh脚本负责进入项目的目录，然后利用git命令拉取最新的代码，还是直接贴代码:

```Bash Shell
 #!/bin/bash

WEB_PATH='/root/tools/'$1
WEB_USER='root'
WEB_USERGROUP='root'

echo "Start deployment"
cd $WEB_PATH
echo "pulling source code..."
git reset --hard origin/master
git clean -f
git pull
git checkout master
echo "changing permissions..."
chown -R $WEB_USER:$WEB_USERGROUP $WEB_PATH
echo "Finished."
```
deploy.sh 会接受第一个参数当做项目名字，然后进入这个项目的目录执行git操作，这个参数是在deploy.js中根据hook返回的项目名字来的，代码应该比较容易懂，都是些简单的git命令。

> 如果是全新的项目，需要在你的服务器上先clone要部署的项目
> 你需要根据自己的实际项目位置，修改WEB_PATH的值

### 后台运行deploy.js
利用Linux提供的nohup命令，让deploy.js运行在后台

```
nohup node deploy.js > deploy.log &
```

### 去Github后台添加webhook
进入你需要自动部署的项目的github地址，进入项目的设置页面，点击左侧的`Webhooks & services`
![2016-06-29_14635620989191.jpg](http://pic.mylonly.com/2016-06-29_14635620989191.jpg)


