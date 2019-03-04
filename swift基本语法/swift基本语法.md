[swift编程语言参考](https://www.cnswift.org)


## 一 基本内容

### 1 常量和变量

+ 使用关键字 let 来声明常量
+ 使用关键字 var 来声明变量
+ 你可以在一行中声明多个变量或常量，用逗号分隔

### 2 类型标注

+ 在变量或常量的名字后边加一个冒号，再跟一个空格，最后加上要使用的类型名称

  ```swift
  var welcomeMessage: String = "str"
  
  必须要赋值，这样才不会报错
  ```


### 3 输出常量和变量

+ print(_:separator:terminator:) 

  ```swift
  分隔符: separator
  print(1.0, 2.0, 3.0, 4.0, 5.0, separator: " ... ")
  // Prints "1.0 ... 2.0 ... 3.0 ... 4.0 ... 5.0"
  
  终止符:terminator 默认是换行符
  for n in 1...5 {
      print(n, terminator: "")
  }
  // Prints "12345"
  ```


### 4 分号

+ Swift 并不要求你在每一句代码结尾写分号（ ; ）
+ 如果你想在一行里写多句代码，分号还*是*需要的。



### 5 整数

+ Int, Int32, Int64, UInt,  UInt8, UInt32, UInt64等

+ 你可以通过 min 和 max 属性来访问每个整数类型的最小值和最大值

  + `let minValue = UInt8.min`, ` let maxValue = UInt8.max `

+ 在32位平台上， Int 的长度和 Int32 相同, 在64位平台上， Int 的长度和 Int64 相同。

+ 在32位平台上， UInt 长度和 UInt32 长度相同。在64位平台上， UInt 长度和 UInt64 长度相同。

+ 整数转换: 不同整数的类型在变量和常量中存储的数字范围是不同的,要注意转换是否错误。

   ```swift
    let twoThousand: UInt16 = 2_000
    let one: UInt8 = 1
    let twoThousandAndOne = twoThousand + UInt16(one) //twoThousandAndOne为UInt16
   ```

  

### 6 浮点数

+ Double代表 64 位的浮点数。
+ Float 代表 32 位的浮点数。
+ Double 有至少 15 位数字的精度，而 Float 的精度只有 6 位

### 7 类型安全和类型推断

+ Swift 是类型安全的，它在编译代码的时候会进行类型检查，任何不匹配的类型都会被标记为错误
+ 通过检查你给变量赋的值，类型推断能够在编译阶段自动的推断出值的类型
  + 注意: Swift 在推断浮点值的时候始终会选择 Double （而不是 Float ）

### 8 数值型字面量

- 一个十进制数，没有前缀 `let decimalInteger = 17`
- 一个二进制数，前缀是 0b `let binaryInteger = 0b10001`
- 一个八进制数，前缀是 0o `let octalInteger = 0o21`
- 一个十六进制数，前缀是 0x  `let hexadecimalInteger = 0x11`
- 十进制数与 exp  的指数,`1.25e2 意味着 1.25 x 102(2位于右上角), 或者 125.0`
- 十六进制数与 exp 指数, `0xFp2  意味着 15 x 22(2位于右上角), 或者 60.0 .`

### 9 整数和浮点数转换

+ 整数转换为浮点数类型必须显式地指定类型

  ```swift
  let three = 3
  let pointOneFourOneFiveNine = 0.14159
  let pi = Double(three) + pointOneFourOneFiveNine
  // pi equals 3.14159, and is inferred to be of type Double
  ```

+ 浮点转换为整数也必须显式地指定类型

   ```swift
    let integerPi = Int(pi)
    // integerPi equals 3, and is inferred to be of type Int
   ```



### 10 类型别名

+ 类型别名可以为已经存在的类型定义了一个新的可选名字。用 typealias 关键字定义类型别名。

  ```swift
  typealias AudioSample = UInt16
  ```


### 11 元组

+ 元组把多个值合并成单一的复合型的值。元组内的值可以是任何类型，而且可以不必是同一类型。

  ```swift
  //定义1
  let http404Error = (404, "Not Found")
  //取值
  let (statusCode, statusMessage) = http404Error
  //取一部分值
  let (justTheStatusCode, _) = http404Error
  //用下标取值
  print("The status code is \(http404Error.0)")
  
  //定义2
  let http200Status = (statusCode: 200, description: "OK")
  //取值
  print("The status code is \(http200Status.statusCode)")
  
  ```


### 12 可选项

+ 处理值可能缺失的情况, 有值或者没有值(nil)

  ```swift
  let possibleNumber = "123a"
  let convertedNumber = Int(possibleNumber)
  
  因为这个初始化器可能会失败，所以他会返回一个可选的Int(Int?) ，而不是 Int
  ```


### 13 nil

+ 你可以通过给`可选变量`赋值一个 nil 来将之设置为没有值

  ```swift
  var serverResponseCode: Int? = 404
  // serverResponseCode contains an actual Int value of 404
  serverResponseCode = nil
  // serverResponseCode now contains no value
  ```

+ nil `不能用于非可选的常量或者变量`，如果你的代码中变量或常量需要作用于特定条件下的值缺失，可以给他声明为相应类型的可选项。

+ 如果你定义的可选变量没有提供一个默认值，变量会被自动设置成 nil。`var surveyAnswer: String?`

+ Swift 中的 nil 和Objective-C 中的 nil 不同，在 Objective-C 中 nil 是一个指向不存在对象的指针。在 Swift中，nil 不是指针，它是值缺失的一种特殊类型，任何类型的可选项都可以设置成 nil 而不仅仅是对象类型。

### 14 If 语句以及强制展开

+ 利用 if 语句通过比较 nil 来判断一个可选中是否包含值

   ````swift
    如果一个可选有值，他就“不等于” nil 
    //convertedNumber是可选的
    if convertedNumber != nil {
      print("convertedNumber has an integer value of \(convertedNumber!).")
    }
   ````

+ 如上所示:`可选的名字!`获取对应的值

+ 使用 ! 来获取一个不存在的可选值会导致运行错误，在使用`!`强制展开之前必须确保可选项中包含一个`非 nil `的值

### 15 可选项绑定

+ 可以使用**可选项绑定**来判断可选项是否包含值，如果包含就把值赋给一个临时的常量或者变量

  ```swift
  if let constantName = someOptional { 
       print("\'\(someOptional!)\' has an  value of \(constantName)") 
  } 
  ```

+ 在同一个 if 语句中包含多可选项绑定，用逗号分隔即可

  ```swift
  if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
      print("\(firstNumber) < \(secondNumber) < 100")
  }
  ```

+ 使用 if 语句创建的常量和变量只在if语句的函数体内有效

### 16 隐式展开可选项

+ 隐式展开可选项: 如果某个可选项一旦被设定值之后，就会一直拥有值。

+ 声明一个隐式可选项，如果不赋值，其默认值为nil

+ 通过在声明的类型后边添加一个叹号（ String! ）而非问号（ String? ） 来书写隐式展开可选项

  ```swift
  //普通可选项
  let possibleString: String? = "An optional string."
  let forcedString: String = possibleString! // 需要感叹号
   
  //隐式展开可选项
  let assumedString: String! = "An implicitly unwrapped optional string."
  let implicitString: String = assumedString // 不需要感叹号
  ```

+ 可以像对待普通可选一样对待隐式展开可选项来检查里边是否包含一个值

  ```swift
  if assumedString != nil {
      print(assumedString)
  }
  ```

+ 可以使用隐式展开可选项通过可选项绑定在一句话中检查和展开值

  ```swift
  if let definiteString = assumedString {
      print(definiteString)
  }
  ```




### 17 错误处理

+ 通过在函数声明过程当中加入 throws 关键字来表明这个函数会抛出一个错误

  ```swift
  当一个函数遇到错误情况，他会抛出一个错误，这个函数的访问者会捕捉到这个错误，并作出合适的反应
  func canThrowAnError() throws {
      // this function may or may not throw an error
  }
  ```

+ 当你调用了一个可以抛出错误的函数时，需要在表达式前预置 try 关键字

  ```swift
  func makeASandwich() throws {
      // ...
  }
   
  do {
      try makeASandwich()
      eatASandwich()
  } catch Error.OutOfCleanDishes {
      washDishes()
  } catch Error.MissingIngredients(let ingredients) {
      buyGroceries(ingredients)
  }
  ```

### 18 断言和先决条件

+ 使用它们来保证在执行后续代码前某必要条件是满足的
+ 如果布尔条件在断言或先决条件中计算为 true ，代码就正常继续执行。
+ 如果条件计算为 false ，那么程序当前的状态就是非法的；代码执行结束，然后你的 app 终止。

### 19 使用断言进行调试

+ 可以使用全局函数 assert(_:_:)  函数来写断言

  ```swift
  let age = -3
  assert(age >= 0, "A person's age cannot be less than zero")
  //条件为true,继续执行。条件为false,程序终止。
  ```

### 20 强制先决条件

+ 通过调用 precondition(_:_:file:line:) 函数，对代码中`可能潜在为假但必须肯定为真`才能继续执行的地方使用先决条件

  ```swift
  precondition(index > 0, "Index must be greater than zero.")
  //如果条件的结果是 false 信息就会显示出来。
  ```


## 二 基本运算符

### 1 赋值运算符

* 赋值运算符（ a = b ）

### 2 算术运算符

+ 与 C 和 Objective-C 中的算术运算符不同，Swift 算术运算符默认不允许值溢出
+ 加 ( + ), 减 ( - ),乘 ( * ),除 ( / )
+ 加法运算符同时也支持 String  的拼接：`"hello, " + "world" // equals "hello, world"`

### 3 余数运算符

+ 余数运算符（ % ）同样会在别的语言中称作取模运算符

### 4 一元减号运算符

+ 数字值的正负号可以用前缀 `–`来切换，我们称之为 *一元减号运算符*

### 5 组合赋值符号

+ += , -=, *= 

### 6 比较运算符

- 相等 ( a == b )

- 不相等 ( a != b )

- 大于 ( a > b )

- 小于 ( a < b )

- 大于等于 ( a >= b )

- 小于等于 ( a <= b )

- 元组的比较

  ```swift
  (1, "zebra") < (2, "apple")   // true because 1 is less than 2
  (3, "apple") < (3, "bird")    // true because 3 is equal to 3, and "apple" is less than "bird"
  (4, "dog") == (4, "dog")      // true because 4 is equal to 4, and "dog" is equal to "dog"
  ```


### 7 三元条件运算符

+ `question ? answer1 : answer2`

### 8 合并空值运算符

+ 合并空值运算符 `（ a ?? b ）`
+ 如果`可选项 a  `有值则返回a的展开值，如果`可选项 a  `是 nil  ，则返回默认值 b。
+ 表达式 a 必须是一个可选类型
+ 表达式 b  必须与 a  的储存类型相同。
+ 等价于`a != nil ? a! : b`

### 9 闭区间运算符

+ *闭区间运算符*（ a...b ）(包含a ， b， 且a <= b)

### 10  半开区间运算符

+ *半开区间运算符*（ a..<b ）(包含a ,不包含 b，且 a<=b, 若a=b,则区间为空)
+ 没有a<..b的形式

### 11 单侧区间

+ arr[...a]数组中从0到下标为a的子数组
+ arr[a...]数组中从下标为a到数组末尾的子数组
+ 单侧形式:arr[..<a]

### 12 逻辑运算符   &&, |, !

- 逻辑 与  ( a && b )
- 逻辑 或  ( a || b )
- 逻辑 非  ( !a )



## 三 字符串和字符(String and Character)

### 1 字面量

* 字符串字面量是被双引号（ "）包裹的固定顺序文本字符

  ```swift
  作为常量或者变量的初始值
  let someString = "Some string literal value"
  ```

* 多行字符串字面量是用三个双引号

  + 普通使用多行字符串字面量

    ```swift
    let quotation = """
    The White Rabbit put on his spectacles.  "Where shall I begin,
    please your Majesty?" he asked.\"""
     
    "Begin at the beginning," the King said gravely, "and go on
    till you come to the end; then stop."
    """
    
    注意:
    1 如果字符串里面也有"""，那么用转义字符\
    
    
    ```

  + 多行字符串字面量开始或结束带有换行，写一个空行作为第一行或者是最后一行

    ```swift
    """
     
    This string starts with a line feed.
    It also ends with a line feed.
     
    """
    ```

  + 多行字符串可以缩进以匹配周围的代码

    ```swift
    let quotation = """
            The White Rabbit put on his spectacles.  "Where shall I begin,
               please your Majesty?" he asked.
     
               "Begin at the beginning," the King said gravely, "and go on
               till you come to the end; then stop."
            """
            
    注意: 如果在某行的空格超过了结束的双引号（ """ ），那么这些空格会被包含
    ```


### 2 初始化一个空字符串

+ 空字符串=> `""`,` String()`
+ `isEmpty`属性来确认一个 String值是否为空

### 3 字符串可变性

+ 通过let, var 来设置字符串是否可变

### 4 字符串是值类型

+ Swift 的 String类型是一种值类型
+ 每一次赋值和传递，现存的 String值都会被复制一次，传递走的是拷贝而不是原本。
+ Swift 的默认拷贝 String行为保证了当一个方法或者函数传给你一个 String值，你就绝对拥有了这个 String值
+ Swift 编译器优化了字符串使用的资源，实际上拷贝只会在确实需要的时候才进行。
  + 这意味着当你把字符串当做值类型来操作的时候总是能够有很棒的性能。

### 5 操作字符

+ 可以通过 for-in循环遍历 String 中的每一个独立的 Character值

  ```swift
  for character in "Dog!?" {
      print(character)
  }
  打印结果
   D
   o
   g
   !
   ?
  ```

+ 创建一个独立的 Character常量或者变量

  ```swift
  let exclamationMark: Character = "!" //值必须是单个字符，否则报错
  ```


### 6 连接字符串和字符

+ String值能够被+,+=连接

+ append()方法来可以给一个 String变量的末尾追加 Character值

  ```swift
  let exclamationMark: Character = "!"
  welcome.append(exclamationMark)
  ```

### 7 字符串插值

+ 每一个你插入到字符串字面量的元素都要被一对圆括号包裹，然后使用反斜杠前缀

  ```swift
  let multiplier = 3
  let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"
  ```


### 8 Unicode

> Unicode是一种在不同书写系统中编码、表示和处理文本的国际标准

#### 8.1 Unicode 标量

+ 一个 Unicode 标量是一个为字符或者修饰符创建的独一无二的21位数字

#### 8.2 字符串字面量中的特殊字符

- 转义特殊字符 \0 (空字符)， \\ (反斜杠)， \t (水平制表符)， \n (换行符)， \r(回车符)， \" (双引号) 以及\' (单引号)；

- 任意的 Unicode 标量，写作 \u{n}，里边的 n是一个 1-8 个与合法 Unicode 码位相等的16进制数字。

  ```swift
  let dollarSign = "\u{24}" // $, Unicode scalar U+0024
  let blackHeart = "\u{2665}" // ♥, Unicode scalar U+2665
  let sparklingHeart = "\u{1F496}" // ?, Unicode scalar U+1F496
  ```


#### 8.3 扩展字形集群 ????

> ??????



### 9 字符统计

+ 使用字符串的 count属性
+ 某些特殊情况下, count并不总是与NSString的length相同。



### 10 访问和修改字符串

>  你可以通过下标脚本语法或者它自身的属性和方法来访问和修改字符串。

#### 10.1 字符串索引

+ 使用下标语法访问字符串中特定的 Character

  ```swift
  let greeting = "Guten Tag!"
  
  greeting[greeting.startIndex]
  // G
  greeting[greeting.index(before: greeting.endIndex)]
  // !
  greeting[greeting.index(after: greeting.startIndex)]
  // u
  
  let index0 = greeting.index(greeting.startIndex, offsetBy: 0)
  greeting[index0]
  // G
  let index1 = greeting.index(greeting.startIndex, offsetBy: 1)
  greeting[index1]
  // u
  let index2 = greeting.index(greeting.startIndex, offsetBy: 2)
  greeting[index2]
  // t
  
  注意:
  1 startIndex访问第一个下标，endIndex 是String中最后一个字符后面的位置，并不是字符串的合法位置。
  2 index(before:) 和 index(after:) 方法来访问给定索引的前后
  3 index(_ i:offsetBy n:) 访问给定索引更远的索引。 offsetBy即为n相对于i偏移的距离
  ```

+ 尝试访问的 Character如果索引位置在字符串范围之外，就会触发运行时错误

  ```swift
  greeting[greeting.endIndex] // error
  greeting.index(after: endIndex) // error
  ```

+ 使用 indices属性来访问字符串中每个字符的索引。

  ```swift
  for index in greeting.indices {
      print("\(greeting[index]) ", terminator: "")
  }
  ```



#### 10.2 插入和删除

+ 插入字符(Character)到指定索引, 使用 `insert(_:at:)`方法

  ```swift
  var welcome = "hello"
  welcome.insert("!", at: welcome.endIndex)
  // welcome now equals "hello!"
  ```

+ 插入字符串(String)到指定索引,使用insert(contentsOf:at:) 方法

  ```swift
  welcome.insert(contentsOf: " there", at: welcome.index(before: welcome.endIndex))
  // welcome now equals "hello there!"
  ```

+ 从指定索引移除字符(Character),使用 remove(at:)方法

  ```swift
  welcome.remove(at: welcome.index(before: welcome.endIndex))
  // welcome now equals "hello there"
  ```

+ 从指定索引移除字符串(String),使用 removeSubrange(_:)方法

  ````swift
  let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
  welcome.removeSubrange(range)
  // welcome now equals "hello"
  ````

  >注意:
  >
  >你可以在任何遵循了 RangeReplaceableIndexable 协议的类型中使用 insert(_:at:) ， insert(contentsOf:at:) ，remove(at:) 方法。这包括了这里使用的 String ，同样还有集合类型比如 Array ， Dictionary 和 Set 。


### 11 子字符串 ???

> 需要好好理解



### 12 字符串比较

> Swift 提供了三种方法来比较文本值：字符串和字符相等性，前缀相等性以及后缀相等性。

#### 12.1 字符串和字符相等性

+ 使用 ==, != 比较

  ```swift
  let quotation = "We're a lot alike, you and I."
  let sameQuotation = "We're a lot alike, you and I."
  if quotation == sameQuotation {
      print("These two strings are considered equal")
  }
  ```

#### 12.2 前缀和后缀相等性

+ 使用 hasPrefix(\_:)方法比较前缀，使用hasSuffix(\_:)比较后缀

  ```swift
  str1.hasPrefix("abc") 
  str2.hasSuffix("abc") 
  ```


### 13 字符串的Unicode表示 ???

> ?????



## 四 集合类型

> 1 三种集合类型:Array, Set, Dictionary
>
> 2 数组是有序的值的集合
>
> 3 合集是唯一值的无序集合
>
> 4 字典是无序的键值对集合



### 1 集合的可变性

+ 如果把集合赋值给var变量，那么是可变集合
+ 如果把集合赋值给let常量，那么是不可变集合

### 2 数组(Array)

> 数组以有序的方式来储存相同类型的值。相同类型的值可以在数组的不同地方多次出现。

#### 2.1 数组类型简写语法

+ 完整写法是 `Array<Element>`,Element是数组允许存入的值的类型
+ 简写数组的类型为`[Element] `(更推荐)

#### 2.2 创建一个空数组

+ 创建空数组

  ```swift
  var someInts = [Int]()
  print("someInts is of type [Int] with \(someInts.count) items.")
  ```

#### 2.3 使用默认值创建数组

+ Array(repeating:, count:), repeating:传入初始化默认值, count: 新元素的数量

  ```swift
  var threeDoubles = Array(repeating: 0.0, count: 3)
  // threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
  ```

#### 2.4 通过连接两个数组来创建数组 

+ 两个兼容类型的现存数组用加运算符（ +）加在一起来创建一个新数组。

  ```swift
  var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
  // anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]
   
  var sixDoubles = threeDoubles + anotherThreeDoubles
  // sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
  ```

#### 2.5 使用数组字面量创建数组

```swift
1 var shoppingList: [String] = ["Eggs", "Milk"]
2 var shoppingList = ["Eggs", "Milk"]
```



#### 2.6 访问和修改数组

+ 数组中元素的数量，检查只读的 count属性

+ 布尔量 isEmpty属性来作为检查 count属性是否等于 0的快捷方式

+ append(_:)方法给数组末尾添加新的元素

+ 赋值运算符 ( +=)来在数组末尾添加一个或者多个同类型元素

  ```swift
  shoppingList += ["Baking Powder"]
  // shoppingList now contains 4 items
  shoppingList += ["Chocolate Spread", "Cheese", "Butter"]
  // shoppingList now contains 7 items
  ```

+ *下标脚本语法*来从数组访问和修改值

   ```swift
    var firstItem = shoppingList[0]
    shoppingList[0] = "123"
    shoppingList[4...6] = ["Bananas", "Apples"]
   ```

+ 要把元素插入到特定的索引位置，调用数组的 insert(_:at:)方法

  ```swift
  shoppingList.insert("Maple Syrup", at: 0)
  ```

+ 要把元素从特定的索引位置移除，调用数组的 remove(_:at:)方法

  ```swift
  let mapleSyrup = shoppingList.remove(at: 0)
  ```

#### 2.7 遍历一个数组

+ 直接遍历

  ```swift
  for item in shoppingList {
      print(item)
  }
  ```

+  enumerated()方法来遍历数组,enumerated() 返回数组中每一个元素的元组。

  ```swift
  
  for (index, value) in shoppingList.enumerated() {
      print("Item \(index + 1): \(value)")
  }
  // Item 1: Six eggs
  // Item 2: Milk
  // Item 3: Flour
  // Item 4: Baking Powder
  // Item 5: Bananas
  ```

  

### 3 合集(Set)

> 合集将同一类型且不重复的值无序地储存在一个集合当中。
>
> Swift 的 Set类型桥接到了基础框架的 NSSet类上。



#### 3.1 Set 类型的哈希值

+ 为了能让类型储存在合集当中，它必须是*可哈希*的
+ 所有 Swift 的基础类型（比如 String, Int, Double, 和 Bool）默认都是可哈希的
+ 自定义的类型作为合集的值类型或者字典的键类型，它们遵循 Swift 基础库的 Hashable协议即可。
+ 因为 Hashable协议遵循 Equatable，遵循的类型必须同时处理“等于”运算符 ( ==)的实现

#### 3.2 合集类型语法

+ 合集类型写做` Set<Element>`

#### 3.3 创建并初始化一个空合集

+ 初始化器语法创建

  ```swift
  var letters = Set<Character>()
  print("letters is of type Set<Character> with \(letters.count) items.")
  ```

+ 内容推断元素类型

  ```swift
  letters.insert("a")
  // letters now contains 1 value of type Character
  letters = []
  // letters is now an empty set, but is still of type Set<Character>
  ```

#### 3.4 使用数组字面量创建合集

+ 使用数组字面量来初始化一个合集

  ```swift
  var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
  ```

#### 3. 5访问和修改合集

+ 要得出合集当中元素的数量，检查它的只读 count属性
+ 使用布尔量 isEmpty属性作为检查 count属性是否等于 0的快捷方式
+ 调用 insert(_:)方法来添加一个新的元素到合集
+ remove(_:)方法来从合集当中移除一个元素
+ 要检查合集是否包含了特定的元素，使用 contains(_:)方法



#### 3.6 遍历合集

+ 普通遍历

  ```swift
  for genre in favoriteGenres {
      print("\(genre)")
  }
  // Classical
  // Jazz
  // Hip hop
  ```

+ 要以特定的顺序遍历合集的值，使用 sorted()方法，它把合集的元素作为使用 < 运算符排序了的数组返回

  ```swift
  for genre in favoriteGenres.sorted() {
      print("\(genre)")
  }
  // Classical
  // Hip hop
  // Jazz
  ```

### 4 执行合集操作 

> 你可以高效的执行基本合集操作，比如合并两个合集等

#### 4.1 基本合集操作

+ a.intersection(b) 返回一个新的集合为a与b的交集
+ a.symmetricDifference(b) 返回一个新的集合为a，b各自独有的元素的集合
+ a.union(b) 返回一个新的集合为a，b的并集
+ a.subtracting(b) 返回一个新的集合为a中独有的元素的集合

#### 4.2 合集成员关系和相等性

- 使用“相等”运算符 ( == )来判断两个集合是否完全相等
- 使用 isSubset(of:) 方法来确定一个合集的所有值是被某合集包含；
- 使用 isSuperset(of:)方法来确定一个合集是否包含某个合集的所有值；
- 使用 isStrictSubset(of:) 或者 isStrictSuperset(of:)方法来确定是个合集是否为某一个合集的子集或者超集，但并不相等；
- 使用 isDisjoint(with:)方法来判断两个合集是否拥有完全不同的值。

### 5 字典 (Dictionary)

> *字典*储存无序的互相关联的同一类型的键和同一类型的值的集合。
>
> 不同于数组中的元素，字典中的元素没有特定的顺序
>
> Swift 的 Dictionary桥接到了基础框架的 NSDictionary类。

#### 5.1 字典类型简写语法

+ 字典类型完整写法：` Dictionary<Key, Value>`
+ 简写的形式:` [Key: Value]` (推荐)
  + 注意: 字典的 Key类型必须遵循` Hashable`协议，就像合集的值类型。

#### 5.2 创建一个空字典

+ 用初始化器语法来创建一个空 Dictionary

  ```swift
  var namesOfIntegers = [Int: String]()
  // namesOfIntegers is an empty [Int: String] dictionary
  ```

+ 如果内容已经提供了信息，你就可以用字典字面量创建空字典了

  ```swift
  namesOfIntegers[16] = "sixteen"
  // namesOfIntegers now contains 1 key-value pair
  namesOfIntegers = [:]
  // namesOfIntegers is once again an empty dictionary of type [Int: String]
  ```


#### 5.3 用字典字面量创建字典

+ 字面量创建字典

  ```swift
  var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
  ```

#### 5.4 访问和修改字典

+ 用 count只读属性来找出 Dictionary拥有多少元素

+ isEmpty属性作为检查 count属性是否等于 0的快捷方式

+ 用下标脚本给字典添加/修改元素

  ```swift
  airports["LHR"] = "London"
  // the airports dictionary now contains 3 items
  ```

+ updateValue(_:forKey:)方法会在键没有值的时候设置一个值，或者在键已经存在的时候更新它, 执行更新之后返回旧的值（可选类型）。

  ```swift
  if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
      print("The old value for DUB was \(oldValue).")
  }
  
  这里airports.updateValue的返回值类型为String? (可选类型)
  ```

+ 直接用下标获取值返回可选类型的值

  ```swift
  if let airportName = airports["DUB"] {
      print("The name of the airport is \(airportName).")
  } else {
      print("That airport is not in the airports dictionary.")
  }
  ```

+ 使用下标脚本语法给一个键赋值 nil来从字典当中移除一个键值对

  ```swift
  airports["APL"] = "Apple International"
  // "Apple International" is not the real airport for APL, so delete it
  airports["APL"] = nil
  // APL has now been removed from the dictionary
  ```

+ 使用 removeValue(forKey:)来从字典里移除键值对。这个方法移除键值对如果他们存在的话，并且返回移除的值，如果值不存在则返回 nil, 所以其`返回值也是可选类型的`

  ```swift
  if let removedValue = airports.removeValue(forKey: "DUB") {
      print("The removed airport's name is \(removedValue).")
  } else {
      print("The airports dictionary does not contain a value for DUB.")
  }
  // Prints "The removed airport's name is Dublin Airport."
  ```

#### 5.5 遍历字典

+ for-in 遍历， 字典中的每一个元素返回为 (key, value)元组

  ```swift
  for (airportCode, airportName) in airports {
      print("\(airportCode): \(airportName)")
  }
  // YYZ: Toronto Pearson
  // LHR: London Heathrow
  ```

+ 同样可以通过访问字典的 keys和 values属性来取回可遍历的字典的键或值的集合

  ```swift
  
  for airportCode in airports.keys {
      print("Airport code: \(airportCode)")
  }
  // Airport code: YYZ
  // Airport code: LHR
   
  for airportName in airports.values {
      print("Airport name: \(airportName)")
  }
  // Airport name: Toronto Pearson
  // Airport name: London Heathrow
  ```

+  Swift 的 Dictionary类型是无序的。要以特定的顺序遍历字典的键或值，使用键或值的 sorted()方法。



## 五 控制流

### 1 For-in 循环

+ for-in 遍历数组，字典，合集等等

  ```swift
  let names = ["Anna", "Alex", "Brian", "Jack"]
  for name in names {
      print("Hello, \(name)!")
  }
  ```


### 2 while循环

+ while 在每次循环开始的时候计算它自己的条件；

  ```swift
  while condition {
      statements
  }
  ```

+ repeat-while 在每次循环结束的时候计算它自己的条件。

  ```swift
  repeat {
      statements
  } while condition
  ```



### 3 条件语句

#### 3.1 if

+ 语法格式

  ```swift
  if condition {
      statements
  } else condition {
      statements
  } else {
      statements
  }
  ```


#### 3.2 Switch 

+ 基本使用

  ```swift
  let someCharacter: Character = "z"
  switch someCharacter {
  case "a":
      print("The first letter of the alphabet")
  case "z":
      print("The last letter of the alphabet")
  default:
      print("Some other character")
  }
  // Prints "The last letter of the alphabet"
  ```


##### 3.2.1 没有隐式贯穿

+ Swift 里的 switch 语句不会默认从每个情况的末尾贯穿到下一个情况里,不再需要显式的 break 语句
+ 尽管 break 在 Swift 里不是必须的，你仍然可以使用 break 语句来匹配和忽略特定的情况，或者在某个情况执行完成之前就打断它
+ 要在特定的 switch 情况中使用贯穿行为，使用 fallthrough 关键字。



##### 3.2.2 区间匹配

+ switch情况的值可以在一个区间中匹配

  ````swift
  
  let approximateCount = 62
  let countedThings = "moons orbiting Saturn"
  var naturalCount: String
  switch approximateCount {
  case 0:
      naturalCount = "no"
  case 1..<5:
      naturalCount = "a few"
  case 5..<12:
      naturalCount = "several"
  case 12..<100:
      naturalCount = "dozens of"
  case 100..<1000:
      naturalCount = "hundreds of"
  default:
      naturalCount = "many"
  }
  print("There are \(naturalCount) \(countedThings).")
  // prints "There are dozens of moons orbiting Saturn."
  ````

##### 3.2.3 元组

+ 你可以使用元组来在一个 switch 语句中测试多个值, 使用下划线（ _）来表明匹配所有可能的值

```swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// prints "(1, 1) is inside the box"
```



##### 3.2.4 值绑定

+ switch 情况可以将匹配到的值临时绑定为一个常量或者变量，来给情况的函数体使用。这就是所谓的*值绑定*，因为值是在情况的函数体里“绑定”到临时的常量或者变量的。

  ```swift
  let anotherPoint = (2, 0)
  switch anotherPoint {
  case (let x, 0):
      print("on the x-axis with an x value of \(x)")
  case (0, let y):
      print("on the y-axis with a y value of \(y)")
  case let (x, y):
      print("somewhere else at (\(x), \(y))")
  }
  ```

+ 注意:这个 switch 语句没有任何的 default 情况。最后的情况， case let (x,y) ，声明了一个带有两个占位符常量的元组，它可以匹配所有的值。

##### 3.2.5 Where

+ switch 情况可以使用 where 分句来检查额外的情况。

  ````swift
  let yetAnotherPoint = (1, -1)
  switch yetAnotherPoint {
  case let (x, y) where x == y:
      print("(\(x), \(y)) is on the line x == y")
  case let (x, y) where x == -y:
      print("(\(x), \(y)) is on the line x == -y")
  case let (x, y):
      print("(\(x), \(y)) is just some arbitrary point")
  }
  // prints "(1, -1) is on the line x == -y"
  ````

+ 注意:这个 switch 语句没有任何的 default 情况。最后的情况， case let (x,y) ，声明了一个带有两个占位符常量的元组，它可以匹配所有的值。

##### 3.2.6 复合情况

+ 多个 switch 共享同一个函数体的多个情况可以在 case 后写多个模式来复合，在每个模式之间用逗号分隔

  ```swift
  let someCharacter: Character = "e"
  switch someCharacter {
  case "a", "e", "i", "o", "u":
      print("\(someCharacter) is a vowel")
  case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
       "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
      print("\(someCharacter) is a consonant")
  default:
      print("\(someCharacter) is not a vowel or a consonant")
  }
  ```

+ 复合情况可以包含值绑定

  ```swift
  let stillAnotherPoint = (9, 0)
  switch stillAnotherPoint {
  case (let distance, 0), (0, let distance):
      print("On an axis, \(distance) from the origin")
  default:
      print("Not on an axis")
  }
  // Prints "On an axis, 9 from the origin"
  ```


### 4 控制转移语句

- `continue`
- `break`
- `fallthrough`
- `return`
- `throw`

#### 4.1 给语句打标签 

+ 对于多层嵌套的循环和添加语句，可以通过显式的标记循环语句或者条件语句，然后配合 break 或者 continue来结束或继续你想要操作的循环语句或者条件语句。

  ```swift
  label name: while condition {
      statements
  }
  
  //example
   var i = 0
   loop : while i <= 5 {
              i = i+1
              switch i {
              case 1:
                  print("哈哈1")
              case 2:
                  print("哈哈2")
              case 3:
                  print("哈哈3")
                  //跳过此次loop循环，进行下一次循环
                  continue loop
              case 4:
                  print("哈哈4")
                  //跳出loop循环
                  break loop
              default:
                  print("哈哈5")
              }
              print(i)
        }
  
  //prints
  哈哈1
  1
  哈哈2
  2
  哈哈3
  哈哈4
  ```

### 5 提前退出

+ 使用 guard 语句来要求一个条件必须是真才能执行guard 之后的语句

+ guard 语句总是有一个 else 分句—— else 分句里的代码会在条件不为真的时候执行。

  ```swift
  func greet(person: [String: String]) {
      guard let name = person["name"] else {
          return
      }
      
      print("Hello \(name)!")
      
      guard let location = person["location"] else {
          print("I hope the weather is nice near you.")
          return
      }
      
      print("I hope the weather is nice in \(location).")
  }
   
  greet(person: ["name": "John"])
  // Prints "Hello John!"
  // Prints "I hope the weather is nice near you."
  greet(person: ["name": "Jane", "location": "Cupertino"])
  // Prints "Hello Jane!"
  // Prints "I hope the weather is nice in Cupertino."
  ```





### 6 检查API的可用性

+ Swift 拥有内置的对 API 可用性的检查功能，它能够确保你不会悲剧地使用了对部属目标不可用的 API。

+ 当验证在代码块中的 API 可用性时，编译器使用来自可用性条件里的信息来检查。

  ```swift
  //语法
  if #available(platform name version, ..., *) {
      statements to execute if the APIs are available
  } else {
      fallback statements to execute if the APIs are unavailable
  }
  
  
  //example
  if #available(iOS 10, macOS 10.12, *) {
      // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
  } else {
      // Fall back to earlier iOS and macOS APIs
  }
  ```



## 六 函数

### 1 定义和调用函数 

+ 它使用一个 func的关键字前缀。你可以用一个返回箭头`->` (一个连字符后面跟一个向右的尖括号)来明确函数返回的类型。

  ```swift
  //定义
  func functionName(参数名:参数类型) -> 返回值类型 {
      函数体
  }
  
  //调用
  functionName(参数名:参数)
  ```

### 2 函数的形式参数和返回值

> 在 Swift 中，函数的形式参数和返回值非常灵活

#### 2.1 无形式参数的函数

+ 一个无参函数示例

  ```swift
  func sayHelloWorld() -> String {
      return "hello, world"
  }
  print(sayHelloWorld())
  // prints "hello, world"
  ```


#### 2.2 多形式参数的函数

+ 函数可以输入多个形式参数，可以写在函数后边的圆括号内，用逗号分隔。

  ```swift
  func greet(person: String, alreadyGreeted: Bool) -> String {
      if alreadyGreeted {
          return greetAgain(person: person)
      } else {
          return greet(person: person)
      }
  }
  print(greet(person: "Tim", alreadyGreeted: true))
  ```

  + person, alreadyGreeted 是标签



#### 2.3 无返回值的函数

+ 无返回值参数示例

  ```swift
  func greet(person: String) {
      print("Hello, \(person)!")
  }
  greet(person: "Dave")
  // Prints "Hello, Dave!"
  ```

  + 严格来讲，函数 greet(person:)尽管没有定义返回值,但还是有一个返回值的。
  + 没有定义返回类型的函数实际上会返回一个特殊的类型 Void。
  + 它其实是一个空的元组，作用相当于没有元素的元组，可以写作 ()。

#### 2.4 多返回值的函数

+ 为了让函数返回多个值作为一个复合的返回值，你可以使用元组类型作为返回类型。

  ```swift
  func minMax(array: [Int]) -> (min: Int, max: Int) {
      var currentMin = array[0]
      var currentMax = array[0]
      for value in array[1..<array.count] {
          if value < currentMin {
              currentMin = value
          } else if value > currentMax {
              currentMax = value
          }
      }
      return (currentMin, currentMax)
  }
  ```



#### 2.5 可选元组返回类型

+ 如果返回的元组可能为nil，那么返回值要定义为可选的元组。例如 ` (Int, Int)?`  或者` (String, Int, Bool)?`

+ 类似 (Int, Int)?的可选元组类型和包含可选类型的元组 (Int?, Int?)是不同的

  ```swift
  func minMax(array: [Int]) -> (min: Int, max: Int)? {
      if array.isEmpty { return nil }
      var currentMin = array[0]
      var currentMax = array[0]
      for value in array[1..<array.count] {
          if value < currentMin {
              currentMin = value
          } else if value > currentMax {
              currentMax = value
          }
      }
      return (currentMin, currentMax)
  }
  ```


### 3 函数实际参数标签和形式参数名

+ 每一个函数的形式参数都包含*实际参数标签*和*形式参数名*

+ 默认情况下，形式参数使用它们的形式参数名作为实际参数标签。

  ```swift
  func someFunction(firstParameterName: Int, secondParameterName: Int) {
      // In the function body, firstParameterName and secondParameterName
      // refer to the argument values for the first and second parameters.
  }
  someFunction(firstParameterName: 1, secondParameterName: 2)
  
  //firstParameterName, secondParameterName是形式参数名
  //同时默认firstParameterName，secondParameterName为实际参数标签
  ```


#### 3.1 指定实际参数标签

+ 在提供形式参数名之前写实际参数标签，用空格分隔：

  ````swift
  //语法
  func someFunction(argumentLabel parameterName: Int) {
      // In the function body, parameterName refers to the argument value
      // for that parameter.
  }
  
  //example
  func greet(person: String, from hometown: String) -> String {
      return "Hello \(person)!  Glad you could visit from \(hometown)."
  }
  print(greet(person: "Bill", from: "Cupertino"))
  // Prints "Hello Bill!  Glad you could visit from Cupertino."
  ````

+ 实际参数标签的使用能够让函数的调用更加明确，更像是自然语句



#### 3.2 省略实际参数标签

+ 对于函数的形式参数不想使用实际参数标签的话，可以利用下划线（ _ ）来为这个形式参数代替显式的实际参数标签。

  ```swift
  func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
      // In the function body, firstParameterName and secondParameterName
      // refer to the argument values for the first and second parameters.
  }
  someFunction(1, secondParameterName: 2)
  ```


#### 3.3 默认形式参数值

+ 通过在形式参数类型后给形式参数赋一个值来给函数的任意形式参数定义一个*默认值*

+ 如果定义了默认值，你就可以在调用函数时候省略这个形式参数。

  ```swift
  func someFunction(parameterWithDefault: Int = 12) {
      // In the function body, if no arguments are passed to the function
      // call, the value of parameterWithDefault is 12.
  }
  someFunction(parameterWithDefault: 6) // parameterWithDefault is 6
  someFunction() // parameterWithDefault is 12
  ```

#### 3.4 可变形式参数 [Type...]

+ 一个*可变形式参数*可以接受零或者多个特定类型的值

+ 可以通过在形式参数的类型名称后边插入三个点符号（ ...）来书写可变形式参数。

+ 一个函数最多只能有一个可变形式参数。

+ 传入到可变参数中的值在函数的主体中被当作是对应类型的数组

  ```swift
  func arithmeticMean(_ numbers: Double...) -> Double {
      var total: Double = 0
      for number in numbers {
          total += number
      }
      return total / Double(numbers.count)
  }
  arithmeticMean(1, 2, 3, 4, 5)
  // returns 3.0, which is the arithmetic mean of these five numbers
  arithmeticMean(3, 8.25, 18.75)
  // returns 10.0, which is the arithmetic mean of these three numbers
  
  //此时numbers被当做[Double]类型的常量数组
  ```

#### 3.5 输入输出形式参数[即为能在函数内部被改变的形式参数]

+ 可变形式参数只能在函数的内部做改变。

+ 如果你想函数能够修改一个形式参数的值，而且你想这些改变在函数结束之后依然生效，那么就需要将形式参数定义为*输入输出形式参数*。

+ 在前边添加一个 inout关键字可以定义一个输入输出形式参数

+ 只能把变量作为输入输出形式参数的实际参数

+ 使用时，直接在实际参数前面加& 来表明可被函数修改。

  ```swift
  func swapTwoInts(_ a: inout Int, _ b: inout Int) {
      let temporaryA = a
      a = b
      b = temporaryA
  }
  
  var someInt = 3
  var anotherInt = 107
  swapTwoInts(&someInt, &anotherInt)
  print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
  //someInt is now 107, and anotherInt is now 3
  
  //如果是不加inout关键字，那么形式参数就是作为常量被传入，不能改变它的值，否则编译报错
  func swapTwoInts2(_ a: Int, _ b: Int) {
      let temporaryA = a
      //Error:Cannot assign to value: 'a' is a 'let' constant
      a = b
      //Error:Cannot assign to value: 'b' is a 'let' constant
      b = temporaryA
  }
  
  ```

  + 输入输出形式参数不能有默认值
  + 可变形式参数(...)不能标记为 inout 
  + 如果你给一个形式参数标记了 inout，那么它们也不能标记 var和 let了。

### 4 函数类型

>  每一个函数都有一个特定的*函数类型*，它由形式参数类型，返回类型组成

```swift
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}

其函数类型为: (Int, Int) -> Int

func printHelloWorld() {
    print("hello, world")
}
其函数类型为: () -> Void
```

#### 4.1 使用函数类型

+ 可以像使用 Swift 中的其他类型一样使用函数类型

  ```swift
  var mathFunction: (Int, Int) -> Int = addTwoInts
  var mathFunction2 = addTwoInts //类型推断
  print("Result: \(mathFunction(2, 3))")
  ```

#### 4.2 函数类型作为形式参数类型

+ 使用一个函数的类型例如 (Int, Int) -> Int作为其他函数的形式参数类型

  ```swift
  func printMathResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
      print("Result: \(mathFunction(a, b))")
  }
  printMathResult(addTwoInts, 3, 5)
  //用途: 预留部分实现，由函数调用者提供
  ```


#### 4.3 函数类型作为返回类型

+ 函数的类型作为另一个函数的返回类型

  ```swift
  func chooseStepFunction(backwards: Bool) -> (Int) -> Int {
      return backwards ? stepBackward : stepForward
  }
  ```

### 5  内嵌函数

+ 内嵌函数:也就是在函数的内部定义函数

  ```swift
  func chooseStepFunction(backward: Bool) -> (Int) -> Int {
      func stepForward(input: Int) -> Int { return input + 1 }
      func stepBackward(input: Int) -> Int { return input - 1 }
      return backward ? stepBackward : stepForward
  }
  ```


## 七 闭包

> 闭包是可以在你的代码中被传递和引用的功能性独立模块
>
> 闭包能够捕获和存储定义在其上下文中的任何常量和变量的引用,因此被称为“闭包”
>
> Swift 能够为你处理所有关于捕获的内存管理的操作
>
> 全局和内嵌函数，实际上是特殊的闭包



- 闭包符合如下三种形式中

  - 全局函数是一个有名字但不会捕获任何值的闭包；

  - 内嵌函数是一个有名字且能从其上层函数捕获值的闭包；

  - 闭包表达式是一个轻量级语法所写的可以捕获其上下文中常量或变量值的没有名字的闭包。

- Swift 的闭包表达式拥有简洁的风格,常见的优化包括: 

  - 利用上下文推断形式参数和返回值的类型；
  - 单表达式的闭包可以隐式返回；
  - 简写实际参数名；
  - 尾随闭包语法。

### 1 闭包表达式

>  `闭包表达式`是一种在简短行内就能写完闭包的语法， 为了缩减书写长度同时又不失易读明晰。

#### 1.1 sorted方法

- Swift 的标准库提供了一个叫做 `sorted(by:)` 的方法，会根据你提供的`排序闭包`将已知类型的数组的值进行排序

  - 返回与原数组类型大小完全相同的一个已经排好序新数组，原始数组并不会被改变

  - 方法接收`一个接收两个与数组内容相同类型的实际参数的闭包`，然后返回一个 Bool 值。bool为true，则第一个值在第二个值之前。false,则相反

    ```swift
    //sorted传入的闭包参数，如果返回值为true,则表明排序后第一个参数，在第二个参数前面
    func backward(_ s1: String, _ s2: String) -> Bool {
        return s1 > s2
    }
    var reversedNames = names.sorted(by: backward)
    ```


#### 1.2 闭包表达式语法

- 一般语法格式

  ```swift
    { (parameters) -> (return type) in
      statements
    }
  ```

- 闭包表达式语法能够使用`常量形式参数`、`变量形式参数`和`输入输出形式参数`，但不能提供默认值

- 可变形式参数(Type...)也能使用，但需要在形式参数列表的最后面使用 

- 元组也可被用来作为形式参数和返回类型

- 闭包的函数整体部分由关键字` in` 导入。这个关键字之前是闭包形式参数和返回值类型的定义，其后面的是函数体部分。

  ```swift
  reversedNames = names.sorted(by:{ (s1: String, s2: String) -> Bool in return s1 > s2 })
  ```

#### 1.3 从语境中推断类型

- 因`排序闭包`为`实际参数`来传递给函数，故 Swift 能推断它的形式参数类型和返回类型。因为所有的类型都能被推断，所以形式参数周围的括号，以及返回值类型都能被省略

  ```swift
  reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
  ```

  - 省略，使语法更为简洁。不省略，使别人看的更加清晰。(自己把握是否省略)

#### 1.4 从单表达式闭包隐式返回

- 例如`sorted(by:)`已经明确返回一个bool值，return 关键字能够被省略。

  ```swift
  reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )
  ```

#### 1.5 简写的实际参数名

- Swift 自动对行内闭包提供简写实际参数名，你也可以通过 $0 , $1 , $2 等名字来引用闭包的实际参数值

- in  关键字也能被省略，因为闭包表达式完全由它的`函数体`组成

  ```swift
  reversedNames = names.sorted(by: { $0 > $1 } )
  ```

### 2 尾随闭包

- 将一个很长的`闭包表达式`作为函数`最后一个实际参数`传递给函数，使用`尾随闭包`将增强函数的可读性

  ```swift
  func someFunctionThatTakesAClosure(closure:() -> Void){
         //function body goes here
    }
   
    //here's how you call this function without using a trailing closure
    someFunctionThatTakesAClosure(closure:{
         //closure's body goes here
    })
      
    //here's how you call this function with a trailing closure instead
        
    someFunctionThatTakesAClosure() {
         // trailing closure's body goes here
    }
  ```

- 如果闭包表达式作为函数的唯一实际参数传入，而你又使用了尾随闭包的语法，那你就不需要在函数名后边写圆括号了

  ```swift
  reversedNames = names.sorted { $0 > $1 }
  ```


### 3 捕获值

- 一个闭包能够从上下文捕获已被定义的常量和变量。即使定义这些常量和变量的原作用域已经不存在，闭包仍能够在其函数体内引用和修改这些值。
- 在 Swift 中，一个能够捕获值的闭包最简单的模型是内嵌函数，即被书写在另一个函数的内部
- 一个内嵌函数能够捕获外部函数的实际参数并且能够捕获任何在外部函数的内部定义了的常量与变量。

> 注意: 
>
> 作为一种优化，如果一个值没有改变或者在闭包的外面，Swift 可能会使用这个值的*拷贝*而不是捕获。Swift也处理了变量的内存管理操作，当变量不再需要时会被释放。



### 4 闭包是引用类型

-  闭包是引用类型而且是强引用

### 5 逃逸闭包

- `闭包逃逸` : 当闭包作为一个实际参数传递给一个函数的时候,但是在这个函数执返回时，闭包并没有被调用，我们就说这个闭包逃逸了。(即为在当前函数执行完后，才在合适的时间调用闭包)

- 可以在形式参数前写 `@escaping `来明确闭包是允许逃逸的。

  ```swift
  var completionHandlers: [() -> Void] = []
  func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
      completionHandlers.append(completionHandler)
  }
  ```

- 如果传递给函数的闭包是一个逃逸闭包(@escaping修饰),那么调用这个函数的时候，闭包体里`必须显式地`引用 self(必写self)

- 如果传递给函数的闭包是一个非逃逸闭包,那么调用这个函数的时候，闭包体里`可以隐式地`引用 self(即为可写self,也可不写self)

  ```swift
  func someFunctionWithNonescapingClosure(closure: () -> Void) {
      closure()
  }
   
  class SomeClass {
      var x = 10
      func doSomething() {
          someFunctionWithEscapingClosure { self.x = 100 } //显式引用self
          someFunctionWithNonescapingClosure { x = 200 } //隐式引用self
      }
  }
  ```


### 6 自动闭包

- **自动闭包**: 它不接受任何实际参数，并且当它被调用时，内部闭包的表达式的值作为返回值

  ```swift
  let customerProvider = { customersInLine.remove(at: 0) }//定义自动闭包
  let value = customerProvider() //调用
  
  customerProvider就是一个自动闭包，customerProvider的类型为()->String
  ```

- **普通闭包**和**自动闭包**作为函数形式参数时的对比

  ```swift
  //普通闭包
  func serve(customer customerProvider: () -> String) {
      print("Now serving \(customerProvider())!")
  }
  serve(customer: { customersInLine.remove(at: 0) } )
  
  //自动闭包
  func serve(customer customerProvider: @autoclosure () -> String) {
      print("Now serving \(customerProvider())!")
  }
  serve(customer: customersInLine.remove(at: 0))
  ```

  - 自动闭包:将传入的表达式作为闭包表达式的函数体部分，闭包的返回值即为传入的表达式的值
  - 自动闭包作为参数时, 要用`@autoclosure`修饰。

- assert(condition:message:file:line:)的condition,message也是一个闭包,也是在特定条件下执行的

  - condition是只有调试时才执行

  - message也是只有condition为false才执行

    ```swift
    //asset定义
    public func assert(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line)
    ```

- 自动闭包逃逸时也需要加上@escaping

  ```swift
  var customerProviders: [() -> String] = []
  func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
      customerProviders.append(customerProvider)
  }
  ```


## 八 枚举

> 值可以是字符串、字符、任意的整数值，或者是浮点类型
>
> 不同枚举成员关联值的类型可以不同



### 1 枚举语法

- enum关键字来定义一个枚举，然后将其所有的定义内容放在一个大括号`{}`中, 多个成员值可以出现在同一行中，要用逗号隔开

  ```swift
  enum CompassPoint {
      case north
      case south
      case east
      case west
  }
  //或者
  enum Planet {
      case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
  }
  ```

- 每个枚举都定义了一个`全新的类型`

  ```swift
  var directionToHead = CompassPoint.west
  directionToHead = .east
  //directionToHead初始化过后，就已经被类型推断了，直接用.语法即可
  ```



### 2 使用 Switch 语句来匹配枚举值

- 用 switch语句来匹配每一个单独的枚举值,每个成员都必须包括,否则会报错。(没有default时)

  ```swift
  directionToHead = .south
  switch directionToHead {
  case .north:
      print("Lots of planets have a north")
  case .south:
      print("Watch out for penguins")
  case .east:
      print("Where the sun rises")
  case .west:
      print("Where the skies are blue")
  }
  ```

- 如果不能为所有枚举成员都提供一个 case，那你也可以提供一个 default情况

  ```swift
  let somePlanet = Planet.earth
  switch somePlanet {
  case .earth:
      print("Mostly harmless")
  default:
      print("Not a safe place for humans")
  }
  ```



### 3 遍历枚举情况(case) 

- 你可以通过在枚举名字后面写 : CaseIterable 来允许枚举被遍历。Swift 会暴露一个包含对应枚举类型所有情况的集合名为 allCases （Set） 

  ```swift
  enum Beverage: CaseIterable {
      case coffee, tea, juice
  }
  let numberOfChoices = Beverage.allCases.count
  print("\(numberOfChoices) beverages available")
  ```


### 4 关联值

- 不同枚举成员关联值的类型可以不同

  ```swift
  enum Barcode {
      case upc(Int, Int, Int, Int)//upc的关联值为元组类型(Int, Int, Int, Int)
      case qrCode(String)//qrCode的关联值为String类型
  }
  
  var productBarcode = Barcode.upc(8, 85909, 51226, 3)
  或者
  productBarcode = .qrCode("ABCDEFGHIJKLMNOP")
  
  ```

- 提取的每一个相关值都可以作为常量（用 let前缀) 或者变量（用 var前缀）在 switch的 case中使用

  ```swift
  switch productBarcode {
  case .upc(let numberSystem, let manufacturer, let product, let check):
      print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
  case .qrCode(let productCode):
      print("QR code: \(productCode).")
  }
  
  ```

