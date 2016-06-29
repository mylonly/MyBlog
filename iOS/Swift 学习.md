#Swift学习要点-基础部分
![2016-06-29_15:09:33.jpg](http://pic.mylonly.com/2016-06-29_15:09:33.jpg)
> 这一系列的文章都是我本人在学习Swift语法过程中认为需要注意的语法部分，所以介绍的并不会很完整。

1. print函数:`print(_:separator:terminator:)`,默认情况下print会以换行符当做结束符，此外你可以通过terminator参数更改结束符，例如:
	
	```Swift
	print("正常的print结尾时换行符")                  //输出:正常的print结尾时换行符\n
	print("没有换行符的print",terminator:"")         //输出:没有换行符的print
	print("稀奇古怪的结束符都可以",terminator:"(^^)")  //输出:稀奇古怪的结束符都可以(^^)
	```
2. 数值类字面量，包括整数和浮点数可以添加额外的零并且包含下划线，并不会影响字面量的值

	```Swift   
	let paddedDouble = 000123.456
	let oneMillion = 1_000_000
	let justOverOneMillion = 1_000_000.000_000_1
	```
	
3. 可以通过`typealias`关键字来定义类型的别名

	```Swift
	typealias AudioSample = UInt16
	var maxAmplitudeFound = AudioSample.min // maxAmplitudeFound 现在是 0
	```
	
4. 不同的数据类型之间不会隐式转换,必须要用类型名加上括号的方式进入显示转换,其中显示转换成字符串除了使用String()之外，还可以直接在字符串中适用\(其他类型值)来转换

	```Swift
	var maxValue = UInt8.max
	var max64Value = Int64.max
	
	max64Value = maxValue  //Error:Cannot assign value of type 'UInt8' to type 'Int64'
	max64Value = Int64(maxValue) //Ok

  let number = 3
  var str1 = String(number) //str1 = "3"
  var str2 = "\(number)"    //str2 = "3" 
	```
5. 如果你只需要一部分元组值，分解的时候可以把要忽略的部分用下划线（_）标记：
	
	```Swift
	let http404Error = (404, "Not Found")  // http404Error 的类型是 (Int, String)，值是 (404, "Not Found")
	let (justTheStatusCode, _) = http404Error
	print("The status code is \(justTheStatusCode)") // 输出 "The status code is 404"
	```
6. Swift 的`nil`和 Objective-C 中的`nil`并不一样。在 Objective-C 中，`nil`是一个指向不存在对象的指针。在 Swift 中，`nil`不是指针——它是一个确定的值，用来表示值缺失。任何类型的可选状态都可以被设置为`nil`，不只是对象类型。

	```Swift
	var serverResponseCode: Int? = 404
	// serverResponseCode 包含一个可选的 Int 值 404
	serverResponseCode = nil
	// serverResponseCode 现在不包含值
	```	

7. 使用!来强制解析值之前，一定要确定可选包含一个非nil的值，否则当强制解析一个nil值时会报错。

	```Swift
	var option:Int?    //没有初始值的可选类型变量默认值为nil
	var str = "\(option)" //输出:"nil"
	str = "\(option!)"    //Error
	```


