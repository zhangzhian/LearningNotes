
#[学习笔记]菜鸟教程Swift易忘知识点总结（二）

---

[CSDN链接](https://blog.csdn.net/baidu_32237719/article/details/82893443)
# 条件语句

1. if 语句
	```swift
	if boolean_expression {
	   /* 如果布尔表达式为真将执行的语句 */
	}
	```
2. if...else 语句

	```swift
	if boolean_expression {
	   /* 如果布尔表达式为真将执行的语句 */
	} else {
	   /* 如果布尔表达式为假将执行的语句 */
	}
	```
3. if...else if...else 语句
	```swift
	if boolean_expression_1 {
	   /* 如果 boolean_expression_1 表达式为 true 则执行该语句 */
	} else if boolean_expression_2 {
	   /* 如果 boolean_expression_2 表达式为 true 则执行该语句 */
	} else if boolean_expression_3 {
	   /* 如果 boolean_expression_3 表达式为 true 则执行该语句 */
	} else {
	   /* 如果以上所有条件表达式都不为 true 则执行该语句 */
	}
	```
4. switch 语句
	```swift 
	 switch expression {
	   case expression1  :
	      statement(s)
	      fallthrough /* 可选 */
	   case expression2, expression3  :
	      statement(s)
	      fallthrough /* 可选 */
	  
	   default : /* 可选 */
	      statement(s);
	}
	```

	如果没有使用 fallthrough 语句，则在执行当前的 case 语句后，switch 会终止，控制流将跳转到 switch 语句后的下一行。
	**如果使用了fallthrough 语句，则会继续执行之后的 case 或 default 语句，不论条件是否满足都会执行。**

5. ? : 运算符

   条件运算符 ? :，可以用来替代 if...else 语句:` Exp1 ? Exp2 : Exp3`

#  循环

1.  for-in 循环
	```swift
	for index in var {
	   循环体
	}
	```
2. While 循环
	```swift
	while condition
	{
	   statement(s)
	}
	```
3. repeat...while 循环:开始执行前先判断条件语句，而是在循环执行结束时判断条件是否符合。
	```swift
	repeat
	{
	   statement(s);
	}while( condition );
	```
4. 循环控制语句

	| 控制语句 | 描述 |
	|--|--|
	|  continue 语句|告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代。  |
	| break 语句| 中断当前循环。| 
	| fallthrough 语句| 如果在一个case执行完后，继续执行下面的case，需要使用fallthrough(贯穿)关键字。| 

# 字符串

1. 创建字符串
通过使用字符串字面量或 String 类的实例来创建一个字符串：

	```swift
	// 使用字符串字面量
	var stringA = "Hello, World!"
	print( stringA )
	
	// String 实例化
	var stringB = String("Hello, World!")
	print( stringB )
	```

2. 字符串中插入值
字符串插值包含常量、变量、字面量和表达式。 插入的字符串字面量的每一项都在以反斜线为前缀的圆括号中：

	```swift
	var varA   = 20
	let constA = 100
	var varC:Float = 20.0
	
	var stringA = "\(varA) 乘于 \(constA) 等于 \(varC * 100)"
	print( stringA )
	```
3. 字符串长度:使用 String.count 属性来计算
	```swift
	var varA   = "www.runoob.com"	
	print( "\(varA), 长度为 \(varA.count)" )
	```
4. Unicode 字符串

	```swift
	var unicodeString   = "菜鸟教程"
	print("UTF-8 编码: ")
	for code in unicodeString.utf8 {
	   print("\(code) ")
	}
	print("UTF-16 编码: ")
	for code in unicodeString.utf16 {
	   print("\(code) ")
	}
	```
# 字符
1. 空字符变量:Swift 中不能创建空的 Character（字符） 类型变量或常量

2. 遍历字符串中的字符:
  可通过for-in循环来遍历字符串中的characters属性来获取每一个字符的值：

	```swift
	for ch in "Runoob".characters {
	    print(ch)
	}
	```
3. 字符串连接字符: String 的 append() 方法

	```swift
	var varA:String = "Hello "
	let varB:Character = "G"
	
	varA.append( varB )
	
	print("varC  =  \(varA)")
	```
# 数组
1. 创建数组
可以使用构造语法来创建一个由特定数据类型构成的空数组：
`var someArray = [SomeType]()`
以下是创建一个初始化大小数组的语法：
`var someArray = [SomeType](repeating: InitialValue, count: NumbeOfElements)`
	```swift
	//以下实例创建了一个类型为 Int ，数量为 3，初始值为 0 的空数组：
	var someInts = [Int](repeating: 0, count: 3)
	//以下实例创建了含有三个元素的数组：
	var someInts:[Int] = [10, 20, 30]
	```
2. 访问数组
`var someVar = someArray[index]`

3. 修改数组
可以使用 append() 方法或者赋值运算符 += 在数组末尾添加元素

	```swift
	var someInts = [Int]()
	
	someInts.append(20)
	someInts.append(30)
	someInts += [40]
	```
    可以通过索引修改数组元素的值：

	```swift
	// 修改最后一个元素
	someInts[2] = 50
	```

4. 遍历数组
可以使用for-in循环来遍历所有数组中的数据项
	```swift
	for item in someStrs {
	   print(item)
	}
	```
	每个数据项的值和索引值，可以使用 String 的 enumerate() 方法来进行数组遍历。
	```swift
	for (index, item) in someStrs.enumerated() {
	    print("在 index = \(index) 位置上的值为 \(item)")
	}
	```
5. 合并数组:以使用加法操作符（+）来合并两种已存在的相同类型数组。
	```swift
	var intsA = [Int](repeating: 2, count:2)
	var intsB = [Int](repeating: 1, count:3)
	
	var intsC = intsA + intsB
	
	for item in intsC {
	    print(item)
	}
	```

6. count 属性
我们可以使用 count 属性来计算数组元素个数：
7. isEmpty 属性
我们可以通过只读属性 isEmpty 来判断数组是否为空，返回布尔值

# 字典
1. Swift 字典的key没有类型限制可以是整型或字符串，但必须是唯一的。
   创建一个字典，并赋值给一个变量，则创建的字典就是可以修改的。将一个字典赋值给常量，字典就不可修改，并且字典的大小和内容都不可以修改。
2. 创建字典

	创建一个特定类型的空字典：
	`var someDict =  [KeyType: ValueType]()`
	
	```swift
	//以下是创建一个空字典，键的类型为 Int，值的类型为 String 的简单语法：
	var someDict = [Int: String]()
	//以下为创建一个字典的实例：
	var someDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]
	```
3. 访问字典`var someVar = someDict[key]`
4. 修改字典
 `updateValue(forKey:) `增加或更新字典的内容。如果 key 不存在，则添加值，如果存在则修改 key 对应的值。updateValue(_:forKey:)方法返回Optional值。
	```swift
	var someDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]
	
	var oldVal = someDict.updateValue("One 新的值", forKey: 1)
	```
   通过指定的 key 来修改字典的值
	```swift
	var someDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]
	
	var oldVal = someDict[1]
	someDict[1] = "One 新的值"
	```
5. 移除 Key-Value 对
可以使用 removeValueForKey() 方法来移除字典 key-value 对。如果 key 存在该方法返回移除的值，如果不存在返回 nil 。
	```swift
	var someDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]
	
	var removedValue = someDict.removeValue(forKey: 2)
	```
   可以通过指定键的值为 nil 来移除 key-value（键-值）对。
	```swift
	someDict[2] = nil
	```
6. 遍历字典
	使用 for-in 循环来遍历某个字典中的键值对
	```swift
	for (key, value) in someDict {
	   print("字典 key \(key) -  字典 value \(value)")
	}
	```
	可以使用enumerate()方法来进行字典遍历，返回的是字典的索引及 (key, value) 对
	```swift
	for (key, value) in someDict.enumerated() {
	    print("字典 key \(key) -  字典 (key, value) 对 \(value)")
	}
	```
7. 字典转换为数组

	```swift
	var someDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]
	
	let dictKeys = [Int](someDict.keys)
	let dictValues = [String](someDict.values)
	
	print("输出字典的键(key)")
	for (key) in dictKeys {
	    print("\(key)")
	}
	
	print("输出字典的值(value)")
	for (value) in dictValues {
	    print("\(value)")
	}
	```
8. count 属性
可以使用只读的 count 属性来计算字典有多少个键值对

9. isEmpty 属性
可以通过只读属性 isEmpty 来判断字典是否为空，返回布尔值

