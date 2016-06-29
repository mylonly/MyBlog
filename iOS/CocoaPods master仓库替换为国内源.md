#CocoaPods master仓库替换为国内源
![2016-06-29_1396084357114.jpg](http://pic.mylonly.com/2016-06-29_1396084357114.jpg)
国内用CocoaPods 实在是太蛋疼了，一个pod update都要等好久，之前唐巧博客里面推荐的那个国内源已经不可用了，还好今天在V2EX上看到有人提供了别的CocoaPods源。

1. https://git.coding.net/hging/Specs.git
2. http://git.oschina.net/akuandev/Specs

执行如下命令替换Pods源

``` Bash Shell
$ pod repo remove master
$ pod repo add master 'http://git.oschina.net/akuandev/Specs.git' 
```
 
仅仅这样使用pod update时发现仍然会从一个master-1这个官方源中clone

``` Bash Shell
tianxianggendeMacBook-Air:upyun-batch-upload mylonly$ pod install
Creating shallow clone of spec repo `master-1` from `https://github.com/CocoaPods/Specs.git`
```
 
这时候需要在Podfile中加入source命令，就可以直接从国内源更新了。

``` Bash Shell
source 'http://git.oschina.net/akuandev/Specs.git'
platform :ios, '6.0'
pod 'AFNetworking', '~> 1.3.4'
```

