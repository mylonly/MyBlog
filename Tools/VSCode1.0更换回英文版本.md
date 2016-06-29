# VSCode1.0更换回英文版本
VSCode1.0更新之后带来了本地语言的支持，但是，你怎么能连命令行的命令都汉化了？
![](http://pic.mylonly.com/2016-05-04-14623468922061.jpg)
`表示实在受不了每次同步代码 选中git:拉的感觉，还有安装插件的命令也不再是install extension了，而是安装扩展`

###将VSCode的语言修改为英文
[官方文档设置语言](https://code.visualstudio.com/docs/customization/locales)

快捷键Command+Shift+P（Win下为Control）打开命令行工具,输入`设置语言`，会打开一个locale.json的文件，如下面所示

```
{
	// 定义 VSCode 的显示语言。
	// 请参阅 http://go.microsoft.com/fwlink/?LinkId=761051，了解支持的语言列表。
	// 要更改值需要重启 VSCode。
	"locale":"zh-CN" 
}
```
将locale的值改为en-US之后重启VSCode就恢复到英文版本的了!


