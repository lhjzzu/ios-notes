## 十一 方法

+ 方法是关联了`特定类型`的函数
+ `类`，`结构体`和`枚举`同样可以定义`实例方法`和`类型方法`

### 1 实例方法

+ **实例方法** 是属于特定类实例、结构体实例或者枚举实例的函数

#### 1.1 self属性

+ 使用属性和方法时，可以隐式调用
+ 如果属性名和形式参数一样，那么形式参数优先。可以通过显式的self来区分属性名和形式参数

#### 1.2 在实例方法中修改值类型

+ 结构体和枚举是值类型，默认情况下，值类型不能被自身的实例方法修改

+ 对于结构体和枚举，通过在实例方法之前放在`mutating`关键字，将这个方法变异，从而可以修改属性

  ```swift
  struct Point {
      var x = 0.0, y = 0.0
      mutating func moveBy(x deltaX: Double, y deltaY: Double) {
          x += deltaX
          y += deltaY
      }
  }
  var somePoint = Point(x: 1.0, y: 1.0)
  somePoint.moveBy(x: 2.0, y: 3.0)
  print("The point is now at (\(somePoint.x), \(somePoint.y))")
  // prints "The point is now at (3.0, 4.0)"
  ```

+ 注意：如同 [常量结构体实例的存储属性](https://www.cnswift.org/properties#spl-2) 里描述的那样，你不能在`常量结构体`类型里调用异变方法，因为自身属性不能被改变，就算它们是变量属性

  ```swift
  let fixedPoint = Point(x: 3.0, y: 3.0)
  fixedPoint.moveBy(x: 2.0, y: 3.0) //调用的时候会报错
  ```

  

#### 1.3 在异变方法里指定自身 

+ 异变方法可以指定整个实例给隐含的 self属性

  ```swift
  struct Point {
      var x = 0.0, y = 0.0
      mutating func moveBy(x deltaX: Double, y deltaY: Double) {
          self = Point(x: x + deltaX, y: y + deltaY)
      }
  }
  ```

+ 枚举的异变方法可以设置隐含的 self属性为相同`枚举里的不同成员`

  ```swift
  enum TriStateSwitch {
      case off, low, high
      mutating func next() {
          switch self {
          case .off:
              self = .low
          case .low:
              self = .high
          case .high:
              self = .off
          }
      }
  }
  var ovenLight = TriStateSwitch.low
  ovenLight.next()
  // ovenLight is now equal to .high
  ovenLight.next()
  // ovenLight is now equal to .off
  ```

  

### 2 类型方法

+ 通过在 func关键字之前使用 static关键字来明确一个`类型`方法
+ `类`可以使用 class关键字来允许子类重写父类对类型方法的实现。
+ 在oc中，你只能在类中定义类方法，但是在swift中，你可以在类，结构体和枚举中定义类型方法。
+ 类型方法中，隐藏的self指向类型本身。可以通过self，区别形式参数和类型属性
+ 在类型方法中，可以不用加类型前缀，而直接调用类型方法或使用类型属性。



## 十二 下标

> 通常下标是用来访问集合、列表或序列中元素的快捷方式

### 1 下标的语法

+ 下标脚本允许你通过在实例名后面的`方括号`内写`一个`或`多个`值对该类的实例进行查询

+ 它的语法`类似于`实例方法和计算属性

+ 使用关键字 `subscript` 来定义下标, 并且指定一个或多个输入`形式参数`和`返回类型`

+ 下标可以是读写,也可以是只读的

  ```swift
  subscript(index: Int) -> Int {
      get {
          // return an appropriate subscript value here
      }
      set(newValue) {
          // perform a suitable setting action here
      }
  }
  ```

+ 与只读计算属性一样，你可以给只读下标省略 get 关键字：

  ```swift
  subscript(index: Int) -> Int {
      // return an appropriate subscript value here
  }
  ```

  

### 2 下标的用法

- 常下标是用来访问集合、列表或序列中元素的快捷方式

- 例如对Dictionary进行下标操作

  ```swift
  var numberOfLegs = ["spider": 8， "ant": 6， "cat": 4]
  numberOfLegs["bird"] = 2
  ```

### 3 下标选项

+ 下标可以使用变量形式参数和可变形式参数，但是不能使用输入输出形式参数或提供默认形式参数值
+ 类或结构体可以根据自身需要提供多个下标实现，合适被使用的下标会基于值类型或者使用下标时下标方括号里包含的值来推断。这个对多下标的定义就是所谓的*下标重载*。？？？
+ 通常来讲下标接收一个形式参数，但只要你的类型需要也可以为下标定义多个参数



## 十三 继承

- 一个类可以从另一个类继承方法、属性和其他的特性
- 在 Swift 中类可以调用和访问属于它们父类的方法、属性和下标脚本，并且可以重写
- Swift 会通过检查`重写定义`都有一个与之匹配的`父类定义`来确保你的重写是正确的
- 类也可以向继承的任何属性(存储&计算属性)添加属性观察器，以便在属性的值改变时得到通知

### 1 定义一个基类

- 任何不从另一个类继承的类都是所谓的*基类*。

- Swift 类不会从一个通用基类继承。你没有指定特定父类的类都会以基类的形式创建。

  ```swift
  class Vehicle {
      var currentSpeed = 0.0
      var description: String {
          return "traveling at \(currentSpeed) miles per hour"
      }
      func makeNoise() {
          // do nothing - an arbitrary vehicle doesn't necessarily make a noise
      }
  }
  ```

  

### 2 子类

+ 子类是基于现有类创建新类的行为

+ 子类从现有的类继承了一些特征，你可以重新定义它们,你也可以为子类添加新的特征

  ```swift
  lass Bicycle: Vehicle {
      var hasBasket = false
  }
  ```

### 3 重写

- `重写`:除了子类自己提供的，其他从父类继承来属性和方法，子类提供自定义实现
- 重写的重要特征是:在重写定义前加上`override`关键字
- 没有使用 override 关键字的重写都会在编译时被诊断为错误
- **override** 关键字会**执行 Swift 编译器检查**你重写的类的父类(或者父类的父类)是否有**与之匹配的声明**来供你重写。这个检查确保你重写的定义是正确的。

#### 3.1 访问父类的方法、属性和下标脚本

- 你可以通过使用 super 前缀访问父类的方法、属性或下标脚本
  + 一个命名为 someMethod() 的重写方法可以通过 super.someMethod() 在重写方法的实现中调用父类版本的 someMethod() 方法；
  + 一个命名为 someProperty 的重写属性可以通过 super.someProperty 在重写的 getter 或 setter 实现中访问父类版本的someProperty 属性；
  + 一个命名为 someIndex 的重写下标脚本可以使用 super[someIndex] 在重写的下标脚本实现中访问父类版本中相同的下标脚本。

#### 3.2 重写方法

- 子类中重写一个继承的实例或类型方法来提供自定义实现

  ```swift
  class Train: Vehicle {
      override func makeNoise() {
          print("Choo Choo")
      }
  }
  ```

  

#### 3.3 重写属性

+ 重写一个继承的实例或类型属性来为你自己的属性提供你自己自定义的 getter 和 setter 

##### 3.3.1 重写属性的GETTER和SETTER

+ 必须声明你重写的属性名字和类型，以确保编译器可以检查你的重写是否匹配了父类中有相同名字和类型的属性。

+ 在你的子类重写里为继承而来的`只读属性`添加Getter和Setter来把它用作`可读写属性`

+ `不能`把一个继承而来的`可读写属性`表示为`只读属性`

+ 你提供了一个setter作为属性重写的一部分，你也就必须为重写提供一个getter

  ```swift
  class Car: Vehicle {
      var gear = 1
      override var description: String {
          return super.description + " in gear \(gear)"
      }
  }
  ```

##### 3.3.2 重写属性观察器

+ 你可以使用`属性重写`来为继承的属性添加`属性观察器`

+ 你不能给继承而来的`常量存储属性`或者`只读的计算属性`添加属性观察器

+ 你不能为同一个属性同时提供重写的setter和重写的属性观察器（因为在setter中已经可以得到值的改变）

  ```swift
  class AutomaticCar: Car {
      override var currentSpeed: Double {
          didSet {
              gear = Int(currentSpeed / 10.0) + 1
          }
      }
  }
  ```



### 4 阻止重写

- 阻止重写:通过在类以及类的扩展中的方法、属性或者下标脚本的关键字前写` final` 修饰符,此时在子类中重写时,直接编译报错。

  ```swift
  final var ， final func ， final class func ， final subscript等
  ```

  

- 类定义中在` class` 关键字前面写` final` 修饰符(` final class `)标记一整个类为终点。此时创建继承于该类的子类时，编译报错。



## 十四 初始化

- **初始化**是为类、结构体或者枚举准备实例的过程
- 需要给实例里的每一个`存储属性`设置一个`初始值`并且在新实例可以使用之前执行任何其他所必须的配置或初始化
- 定义**初始化器**来实现这个初始化过程， 不同于 Objective-C 的初始化器，Swift 初始化器不返回值

### 1 为存储属性设置初始化值

- 在创建`类`和`结构体`的实例时*必须*为所有的存储属性设置一个合适的初始值
- 你可以在初始化器里为存储属性设置一个初始值，或者通过分配一个默认的属性值作为属性定义的一部分

> 注意:
>
> 当你给一个存储属性分配默认值，或者在一个初始化器里设置它的初始值的时候，属性的值就会被直接设置，不会调用任何属性监听器。 

#### 1.1 初始化器

+ 初始化器在创建特定类型的实例时被调用。使用`init`关键字

  ```swift
  struct Fahrenheit {
      var temperature: Double
      init() {
          temperature = 32.0
      }
  }
  var f = Fahrenheit()
  print("The default temperature is \(f.temperature)° Fahrenheit")
  // prints "The default temperature is 32.0° Fahrenheit"
  ```

  

#### 1.2 默认的属性值

+ 指定一个默认属性值作为属性声明的一部分,这将属性的初始化与声明更紧密地联系到一起

  ```swift
  struct Fahrenheit {
      var temperature = 32.0
  }
  //直接在声明属性的时候，为其指定一个默认值。同时也利用了类型推断机制
  ```

  

### 2  自定义初始化

- 通过输入形式参数和可选类型来自定义初始化过程
- 在初始化的时候分配常量属性

#### 2.1 初始化形式参数

+ 你可以提供*初始化形式参数*作为初始化器的一部分，来定义初始化过程中的类型和值的名称

  ```swift
  struct Celsius {
      var temperatureInCelsius: Double
      init(fromFahrenheit fahrenheit: Double) {
          temperatureInCelsius = (fahrenheit - 32.0) / 1.8
      }
      init(fromKelvin kelvin: Double) {
          temperatureInCelsius = kelvin - 273.15
      }
  }
  let boilingPointOfWater = Celsius(fromFahrenheit: 212.0)
  // boilingPointOfWater.temperatureInCelsius is 100.0
  let freezingPointOfWater = Celsius(fromKelvin: 273.15)
  // freezingPointOfWater.temperatureInCelsius is 0.0
  
  //fromFahrenheit 实际参数标签
  //fahrenheit 局部变量名
  ```

#### 2.2 形式参数名和实际参数标签

+ 初始化的函数名是一样的都是init，所以只能通过参数名称和类型来进行不同初始化函数的区分

+ 与函数和方法一样，初始化器也可以有局部变量名以及对应的实际参数标签

+ 与函数和方法一样，如果你没有提供外部名 (实际参数标签)Swift 会自动为*每一个*形式参数提供一个外部名称。

  ```swift
  struct Color {
      let red, green, blue: Double
      init(red: Double, green: Double, blue: Double) {
          self.red   = red
          self.green = green
          self.blue  = blue
      }
      init(white: Double) {
          red   = white
          green = white
          blue  = white
      }
  }
  
  let magenta = Color(red: 1.0, green: 0.0, blue: 1.0)
  let halfGray = Color(white: 0.5)
  ```

+ 注意: 不使用外部名称是不能调用这些初始化器的，否则编译报错

  ```swift
  let veryGreen = Color(0.0, 1.0, 0.0)
  //此时编译报错
  ```



#### 2.3 无实际参数标签的初始化器形式参数

- 可以写一个下划线`_`替代明确的实际参数标签

  ```swift
  struct Celsius {
      var temperatureInCelsius: Double
      init(fromFahrenheit fahrenheit: Double) {
          temperatureInCelsius = (fahrenheit - 32.0) / 1.8
      }
      init(fromKelvin kelvin: Double) {
          temperatureInCelsius = kelvin - 273.15
      }
      //用下滑线代替明确的实际参数标签
      init(_ celsius: Double) {
          temperatureInCelsius = celsius
      }
  }
  let bodyTemperature = Celsius(37.0)
  // bodyTemperature.temperatureInCelsius is 37.0
  ```

  

#### 2.4 可选属性类型

+ 将属性声明为可选类型，则在初始化期间可以不被设置，会默认为nil

  ```swift
  class SurveyQuestion {
      var text: String
      var response: String?
      init(text: String) {
          self.text = text
      }
      func ask() {
          print(text)
      }
  }
  let cheeseQuestion = SurveyQuestion(text: "Do you like cheese?")
  cheeseQuestion.ask()
  // prints "Do you like cheese?"
  cheeseQuestion.response = "Yes, I do like cheese."
  
  //解释
  response在初始化期间设置并不合适，所以可以声明为可选类型，从而在后面合适的时机进行赋值
  ```

  

#### 2.5 在初始化中分配常量属性

+ 在初始化的任意时刻，你都可以给常量属性赋值，只要它在`初始化结束`时设置了确定的值即可

  ```swift
  class SurveyQuestion {
      let text: String
      var response: String?
      //注意: 在初始化完成的时候，确保常量属性被初始化完成
      init(text: String) {
          self.text = text
      }
      func ask() {
          print(text)
      }
  }
  let beetsQuestion = SurveyQuestion(text: "How about beets?")
  beetsQuestion.ask()
  // prints "How about beets?"
  beetsQuestion.response = "I also like beets. (But not with cheese.)"
  ```

  

### 3 默认初始化器

+ 如果一个类或结构体中**所有的属性都有了一个默认的值**，那么可以使用**默认的初始化器**进行初始化。如果并不是所有属性都有默认值，那么需要自己提供初始化器，否则会报编译错误

  ```swift
  class ShoppingListItem {
      var name: String?
      var quantity = 1
      var purchased = false
  }
  var item = ShoppingListItem()
  //此时ShoppingListItem所有属性都有默认的值
  //name = nil
  //quantity = 1
  //purchased = false
  ```

#### 3.1 结构体类型的成员初始化器

- 如果结构体类型中没有定义任何自定义初始化器，它会自动获得一个**成员初始化器**

- 不同于默认初始化器，结构体会接收成员初始化器即使**它的存储属性没有默认值。**

  ```swift
  struct Size {
      var width = 2.0, height = 2.0
  }
  let twoByTwo = Size(width: 2.0, height: 2.0)
  //width:2.0 height=3.0
  let twoByTwo2 = Size()
  //width:2.0 height=2.0
  
  
  struct Size {
      var width:Float, height:Float
  }
  //此时只有成员初始化器，没有默认初始化器
  let twoByTwo = Size(width: 2.0, height: 2.0)
  //width:2.0 height=3.0
  ```

  

### 4 值类型的初始化器委托

- **初始化器委托**:初始化器可以调用其他初始化器来执行部分实例的初始化的过程

- 如果你为值类型定义了自定义初始化器, 你就不能访问那个类型的默认初始化器(或者是成员初始化器，如果是结构体的话)

  + 目的是让别人使用指定的初始化器, 完成一些特定的配置。

  + 如果要使用默认初始化器(或者是成员初始化器)，除非你自己明确提供或者把自定义的初始化器写到扩展里

    ```swift
    struct Size {
        var width = 0.0, height = 0.0
    }
    struct Point {
        var x = 0.0, y = 0.0
    }
    
    struct Rect {
        var origin = Point()
        var size = Size()
        //自己提供默认的初始化器
        init() {}
        //自己提供成员初始化器
        init(origin: Point, size: Size) {
            self.origin = origin
            self.size = size
        }
        //自己提供的自定义的初始化器
        init(center: Point, size: Size) {
            let originX = center.x - (size.width / 2)
            let originY = center.y - (size.height / 2)
            self.init(origin: Point(x: originX, y: originY), size: size)
        }
    }
    ```

### 5 类的继承和初始化

> 所有类的存储属性:都必须在初始化期间分配初始值

#### 5.1 指定初始化器和便捷初始化器

- 指定初始化器是类的主要初始化器
- 指定初始化器初始化该类，并调用父类的初始化器，形成初始化链
- 每个类`至少有一个`指定初始化器
- 便捷初始化器是次要的，最终调用指定初始化器，可以提供一些默认值，为了方便调用指定初始化器。



#### 5.2 指定初始化器和便捷初始化器语法

+ 指定初始化器

  ```swift
  init(parameters) {
      statements
  }
  ```

+ 便捷初始化器，要使用`convenience` 修饰符

  ```swift
  convenience init(parameters) {
      statements
  }
  ```

#### 5.3 类类型的初始化器委托

+ 指定和便捷初始化器之间的调用关系，有三条规则
  1. 子类的指定初始化器必须调用它的直系父类的指定初始化器。
  2. 便捷初始化器必须从相同的类里调用另一个初始化器。
  3. 便捷初始化器最终必须调用一个指定初始化器。
     + 指定初始化器必须总是向上委托。
     + 便捷初始化器必须总是横向委托。

#### 5.4 两段式初始化

+ Swift 的类初始化是一个两段式过程

  + 在第一个阶段

    ```
    每一个存储属性(该类的属性以及继承的父类的属性)被引入类为分配了一个初始值
    ```

  + 在第二阶段

    ```
    每个类都有机会在新的实例准备使用之前来定制它的存储属性
    ```

    

+ Swift编译器执行`四种`有效的`安全检查`来确保两段式初始化过程能够顺利完成(不符合编译器就报错)

  1. 指定初始化器必须保证在向上委托给父类初始化器之前，其所在类引入的所有属性都要初始化完成。

     ```
     原因: 如上所述，一个对象的内存只有在其所有储存型属性确定之后才能完全初始化。为了满足这一规则，指定初始化器必须保证它自己的属性在它上交委托之前先完成初始化。
     ```

  2. 指定初始化器必须先向上委托父类初始化器，然后才能为继承的属性设置新值

     ```
     原因: 如果不这样做，指定初始化器赋予的新值将被父类中的初始化器所覆盖。
     ```

  3. 便捷初始化器必须先委托同类中的其它初始化器，然后再为任意属性赋新值

     ```
     原因: 如果没这么做，便捷构初始化器赋予的新值将被自己类中其它指定初始化器所覆盖。
     ```

  4. 初始化器在第一阶段初始化完成之前，不能调用任何实例方法、不能读取任何实例属性的值，也不能引用 self 作为值

     ```
     原因: 直到第一阶段结束的时候，这个类实例才被看做是合法的。
     ```

+ 阶段1 - 完成初始化

  - 指定或便捷初始化器在类中被调用；
  - 为这个类的新实例分配内存。内存还没有被初始化；
  - 这个类的指定初始化器确保所有由此类引入的存储属性都有一个值。现在这些存储属性的内存被初始化了；
  - 指定初始化器上交父类的初始化器为其存储属性执行相同的任务；
  - 这个调用父类初始化器的过程将沿着初始化器链一直向上进行，直到到达初始化器链的最顶部；
  - 一旦达了初始化器链的最顶部，在链顶部的类确保所有的存储属性都有一个值，此实例的内存被认为完全初始化了，此时第一阶段完成。

+ 阶段2 - 有机会定制属性

  - 从顶部初始化器往下，链中的每一个指定初始化器都有机会进一步定制实例。初始化器现在能够访问 self 并且可以修改它的属性，调用它的实例方法等等；
  - 最终，链中任何便捷初始化器都有机会定制实例以及使用 self 。

+ 便捷初始化器和指定初始化器的理解

  + 便捷初始化器，先调用指定初始化器，然后再定制属性

  + 指定初始化器，先初始化该类定义的存储属性，然后调用父类的指定初始化器，然后再定制属性。

    

#### 5.5 初始化器的继承和重写

+ Swift 的子类不会默认继承父类的初始化器-----这种机制防止子类没有被完全初始化。
+ 只有在特定情况下才会继承父类的初始化器，但只有这样是安全且合适的时候。
+ 当重写父类指定初始化器时，你必须写 override 修饰符
+ 当提供一个匹配的父类便捷初始化器的实现时，你不用写 override 修饰符。

#### 5.6 初始化器的自动继承

+ 在特定的情况下父类初始化器*是*可以被自动继承的
  + 假设你为你子类引入的任何新的属性都提供了默认值， 请遵守以下2个规则：
    1. 如果你的子类没有定义任何指定初始化器，它会自动继承父类所有的指定初始化器。
    2. 如果你的子类提供了*所有*父类指定初始化器的实现——要么是通过规则1继承来的，要么通过在定义中提供自定义实现的——那么它自动继承所有的父类便捷初始化器。



#### 5.7 指定和便捷初始化器的实战

[实战示例](https://www.cnswift.org/initialization#spl-20)



### 6 可失败初始化器

- 通过`init?`, 在类、结构体或枚举中定义一个或多个可失败的初始化器。

- 可失败的初始化器:主要用来处理一些初始化可能失败的情况

  ```
  1 给初始化传入无效的形式参数值
  2 缺少某种外部所需的资源
  3 或是其他阻止初始化的情况
  ```

- 可失败的初始化器创建了一个初始化类型的`可选值`。

- 严格来讲，初始化器不会有返回值。初始化的作用是确保在初始化结束时， self 能够被正确初始化。

- 使用`return nil`来触发初始化失败。但是你不能使用return 关键字来表示初始化成功了

  ```swift
  struct Animal {
      let species: String
      //可失败的初始化器
      init?(species: String) {
          if species.isEmpty { return nil }
          self.species = species
      }
  }
  
  let someCreature = Animal(species: "Giraffe")
  // someCreature is of type Animal?, not Animal
  if let giraffe = someCreature {
      print("An animal was initialized with a species of \(giraffe.species)")
  }
  // prints "An animal was initialized with a species of Giraffe"
  ```

#### 6.1 枚举的可失败初始化器

+ 定义一个枚举可失败初始化器

  ```swift
  enum TemperatureUnit {
      case Kelvin, Celsius, Fahrenheit
      init?(symbol: Character) {
          switch symbol {
          case "K":
              self = .Kelvin
          case "C":
              self = .Celsius
          case "F":
              self = .Fahrenheit
          default:
              return nil
          }
      }
  }
  
  let fahrenheitUnit = TemperatureUnit(symbol: "F")
  if fahrenheitUnit != nil {
      print("This is a defined temperature unit, so initialization succeeded.")
  }
  // prints "This is a defined temperature unit, so initialization succeeded."
  let unknownUnit = TemperatureUnit(symbol: "X")
  if unknownUnit == nil {
      print("This is not a defined temperature unit, so initialization failed.")
  }
  // prints "This is not a defined temperature unit, so initialization failed."
  ```

  

#### 6.2 带有原始值枚举的可失败初始化器

+ 带有原始值的枚举会自动获得一个可失败初始化器 `init?(rawValue:)`

  ```swift
  enum TemperatureUnit: Character {
      case Kelvin = "K", Celsius = "C", Fahrenheit = "F"
  }
   
  let fahrenheitUnit = TemperatureUnit(rawValue: "F")
  if fahrenheitUnit != nil {
      print("This is a defined temperature unit, so initialization succeeded.")
  }
  // prints "This is a defined temperature unit, so initialization succeeded."
   
  let unknownUnit = TemperatureUnit(rawValue: "X")
  if unknownUnit == nil {
      print("This is not a defined temperature unit, so initialization failed.")
  }
  // prints "This is not a defined temperature unit, so initialization failed."
  ```

  

#### 6.3 初始化失败的传递

- 类，结构体或枚举的可失败初始化器可以横向委托到同一个类，结构体或枚举里的另一个可失败初始化器。

- 子类的可失败初始化器可以向上委托到父类的可失败初始化器。

- 如果委托到另一个初始化器导致了初始化失败，那么整个初始化过程也会立即失败，并且后续任何初始化代码都不会执行。

- 可失败初始化器也可以委托其他的非可失败初始化器。通过这个方法，你可以为已有的初始化过程添加初始化失败的条件。

  ```swift
  class Product {
      let name: String
      //可失败初始化器
      init?(name: String) {
          if name.isEmpty { return nil }
          self.name = name
      }
  }
   
  class CartItem: Product {
      let quantity: Int
      //可失败初始化器
      init?(name: String, quantity: Int) {
          if quantity < 1 { return nil }
          self.quantity = quantity
          //调用父类的可失败初始化器
          super.init(name: name)
      }
  }
  
  //====== 情况1 quantity 的值为 0 ， CartItem 初始化器会导致初始化失败,后续代码不会执行
  if let zeroShirts = CartItem(name: "shirt", quantity: 0) {
      print("Item: \(zeroShirts.name), quantity: \(zeroShirts.quantity)")
  } else {
      print("Unable to initialize zero shirts")
  }
  // prints "Unable to initialize zero shirts"
  
  //======= 情况2 name 为空值，那么父类 Product 的初始化器就会导致初始化失败,后续代码不会执行
  
  if let oneUnnamed = CartItem(name: "", quantity: 1) {
      print("Item: \(oneUnnamed.name), quantity: \(oneUnnamed.quantity)")
  } else {
      print("Unable to initialize one unnamed product")
  }
  // prints "Unable to initialize one unnamed product"
  ```

  

#### 6.4 重写可失败初始化器

+ 可以在子类里重写父类的可失败初始化器

+ 可以用**子类的非可失败初始化器**来重写**父类可失败初始化器**，则向上委托到父类初始化器的唯一办法是强制展开父类可失败初始化器的结果。

+ 不可以用**子类的可失败初始化器**来重写**父类非可失败初始化器**

  ```swift
  class Document {
      var name: String?
      // this initializer creates a document with a nil name value
      init() {}
      // this initializer creates a document with a non-empty name value
      init?(name: String) {
          self.name = name
          if name.isEmpty { return nil }
      }
  }
  
  class AutomaticallyNamedDocument: Document {
      override init() {
          super.init()
          self.name = "[Untitled]"
      }
      
      //不可失败的初始化器
      override init(name: String) {
          //此时调用的是父类的不可失败的初始化器
          super.init()
          if name.isEmpty {
              self.name = "[Untitled]"
          } else {
              self.name = name
          }
      }
  }
  
  ```

+ 初始化器里使用`强制展开`来从父类调用一个可失败初始化器作为子类非可失败初始化器的一部分

  ```swift
  class AutomaticallyNamedDocument: Document {
      override init() {
          super.init()
          self.name = "[Untitled]"
      }
      
      //不可失败的初始化器
      override init(name: String) {
          //此时调用的是父类的可失败的初始化器
          super.init(name: "[Untitled]")!
          self.name = name
      }
  }
  
  ```

  

#### 6.5 可失败初始化器 init!

+ 当 init! 初始化器导致初始化失败时会触发断言。

### 7 必要初始化器

+ 在类的初始化器前添加 `required  `修饰符来表明所有该类的`子类`都必须实现该初始化器

+ 子类重写父类的必要初始化器时，不需要`override`修饰

+ 如果子类继承的初始化器能够满足需求，则你无需显式地在子类中提供必要初始化器的实现

  ```swift
  class SomeSubclass: SomeClass {
      required init() {
          // subclass implementation of the required initializer goes here
      }
  }
  ```

### 8 通过闭包和函数来设置属性的默认值

+ 可以使用闭包或全局函数来为属性提供默认值

+ 当这个属性属于的实例初始化时，闭包或函数就会被调用，并且它的返回值就会作为属性的默认值。

  ```swift
  class SomeClass {
      let someProperty: SomeType = {
          // create a default value for someProperty inside this closure
          // someValue must be of the same type as SomeType
          return someValue
      }()
  }
  ```

+ 注意:此时self还没有被实例化，所以你不能在闭包中使用其他属性，或者调用实例方法

  ```swift
  struct Chessboard {
      //使用闭包提供默认值
      let boardColors: [Bool] = {
          var temporaryBoard = [Bool]()
          var isBlack = false
          for i in 1...8 {
              for j in 1...8 {
                  temporaryBoard.append(isBlack)
                  isBlack = !isBlack
              }
              isBlack = !isBlack
          }
          return temporaryBoard
      }()
      func squareIsBlackAt(row: Int, column: Int) -> Bool {
          return boardColors[(row * 8) + column]
      }
  }
  
  let board = Chessboard()
  print(board.squareIsBlackAt(row: 0, column: 1))
  // Prints "true"
  print(board.squareIsBlackAt(row: 7, column: 7))
  // Prints "false"
  ```

  

## 十五 反初始化

- 在类实例被释放的时候，反初始化器就会立即被调用
- 用 `deinit` 关键字来写反初始化器

### 1 反初始化器原理

+ 反初始化器(`deinit`)会在实例被释放之前自动被调用,不能自行调用反初始化器

+ Swift 通过自动引用计数（ARC）来处理实例的内存管理。基本上，当你的实例被释放时，你不需要手动清除它们实例。

+ 你可能需要在实例释放之前在`deinit`中执行一些额外的清理工作。

+ **先调用子类**的deinit，**再调用父类**的deinit。

  + 在oc中也是如此，**先调用子类**的dealloc，**再调用父类**的dealloc。

+ 父类的反初始化器总会被调用，就算子类没有反初始化器

  ```swift
  class BaseObject:NSObject {
      override init() {
          super.init()
      }
      
      deinit {
          print("BaseObject deinit")
      }
  }
  
  
  class TestObject: BaseObject {
      override init() {
          super.init()
      }
      
      deinit {
          print("TestObject deinit")
      }
  }
  
  var  testObject:TestObject? = TestObject()
  testObject = nil
  
  //先调用子类的deinit，再调用父类的deinit。
  //prints: TestObject deinit
  //prints: BaseObject deinit
  ```

  

## 十六 引用计数

- swift 使用引用计数机制(ARC)来管理你的内存，所以大多数情况下，我们不需要考虑内存管理
- 在少数情况下，为了能帮助你管理内存，ARC需要更多关于你代码之间的关系信息。

### 1 ARC的工作机制

+ 无论你将实例分配给属性，常量或变量，它们都会创建该实例的**强引用**。
+ 如果 ARC 释放了正在使用的实例内存,此后再去访问其属性和方法，很可能造成你的app很可能会崩溃。

### 2 ARC

+ 例子展示了自动引用计数的工作机制

  ```swift
  class Person {
      let name: String
      init(name: String) {
          self.name = name
          print("\(name) is being initialized")
      }
      deinit {
          print("\(name) is being deinitialized")
      }
  }
  
  
  var reference1: Person?
  var reference2: Person?
  var reference3: Person?
  
  reference1 = Person(name: "John Appleseed")
  reference2 = reference1
  reference3 = reference1
  
  有三个强引用，只有三个强引用全部断开时，实例才释放
  reference1 = nil
  reference2 = nil
  reference3 = nil
  ```



### 3 类实例之间的循环强引用

- 如果两个类实例彼此持有对方的强引用,这就是循环引用

  ```swift
  class Person {
      let name: String
      init(name: String) { self.name = name }
      var apartment: Apartment?
      deinit { print("\(name) is being deinitialized") }
  }
   
  class Apartment {
      let unit: String
      init(unit: String) { self.unit = unit }
      var tenant: Person?
      deinit { print("Apartment \(unit) is being deinitialized") }
  }
  
  var john: Person?
  var unit4A: Apartment?
  john = Person(name: "John Appleseed")
  unit4A = Apartment(unit: "4A")
  john!.apartment = unit4A
  unit4A!.tenant = john
  
  //即使都变为nil也不能释放,因为相互强引用造成循环，无法释放
  john = nil
  unit4A = nil
  ```

  

### 4 解决实例之间的循环强引用

+ 弱引用（ weak reference ）和无主引用（ unowned reference )解决实例之间循环强引用

+ 在实例的生命周期中，当引用可能“没有值”的时候，就使用weak来避免循环引用

+ 如果引用始终有值，则可以使用unowned来代替

  

#### 4.1 弱引用

+ ARC 会在被引用的实例被释放是自动地设置弱引用为 nil，避免了野指针问题

+ 由于弱引用需要允许它们的值为 nil ，它们一定得是可选类型

+  ARC 给弱引用设置 nil 时不会调用属性观察者。

+ 在实例的生命周期中，当引用可能“没有值”的时候，就使用weak来避免循环引用

  ```swift
  
  class Person {
      let name: String
      init(name: String) { self.name = name }
      var apartment: Apartment?
      deinit { print("\(name) is being deinitialized") }
  }
   
  class Apartment {
      let unit: String
      init(unit: String) { self.unit = unit }
      weak var tenant: Person?
      deinit { print("Apartment \(unit) is being deinitialized") }
  }
  
  //Apartment中可能没有tenant，那么使用weak来避免循环引用
  ```

  

  

#### 4.2 无主引用

- 无主引用假定是*永远*有值的

- 无主引用总是被定义为非可选类型

- ARC 无法在实例被释放后将无主引用设为 nil

- 如果你试图在实例的被释放后访问无主引用，那么你将触发运行时错误。只有在你确保引用会*一直*引用实例的时候才使用无主引用。

- 引用始终有值，则可以使用unowned来代替

  ```swift
  class Customer {
      let name: String
      var card: CreditCard?
      init(name: String) {
          self.name = name
      }
      deinit { print("\(name) is being deinitialized") }
  }
   
  class CreditCard {
      let number: UInt64
      unowned let customer: Customer
      init(number: UInt64, customer: Customer) {
          self.number = number
          self.customer = customer
      }
      deinit { print("Card #\(number) is being deinitialized") }
  }
  
  var john: Customer?
  john = Customer(name: "John Appleseed")
  john!.card = CreditCard(number: 1234_5678_9012_3456， customer: john!)
  
  john = nil
  // prints "John Appleseed is being deinitialized"
  // prints "Card #1234567890123456 is being deinitialized"
  
  //john对Customer实例强引用,Customer实例对card实例强引用，CreditCard实例对customer实例无主引用
  //john = nil时，打断了对Customer实例的强引用，Customer实例没有被任何变量强引用，所以Customer实例释放，Customer实例释放后，此时也没有对CreditCard实例引用，所以CreditCard实例释放。所以Customer的deinit先执行，CreditCard实例的deinit后执行
  ```

  - 实例通过它的属性持有别的对象，只有当这个实例完全释放后，才失去对其他类的实例的持有，其他类的实例才会被释放

    ```swift
    class A {
        let b = B()
        let c = C()
        deinit {
            print(A deinit)
        }
    }
    
    class B {
      deinit {
           print(B deinit)
      }
    }
    
    class C {
        deinit {
          print(C deinit)
      }
    }
    
    let a = A()
    a = nil
    
    //此时A先释放，然后B,C释放
    
    //print(A deinit)
    //print(B deinit)
    //print(C deinit)
    ```

- 使用 unowned(unsafe) 来明确使用了一个不安全无主引用。如果你在实例的引用被释放后访问这个不安全无主引用，你的程序就会尝试访问这个实例曾今存在过的内存地址，这就是不安全操作

#### 4.3  无主引用和隐式展开的可选属性

1. Person 和 Apartment 的例子展示了两个属性的值都允许为 nil ，并会潜在的产生循环强引用。这种场景最适合用弱引用来解决。

2. Customer 和 CreditCard 的例子展示了一个属性的值允许为 nil ，而另一个属性的值不允许为 nil ，这也可能导致循环强引用。这种场景最好使用无主引用来解决

3. 两个属性都必须有值，并且初始化完成后永远不会为 nil 。在这种场景中，需要一个类使用无主属性，而另外一个类使用隐式展开的可选属性

   - 一旦初始化完成，这两个属性能被直接访问(不需要可选展开)，同时避免了循环引用

     ```swift
     class Country {
         let name: String
         var capitalCity: City!
         init(name: String, capitalName: String) {
             self.name = name
             self.capitalCity = City(name: capitalName, country: self)
         }
     }
      
     class City {
         let name: String
         unowned let country: Country
         init(name: String, country: Country) {
             self.name = name
             self.country = country
         }
     }
     
     var country = Country(name: "Canada", capitalName: "Ottawa")
     print("\(country.name)'s capital city is called \(country.capitalCity.name)")
     // prints "Canada's capital city is called Ottawa"
     
     //我们来看这个初始化器的执行流程
     1 根据两段初始化器原理:必须要先将实例的存储属性进行初始化赋值,然后我们用self代表这个实例，作为值传递或者调用方法
     2 先初始化name, capitalCity是隐式可选项所以默认值为nil.所以在初始化完name后，self就已经被初始化完成了，所以此后Country的初始化器就能引用并传递隐式的self
     3 通过一条语句同时创建 Country 和 City 的实例，而不产生循环强引用，并且 capitalCity 的属性能被直接访问，而不需要通过感叹号来展开它的可选值
     4 使用隐式展开的可选属性的意义在于满足了两段式类初始化器的需求
     5 使用unowned避免了循环引用
     ```

### 5 闭包的循环强引用

+ 闭包和实例相互强引用最终造成循环强引用，无法释放

  ```swift
  class HTMLElement {
      
      let name: String
      let text: String?
      
      lazy var asHTML: () -> String = {
          if let text = self.text {
              return "<\(self.name)>\(text)</\(self.name)>"
          } else {
              return "<\(self.name) />"
          }
      }
      
      init(name: String, text: String? = nil) {
          self.name = name
          self.text = text
      }
      
      deinit {
          print("\(name) is being deinitialized")
      }
      
  }
  
  
  var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
  print(paragraph!.asHTML())
  
  paragraph = nil
  //因为彼此强引用，所以实例无法释放，无法执行deinit方法
  ```

  

### 6 解决闭包的循环强引用

+ 通过定义`捕获列表`作为闭包的定义来解决在闭包和类实例之间的循环强引用

+ `捕获列表`定义了当在闭包体里`捕获`一个或多个引用类型的`规则`

+ `捕获列表`中声明每个捕获的引用为弱引用或无主引用

+ 根据代码关系来决定使用弱引用还是无主引用

+ Swift 要求你在闭包中引用self成员时**使用** `self.someProperty `或者 `self.someMethod `（**而不只是** `someProperty `或 `someMethod` ）。这有助于提醒你可能会一不小心就捕获了 self 。

  + 把捕获列表放在形式参数和返回类型前边

    ```swift
    lazy var someClosure: (Int, String) -> String = {
        [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
        // closure body goes here
    }
    ```

  + 捕获列表放在关键字 in 前边（形式参数列表和返回值类型会根据上下文进行推断）

    ```swift
    lazy var someClosure: () -> String = {
        [unowned self, weak delegate = self.delegate!] in
        // closure body goes here
    }
    ```

#### 6.1 弱引用和无主引用

+ 在`闭包`和`闭包捕获的实例`总是互相引用并且总是同时释放时，将闭包内的捕获定义为无主引用。

+ 在`被捕获的引用`可能会变为 nil 时，定义一个弱引用的捕获。`弱引用总是可选项`，当实例的引用释放时会`自动变为 nil`。

+ **注意**: nil 不是指针，它是值缺失的一种特殊类型。可选类型可以被置为nil，作为它的值。但是非可选类型，即使是被释放时，其值也不是nil。

  ```swift
  class HTMLElement {
      
      let name: String
      let text: String?
      
      lazy var asHTML: () -> String = {
      //无主引用避免了循环引用
          [unowned self] in
          if let text = self.text {
              return "<\(self.name)>\(text)</\(self.name)>"
          } else {
              return "<\(self.name) />"
          }
      }
      
      init(name: String, text: String? = nil) {
          self.name = name
          self.text = text
      }
      
      deinit {
          print("\(name) is being deinitialized")
      }
  }
  ```

  

## 十七 可选链

+ 可选链是一个调用和查询可选属性、方法和下标的`过程`，它可能为 nil
+ 如果`可选项包含值`，那么属性、方法或者下标的调用成功
+ 如果`可选项是 nil `，那么属性、方法或者下标的调用`会返回 nil`
+ 多个查询可以链接在一起，如果链中`任何一个节点是 nil` ，那么整个链就会得体地`失败`
+ 注意:对可选项值为nil时，使用`!`强制展开时触发运行时错误。

### 1 可选链代替强制展开

+ 可选链调用的结果一定是一个`可选值`，就算你查询的属性、方法或者下标返回的是非可选值

+ 可选链调用的结果与期望的返回值`类型相同`，只是包装成了`可选项`。例如返回Int类型时，通过可选链调用返回Int?

  ```swift
  class Person {
      var residence: Residence?
  }
   
  class Residence {
      var numberOfRooms = 1
  }
  
  let john = Person()
  //可选链调用返回可选项
  if let roomCount = john.residence?.numberOfRooms {
      print("John's residence has \(roomCount) room(s).")
  } else {
      print("Unable to retrieve the number of rooms.")
  }
  // Prints "Unable to retrieve the number of rooms."
  
  // 最好不要使用下面的方式，因为如果residence值为nil时，触发运行时错误
  let roomCount = john.residence!.numberOfRooms
  ```

### 2 为可选链定义模型类

+ 可以使用可选链来调用属性、方法和下标不止一个层级

### 3 通过可选链访问属性

+ 你可以使用可选链来访问可选值里的属性，并且检查这个属性的访问是否成功。

+ 获取属性值

  ```swift
  let john = Person()
  if let roomCount = john.residence?.numberOfRooms {
      print("John's residence has \(roomCount) room(s).")
  } else {
      print("Unable to retrieve the number of rooms.")
  }
  ```

+ 给属性赋值->如果中间失败，后面的不会被调用。例如等号后，不会被调用。

  ```swift
  func createAddress() -> Address {
      print("Function was called.")
      
      let someAddress = Address()
      someAddress.buildingNumber = "29"
      someAddress.street = "Acacia Road"
      
      return someAddress
  }
  john.residence?.address = createAddress()
  ```

+ 任何通过可选链设置属性的尝试都会返回一个` Void?` 类型值

  ```swift
  
  if (john.residence?.address = someAddress) != nil {
      print("It was possible to set the address.")
  } else {
      print("It was not possible to set the address.")
  }
  // Prints "It was not possible to set the address."
  ```

  

### 4 通过可选链调用方法

+ 你可以使用可选链来调用可选项里的方法，并且检查调用是否成功

+ 对于函数和方法来说，如果没有返回值那么久隐式的指明它的返回值为Void类型，即值为()或空元组

  ```swift
  Residence 类中的 printNumberOfRooms() 方法,就可认为其返回值为Void类型
  func printNumberOfRooms() {
      print("The number of rooms is \(numberOfRooms)")
  }
  ```

+ 所以对于没有返回值的函数和方法，用可选链进行调用时其返回值类型为`Void?`

  ```swift
  if john.residence?.printNumberOfRooms() != nil {
      print("It was possible to print the number of rooms.")
  } else {
      print("It was not possible to print the number of rooms.")
  }
  // Prints "It was not possible to print the number of rooms."
  ```



### 5 通过可选链访问下标

- 你可以使用可选链来给可选项下标取回或设置值，并且检查下标的调用是否成功

- 使用可选链访问下标取值(`?`要在下标符号`[]`之前，因为要先访问residence是可选值，再访问其下标 )

  ```swift
  if let firstRoomName = john.residence?[0].name {
      print("The first room name is \(firstRoomName).")
  } else {
      print("Unable to retrieve the first room name.")
  }
  // Prints "Unable to retrieve the first room name."
  ```

- 通过在可选链里用下标来设置一个新值

  ```swift
  john.residence?[0] = Room(name: "Bathroom")
  ```

#### 5.1 访问可选类型的下标

+ Dictionary访问值，得到的是可选类型

  ```swift
  var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
  testScores["Dave"]?[0] = 91
  testScores["Bev"]?[0] += 1
  testScores["Brian"]?[0] = 72
  // the "Dave" array is now [91, 82, 84] and the "Bev" array is now [80, 94, 81]
  ```

### 6 链的多层连接

- 多层可选链不会给返回的值添加多层的可选性。

  + 如果你尝试通过可选链取回一个 Int 值，就一定会返回 Int? ，不论通过了多少层的可选链；

  + 如果你尝试通过可选链访问 Int? 值， Int? 一定就是返回的类型，无论通过了多少层的可选链

    ```swift
    //此时访问street得到的类型是String?
    if let johnsStreet = john.residence?.address?.street {
        print("John's street name is \(johnsStreet).")
    } else {
        print("Unable to retrieve the address.")
    }
    // Prints "Unable to retrieve the address."
    ```

#### 6.1 用可选返回值链接方法

- 你还可以通过可选链来调用返回可选类型的方法，并且如果需要的话可以继续对方法的返回值进行链接。

  ```swift
  if let beginsWithThe =
      john.residence?.address?.buildingIdentifier()?.hasPrefix("The") {
          if beginsWithThe {
              print("John's building identifier begins with \"The\".")
          } else {
              print("John's building identifier does not begin with \"The\".")
          }
  }
  ```

  

## 十八 错误处理

>  错误处理是响应和接收来自你程序中错误条件的过程

### 1 表示和抛出错误

- 在 Swift 中，错误表示为遵循 Error协议类型的值

- 这个空的协议明确了一个类型可以用于错误处理

  ```swift
  enum VendingMachineError: Error {
      case invalidSelection
      case insufficientFunds(coinsNeeded: Int)
      case outOfStock
  }
  
  throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
  
  //1 通过使枚举遵守Error协议来定义错误类型
  //2 通过throw来抛出指定的错误
  ```

### 2 处理错误

#### 2.1 使用抛出函数传递错误

- 为了明确一个函数或者方法可以抛出错误，你要在它的声明当中的`形式参数后`边写上` throws`关键字

  ```swift
   //能抛出错误的函数声明
    func canThrowErrors() throws -> String
  
    
    //不能抛出错误的函数声明
  
    func cannotThrowErrors() -> String
  
  ```

- 使用 throws标记的函数叫做**抛出函数** 

- 抛出函数可以把它内部抛出的错误传递到它被调用的生效范围之内

- 只有抛出函数能够抛出错误。任何非抛出函数中抛出的错误都必须在该函数内部处 

- ```swift
    struct Item {
        var price: Int
        var count: Int
    }
    class VendingMachine {
        var inventory = [
            "Candy Bar": Item(price: 12, count: 7),
  
            "Chips": Item(price: 10, count: 4),
  
            "Pretzels": Item(price: 7, count: 11)
        ]
        var coinsDeposited = 0
        func vend(itemNamed name: String) throws {
            guard let item = inventory[name] else {
                throw VendingMachineError.invalidSelection
            }
            guard item.count > 0 else {
                throw VendingMachineError.outOfStock
            }
            guard item.price <= coinsDeposited else {
                throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
            }
            coinsDeposited -= item.price
            var newItem = item
            newItem.count -= 1
            inventory[name] = newItem
            print("Dispensing (name)")
        }
    }
  ```

  + vend(itemNamed:)方法的实现使用了 guard语句来提前退出并抛出错误

  + 由于 vend(itemNamed:) 方法会抛出错误，调用的时候要在前边用 try关键字。

  + vend(itemNamed:)方法传递它抛出的任何错误,所以

    * 你可以直接处理错误:使用` do-catch`语句，try?或者 try!

    * 或者继续传递错误

      ```swift
       let favoriteSnacks = [
                "Alice": "Chips",
                "Bob": "Licorice",
                "Eve": "Pretzels",
            ]
            func buyFavoriteSnack(person: String, vendingMachine: VendingMachine) throws {
      
                let snackName = favoriteSnacks[person] ?? "Candy Bar"
                try vendingMachine.vend(itemNamed: snackName)
            }
            //vend继续把错误传递给buyFavoriteSnack函数来抛出
        
      ```

#### 2.2 使用 Do-Catch 处理错误 ????????????? 

- 使用 do-catch语句来通过运行一段代码处理错误

  ```swift
  do {
  
            try expression
  
            statements
  
        } catch pattern 1 {
  
            statements
  
        } catch pattern 2 where condition {
  
            statements
  
        }
  
  ```

- 面的代码用来匹配 VendingMachineError 枚举里的所有三个错误

  ```swift
  var vendingMachine = VendingMachine()
  vendingMachine.coinsDeposited = 8
  do {
      try buyFavoriteSnack("Alice", vendingMachine: vendingMachine)
      // Enjoy delicious snack
  } catch VendingMachineError.invalidSelection {
      print("Invalid Selection.")
  } catch VendingMachineError.outOfStock {
      print("Out of Stock.")
  } catch VendingMachineError.insufficientFunds(let coinsNeeded) {
      print("Insufficient funds. Please insert an additional \(coinsNeeded) coins.")
  }
  // prints "Insufficient funds. Please insert an additional 2 coins."
  ```

- 如果抛出错误，执行会立即切换到 catch分句，它决定是否传递来继续

- 如果没有错误抛出， do语句中剩下的语句将会被执行。

- 如果没有 catch分句能处理这个错误，那错误就会传递到周围的生效范围当中。总之，错误必须得在周围*某个*范围内得到处理

- 在不抛出错误的函数中， do-catch 分句就必须处理错误。在可抛出函数中，要么 do-catch 分句处理错误，要么调用者处理。如果错误被传递到了顶层生效范围但还是没有被处理，你就会得到一个运行时错误了

  

  

#### 2.3 转换错误为可选项

- 使用 `try?`通过将错误转换为可选项来处理一个错误

  ```swift
  func fetchDataFromDisk() throws -> Data {
     
  }
  func fetchData() -> Data? {
      if let data = try? fetchDataFromDisk() { return data }
      if let data = try? fetchDataFromServer() { return data }
      return nil
  }
  
  通过try?将函数的返回值转化为可选项，如果函数发生错误，那么就返回nil
  ```



#### 2.4 取消错误传递

- 对于在某些情况下，你已经确定知道不会抛出错误的方法，  用`try!`来取消错误传递

  ````swift
  let photo = try! loadImage("./Resources/John Appleseed.jpg")
  ````



### 3 指定清理操作

+ 使用 defer语句来在代码离开当前代码块前执行语句合集

  ```swift
  func processFile(filename: String) throws {
      if exists(filename) {
          let file = open(filename)
          defer {
              close(file)
          }
          while let line = try file.readline() {
              // Work with the file.
          }
          // close(file) is called here, at the end of the scope.
      }
  }
  ```

+ 延迟的操作与其指定的顺序相反执行

+ 第二个defer指定的代码在第一个defer指定的代码之前执行