- 对于一个枚举成员的所有的相关值都被提取为常量，或如果都被提取为变量，为了简洁，你可以用一个单独的 var或let在成员名称前标注：

  ```swift
  switch productBarcode {
  case let .upc(numberSystem, manufacturer, product, check):
      print("UPC : \(numberSystem), \(manufacturer), \(product), \(check).")
  case let .qrCode(productCode):
      print("QR code: \(productCode).")
  
  ```


### 5 原始值

- 枚举成员可以用相同类型的默认值预先填充（称为原始值）

  ```swift
  enum ASCIIControlCharacter: Character {
      case tab = "\t"
      case lineFeed = "\n"
      case carriageReturn = "\r"
  }
  ```

#### 5.1 隐式指定的原始值

- 操作存储`整数`或`字符串`原始值枚举的时候，你不必显式地给每一个成员都分配一个原始值。当你没有分配时，Swift 将会自动为你分配值。 使用`rawValue`属性来访问一个枚举成员的原始值

- 整数值被用于作为原始值时, 每成员的隐式值都比前一个大一。如果第一个成员没有值，那么它的值是 0

  ```swift
  enum Planet: Int {
      case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
  }
  let earthsOrder = Planet.Earth.rawValue
  // earthsOrder is 3
  ```

- 当字符串被用于原始值，那么每一个成员的隐式原始值则是那个成员的名称。

  ```swift
  enum CompassPoint: String {
      case north, south, east, west
  }
  
  let sunsetDirection = CompassPoint.west.rawValue
  // sunsetDirection is "west"
  ```


