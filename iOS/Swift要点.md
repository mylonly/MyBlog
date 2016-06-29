# Swift要点
(本文所有代码样例全部来自Swift2.0官方文档)

1. Swift类型间不会隐式转换，必须要显式转换。将值转换成字符串除了使用String()显式转换外，还有中简单的方法,如下:

	``` Swift
	let apples = 3
	let oranges = 5
	let appleSummary = "I have \(apples) apples."
	let fruitSummary = "I have \(apples + oranges) 	pieces of fruit."
	```
	输出:
	
	```
	"I have 3 apples"
	"I have 8 pieces of fruit"
	```
2. Swift声明数组或者字典可以声明指定类型

	``` Swift
	let emptyArray = [String]()
	let emptyDictionary = [String: Float]()
	```
3. if语句中，条件必须是一个布尔表达式,如下:

	```Swift
	 if score > 50 {
        teamScore += 3
    } else {
    	   teamScore += 1
    }
	```
	`if 1 {}` 这样代码会报错，但是`if true{}`这样的代码是可	以的。
4. Switch 支持任意类型的数据以及各种比较，不仅仅限于整数以及判断是否相等,而且Switch匹配到相应的子句之后就会推出整个Switch,不需要给每个Switch子句写上break了。

	```Swift
	let vegetable = "red pepper"
	switch vegetable {
	case "celery":
    	print("Add some raisins and make ants on a 	log.")
	case "cucumber", "watercress":
	    print("That would make a good tea sandwich.")
	case let x where x.hasSuffix("pepper"):
	    print("Is it a spicy \(x)?")
	default:
	    print("Everything tastes good in soup.")
	}
	```
	*Swift 子句中必须要遍历所有可能，否则会报错*
	*上述代码中的lex表达式将匹配等式的值赋给变量x*
5. do{}while()被repeat{}while()取代

	```Swift
	var m = 2
	repeat {
	    m = m * 2
	} while m < 100
	print(m)
	```
6. 循环有更简便的写法,0..<4表示遍历0到4(不包含4，包含4用0...4),传统写法也是支持的。

	```Swift
	var firstForLoop = 0
	for i in 0..<4 {
	    firstForLoop += i
	}
	print(firstForLoop)
	
	
	var firstForLoop = 0
	for i in 0...4 {
	    firstForLoop += i
	}
	print(firstForLoop)
	```
7. 函数可以传入可变的参数，参数在函数内表现为数组形式:
	```Swift
	func sumOf(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
	}
	sumOf()
	sumOf(42, 597, 12)
	```
8. 函数可以作为另一个函数的返回值，类似于OC中的block

	```Swift
	func makeIncrementer() -> (Int -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
	}
	var increment = makeIncrementer()
	increment(7)
	```
	
	同理，函数也可以当做参数传入函数,
	
	```Swift
	func hasAnyMatches(list: [Int], condition: Int -> 	Bool) -> Bool {
    for item in list {
        if condition(item) {
            return true
        }
    }
    return false
	}
	func lessThanTen(number: Int) -> Bool {
	    return number < 10
	}
	var numbers = [20, 19, 7, 12]
	hasAnyMatches(numbers, condition: lessThanTen)
	```
9. 子类中，如果需要重写父类的方法，需要使用override标记

	```Swift
	class NamedShape {
    var numberOfSides: Int = 0
    var name: String

    init(name: String) {
        self.name = name
    }

    func simpleDescription() -> String {
	        return "A shape with \(numberOfSides) sides."
	  }
	}
	
	class Square: NamedShape {
    var sideLength: Double

    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 4
    }

    func area() ->  Double {
        return sideLength * sideLength
    }

    override func simpleDescription() -> String {
	        return "A square with sides of length \(sideLength)."
	    }
	}
	let test = Square(sideLength: 5.2, name: "my test 	square")
	test.area()
	test.simpleDescription()
	```