#### 5.2 从原始值初始化

- 如果你用原始值类型来定义一个枚举，那么枚举就会自动收到一个可以接受原始值类型的值的初始化器，返回值是可选类型, 为枚举成员或者 nil

  ```
  let possiblePlanet = Planet(rawValue: 7)
  // possiblePlanet is of type Planet? and equals Planet.Uranus
  possiblePlanet的类型为Planet?
  ```

### 6 递归枚举

- *递归枚举*是拥有另一个枚举作为枚举成员关联值的枚举
- 当编译器操作递归枚举时必须插入间接寻址层。你可以在声明枚举成员之前使用 indirect关键字来明确它是递归的。

[递归枚举示例](https://www.cnswift.org/enumerations#spl-7)

## 

## 九 类和结构体

> 类和结构体是一种多功能且灵活的构造体



### 1 类和结构体的对比

+ 在 Swift 中类和结构体有很多共同之处
  + 定义属性用来存储值
  + 定义方法用于提供功能
  + 定义下标脚本用来允许使用下标语法访问值；
  + 定义初始化器用于初始化状态
  + 可以被扩展来默认所没有的功能
  + 遵循协议来针对特定类型提供标准功能。
+ 类有独有的功能
  + 继承允许一个类继承另一个类的特征;
  + 类型转换允许你在运行检查和解释一个类实例的类型；
  + 反初始化器允许一个类实例释放任何其所被分配的资源；
  + 引用计数允许不止一个对类实例的引用。
+ 注意: 结构体在你的代码中通过复制来传递，并且并不会使用引用计数。

#### 1.1 定义语法

+ 根据class定义类或struct定义结构体

  ```swift
  struct Resolution {
        var width = 0
        var height = 0 
    }
  
   class VideoMode {
        var resolution = Resolution()
        var interlaced = false
        var frameRate = 0.0
        var name: String?
    }
  ```

#### 1.2 类与结构体实例

+ 类与结构体初始化器语法来生成新的实例, 任何属性都被初始化为它们的默认值

  ```swift
  let someResolution = Resolution()
  let someVideoMode = VideoMode()
  ```

#### 1.3 访问属性

+ 用*点语法*来访问一个实例的属性

#### 1.4 结构体类型的成员初始化器

+ 所有的结构体都有一个`自动生成的成员初始化器`，你可以使用它来初始化新结构体实例的成员属性

  ```swift
  let vga = Resolution(width: 640, height: 480)
  ```

+ 类实例不会自动生成默认的成员初始化器

### 2 结构体和枚举是值类型

+ `值类型`: 是一种当它被指定到常量或者变量，或者被传递给函数时会`被拷贝`的类型。

  + 整数，浮点数，布尔量，字符串，数组和字典, 结构体和枚举--> 都是值类型，并且都以结构体的形式在后台实现。

+ 结构体实例传递是值拷贝示例

  ```swift
  let hd = Resolution(width: 1920, height: 1080)
  var cinema = hd
  
  //hd与cinema在后台是两个完全不同的实例。改变其中一个的属性值，对另一个没有任何影响
  ```

+ 枚举传递是值拷贝示例

  ```swift
  enum CompassPoint {
      case North, South, East, West
  }
  var currentDirection = CompassPoint.West
  let rememberedDirection = currentDirection
  currentDirection = .East
  if rememberedDirection == .West {
      print("The remembered direction is still .West")
  }
  
  //rememberedDirection与currentDirection在后台是两个完全不同的实例。改变其中一个的属性值，对另一个没有任何影响
  ```


### 3 类是引用类型

+ 类是引用类型，多次赋值则多个变量在后台指向同一份实例

  ```swift
  let tenEighty = VideoMode()
  tenEighty.name = "1080i"
  
  let alsoTenEighty = tenEighty
  alsoTenEighty.frameRate = 30.0
  
  //tenEighty, alsoTenEighty都是常量。它们对实例的引用没有改变，只是实例本身的值改变了。
  ```



#### 3.1 特征运算符

+ 相同于 ( ===)  不相同于( !==) , 利用这两个恒等运算符来检查两个常量或者变量是否引用相同的实例：

  ````swift
  if tenEighty === alsoTenEighty {
      print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
  }
  ````

+ “等于”意味着两个实例的在值上被视作“相等”或者“等价”，某种意义上的“相等”，就如同类设计者定义的那样。

#### 3.2 指针

+ 一个 Swift 的常量或者变量指向某个实例的引用类型和 C 中的指针类似，但是这并不是直接指向内存地址的指针

### 4 类和结构体之间的选择

+ 符合以下一条或多条情形时应考虑创建一个结构体

  + 结构体的主要目的是为了封装一些相关的简单数据值；
  + 当你在赋予或者传递结构实例时，有理由需要封装的数据值被拷贝而不是引用
  + 任何存储在结构体中的属性是值类型，也将被拷贝而不是被引用；
  + 结构体不需要从一个已存在类型继承属性或者行为。

         ```swift

     合适的结构体候选者包括：

  几何形状的大小，可能封装了一个 width属性和 height属性，两者都为 double类型；
  一定范围的路径，可能封装了一个 start属性和 length属性，两者为 Int类型；
  三维坐标系的一个点，可能封装了 x , y 和 z属性，他们都是 double类型。
  ​       ```

+ 在其他的情况下，定义一个类，并创建这个类的实例通过引用来管理和传递。

+ 大部分的自定义的数据结构应该是类，而不是结构体。

### 5 字符串，数组和字典的赋值与拷贝行为

+ Swift 的 String , Array 和 Dictionary类型是作为结构体来实现的
+ 这意味着字符串，数组和字典在它们被赋值到一个新的常量或者变量，亦或者它们本身被传递到一个函数或方法中的时候，`其实是传递了拷贝`

## 十 属性

1. 存储属性:会存储常量或变量作为实例的一部分. 存储属性只能由类和结构体定义
2. 计算属性会计算（而不是存储）值, 计算属性可以由类、结构体和枚举定义
3. 存储属性和计算属性通常和特定类型的`实例`相关联。
4. `类型属性`:与类本身相关联
5. 可以定义属性观察器来检查属性中值的变化



### 1 存储属性

+ 存储属性是一个作为特定类和结构体实例一部分的常量或变量
+ 存储属性要么是*变量存储属性*（由 var  关键字引入）要么是*常量存储属性*（由 let  关键字引入）

#### 1.1 常量结构体实例的存储属性

+ 如果你创建了一个结构体的实例并且把这个实例赋给常量，你不能修改这个实例的属性，即使是声明为变量的属性

+ 这是由于结构体是*值类型*。当一个值类型的实例被标记为常量时，该实例的其他属性也均为常量

  ```swift
  struct FixedLengthRange {
      var firstValue: Int
      let length: Int
  }
  let rangeOfFourItems = FixedLengthRange(firstValue: 0, length: 4)
  // this range represents integer values 0, 1, 2, and 3
  rangeOfFourItems.firstValue = 6  //报错
  // this will report an error, even though firstValue is a variable property
  ```

  + 对于类来说则不同，它是*引用类型*。如果你给一个常量赋值引用类型实例，你仍然可以修改那个实例的变量属性。

#### 1.2 延迟存储属性

+ 延迟存储属性的初始值在其第一次使用时才进行计算,  其声明前标注 lazy 修饰语

+ 用于两个场景:

  + 一个属性的初始值可能依赖于某些外部因素，当这些外部因素的值只有在实例的初始化完成后才能得到

  + 而当属性的初始值需要执行复杂或代价高昂的配置才能获得，你又想要在需要时才执行

    ```swift
    class DataImporter {
        var fileName = "data.txt"
    }
     
    class DataManager {
       //读取文件是一个耗时操作。
        lazy var importer = DataImporter()
        var data = [String]()
    }
     
    let manager = DataManager()
    manager.data.append("Some data")
    manager.data.append("Some more data")
    ```

  > 注意:
  >
  > 1 你必须把延迟存储属性声明为变量（使用 var 关键字），因为它的初始值可能在实例初始化完成之前无法取得。常量属性则必须在初始化完成*之前*有值，因此不能声明为延迟。
  >
  > 2 如果被标记为 lazy 修饰符的属性同时被多个线程访问并且属性还没有被初始化，则无法保证属性只初始化一次

#### 1.3 存储属性与实例变量

+ Swift 把这些概念都统一到了属性声明里。Swift 属性没有与之相对应的实例变量，并且属性的后备存储不能被直接访问。 ？？？

### 2 计算属性

+ 实际并不存储值，提供一个读取器和一个可选的设置器来间接得到和设置其他的属性和值。

  ```swift
  struct Point {
        var x = 0.0, y = 0.0
    }
    struct Size {
        var width = 0.0, height = 0.0
    }
    struct Rect {
        var origin = Point()
        var size = Size()
        // 1 center即为计算属性，其有一个读取器getter,设置器setter
        var center: Point {
            get {
                let centerX = origin.x + (size.width / 2)
                let centerY = origin.y + (size.height / 2)
                return Point(x: centerX, y: centerY)
            }
            set(newCenter) {
                origin.x = newCenter.x - (size.width / 2)
                origin.y = newCenter.y - (size.height / 2)
            }
            
            //setter 声明可以简写，被默认命名为newValue
            //set {
            //    origin.x = newValue.x - (size.width / 2)
            //    origin.y = newValue.y - (size.height / 2)
            //}
        }
        
        //3 只读计算属性，只有getter,并且此处getter被简写,没有setter 
        //var center: Point {
        //   let centerX = origin.x + (size.width / 2)
        //   let centerY = origin.y + (size.height / 2)
        //   return Point(x: centerX, y: centerY)
        //}
        
   
        
   }
  
  ```


### 3 属性观察者

+ 每当一个属性的值被设置时，属性观察者都会被调用，即使这个值与该属性当前的值相同。

+ 除了延迟属性，你可以为任意属性定义观察者

+ 你也可以在子类中`重新`属性为任何`继承属性`（无论是存储属性还是计算属性）添加属性观察者 

+ willSet 会在该值被存储之前被调用,` 新的属性值`会以常量形式参数传递,默认名为newValue，你也可以为它自定义参数名

+ didSet 会在一个新值被存储后被调用,  一个包含`旧属性值`的常量形式参数将会被传递,默认名为oldValue，你也可以为它自定义参数名

  ```swift
  class StepCounter {
      var totalSteps: Int = 0 {
          willSet(newTotalSteps) {
              print("About to set totalSteps to \(newTotalSteps)")
          }
          didSet {
              if totalSteps > oldValue  {
                  print("Added \(totalSteps - oldValue) steps")
              }
          }
      }
  }
  let stepCounter = StepCounter()
  stepCounter.totalSteps = 200
  // About to set totalSteps to 200
  // Added 200 steps
  stepCounter.totalSteps = 360
  // About to set totalSteps to 360
  // Added 160 steps
  stepCounter.totalSteps = 896
  // About to set totalSteps to 896
  // Added 536 steps
  ```

+ **注意**

  1. 你不需要为非重写的计算属性定义属性观察者，因为你可以在计算属性的设置器里直接观察和相应它们值的改变。 
  2. 父类属性的 willSet 和 didSet 观察者会在子类初始化器中设置时被调用。它们不会在类的父类初始化器调用中设置其自身属性时被调用。 
  3. 如果你以输入输出形式参数传一个拥有观察者的属性给函数， willSet 和 didSet 观察者一定会被调用。这是由于输入输出形式参数的拷贝入拷贝出存储模型导致的：值一定会在函数结束后写回属性。

### 4 全局和局部变量

1. `全局变量`是定义在任何函数、方法、闭包或者类型环境之外的变量。
2. `局部变量`是定义在函数、方法或者闭包环境之中的变量。
3. 计算属性和观察属性的能力`同样`对*全局变量*和*局部变量*`有效`。
4. 所谓计算变量和存储变量的概念跟计算变量和存储属性类似理解就行了
5. `全局常量和变量`永远是延迟计算的，与[延迟存储属性](https://www.cnswift.org/properties#spl-3)有着相同的行为,但是不需要加`lazy`

### 5 `类型`属性[struct, enumeration,class]

1. `实例属性`是属于特定类型实例的属性
2. `类型属性`是属于类型本身的属性
3. `存储类型属性`可以是变量或者常量。`计算类型属性`总要被声明为变量属性。这与`实例属性`一致。
4. 注意
   + 不同于存储实例属性，你必须总是给存储类型属性一个默认值。这是因为类型本身不能拥有能够在初始化时给存储类型属性赋值的初始化器。
   + 存储类型属性是在它们第一次访问时延迟初始化的。它们保证只会初始化一次，就算被多个线程同时访问，他们也不需要使用 lazy 修饰符标记。

#### 5.1 `类型`属性语法

+ 在swift中类型属性用`static`修饰，类型:如structure，enumeration,class

+ 对于`类类型(class)`的计算类型属性，你可以使用 `class` 关键字来允许`子类重写父类的实现`。

  ```swift
  struct SomeStructure {
      static var storedTypeProperty = "Some value."
      static var computedTypeProperty: Int {
          return 1
      }
  }
  enum SomeEnumeration {
      static var storedTypeProperty = "Some value."
      static var computedTypeProperty: Int {
          return 6
      }
  }
  class SomeClass {
      static var storedTypeProperty = "Some value."
      static var computedTypeProperty: Int {
          return 27
      }
      //overrideableComputedTypeProperty 该属性为只读的
      class var overrideableComputedTypeProperty: Int {
          get {
              return 100
          }
          set {
              return newValue + 10
          }
      }
  }
  ```

#### 5.2 查询和设置类型属性

+ `类型属性`直接通过`类型`使用点语法来查询和设置

  ```swift
  print(SomeStructure.storedTypeProperty)
  // prints "Some value."
  SomeStructure.storedTypeProperty = "Another value."
  print(SomeStructure.storedTypeProperty)
  // prints "Another value."
  print(SomeEnumeration.computedTypeProperty)
  // prints "6"
  print(SomeClass.computedTypeProperty)
  ```
