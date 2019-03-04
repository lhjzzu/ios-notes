## 十九 类型转换

+ Swift 中类型转换的实现为` is `和 `as `操作符
+ `is `用来检查一个值的类型
+ `as`将某个值转换为另一种类型

### 1 为类型转换定义类层次

- 仅仅是为下面的讲解定义示例代码

  ```swift
  //仅仅是为下面定义示例代码
  class MediaItem {
      var name: String
      init(name: String) {
          self.name = name
      }
  }
  
  class Movie: MediaItem {
      var director: String
      init(name: String, director: String) {
          self.director = director
          super.init(name: name)
      }
  }
   
  class Song: MediaItem {
      var artist: String
      init(name: String, artist: String) {
          self.artist = artist
          super.init(name: name)
      }
  }
  ```


### 2 类型检查

- **类型检查操作符** （ `is` ）来检查一个实例是否属于一个特定的子类

- 如果实例是该子类类型，类型检查操作符返回true ，否则返回 false 。

   ```swift
   var movieCount = 0
   var songCount = 0
   -   for item in library {
         if item is Movie {
             movieCount += 1
         } else if item is Song {
             songCount += 1
         }
     }
   
     print("Media library contains (movieCount) movies and (songCount) songs")
     // Prints "Media library contains 2 movies and 3 songs"
   
   
   ```


### 3 向下类型转换

- 某个类类型的常量或变量可能实际上在后台引用自一个`子类`的实例

- 可以尝试使用**类型转换操作符**（` as?` 或` as! `）将它`向下类型转换`至其子类类型

- **条件形式**: as? ，返回了一个你将要向下类型转换的值的可选项。如果出错，返回值为nil。

- **强制形式**:as! ，则将向下类型转换和强制展开结合为一个步骤。如果出错，会运行时报错。

  ```swift
  for item in library {
      if let movie = item as? Movie {
          print("Movie: '\(movie.name)', dir. \(movie.director)")
      } else if let song = item as? Song {
          print("Song: '\(song.name)', by \(song.artist)")
      }
  }
   
  // Movie: 'Casablanca', dir. Michael Curtiz
  // Song: 'Blue Suede Shoes', by Elvis Presley
  // Movie: 'Citizen Kane', dir. Orson Welles
  // Song: 'The One And Only', by Chesney Hawkes
  // Song: 'Never Gonna Give You Up', by Rick Astley
  ```

- 类型转换实际上不会改变实例及修改其值。实例不会改变；**它只是将它当做要转换的类型来访问。**

### 4 Any 和 AnyObject 的类型转换

+ AnyObject  可以表示任何类类型的实例。

+ Any  可以表示任何类型，包括函数类型。

  ```swift
  //使用示例
  var things = [Any]()
   
  things.append(0)
  things.append(0.0)
  things.append(42)
  things.append(3.14159)
  things.append("hello")
  things.append((3.0, 5.0))
  things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
  things.append({ (name: String) -> String in "Hello, \(name)" })
  
  for thing in things {
      switch thing {
      case 0 as Int:
          print("zero as an Int")
      case 0 as Double:
          print("zero as a Double")
      case let someInt as Int:
          print("an integer value of \(someInt)")
      case let someDouble as Double where someDouble > 0:
          print("a positive double value of \(someDouble)")
      case is Double:
          print("some other double value that I don't want to print")
      case let someString as String:
          print("a string value of \"\(someString)\"")
      case let (x, y) as (Double, Double):
          print("an (x, y) point at \(x), \(y)")
      case let movie as Movie:
          print("a movie called \(movie.name), dir. \(movie.director)")
      case let stringConverter as (String) -> String:
          print(stringConverter("Michael"))
      default:
          print("something else")
      }
  }
   
  // zero as an Int
  // zero as a Double
  // an integer value of 42
  // a positive double value of 3.14159
  // a string value of "hello"
  // an (x, y) point at 3.0, 5.0
  // a movie called Ghostbusters, dir. Ivan Reitman
  // Hello, Michael
  
  ```

+ Any类型表示了任意类型的值，包括可选类型。如果你给显式声明的Any类型使用可选项，Swift 就会发出警告。

+ 如果你真心需要在Any值中使用可选项，如下所示，你可以使用as运算符来显式地转换可选项为Any。

  ```swift
  let optionalNumber: Int? = 3
  things.append(optionalNumber)        // Warning
  things.append(optionalNumber as Any) // No warning
  ```


## 二十 内嵌类型

+ 一种类型中嵌套另一种类型，在其支持类型的大括号内定义即可

  ```swift
  struct BlackjackCard {
      // nested Suit enumeration
      enum Suit: Character {
          case Spades = "♠", Hearts = "♡", Diamonds = "♢", Clubs = "♣"
      }
  }
  ```


## 二十一  扩展

- **扩展**为现有的`类`、`结构体`、`枚举类型`、或`协议`**添加了新功能**
- 为无访问权限的源代码扩展类型（即所谓的**逆向建模**）
- 添加计算实例属性和计算类型属性；
- 定义实例方法和类型方法；
- 提供新初始化器；
- 定义下标；
- 定义和使用新内嵌类型；
- 使现有的类型遵循某协议
- 扩展可以向一个类型添加新的方法，但是`不能重写`已有的方法。

### 1 扩展的语法

+ 使用 extension 关键字来声明扩展

  ```swift
  extension SomeType {
      // new functionality to add to SomeType goes here
  }
  ```

+ 扩展可以使已有的类型遵循一个或多个协议

  ```swift
  extension SomeType: SomeProtocol, AnotherProtocol {
      // implementation of protocol requirements goes here
  }
  ```

+ 如果你向已存在的类型添加新功能，新功能会在该类型的`所有实例`中可用，即使实例在该扩展定义之前就已经创建。

### 2 计算属性

+ 扩展可以向已有的类型添加计算实例属性和计算类型属性

  ```swift
  extension Double {
      var km: Double { return self * 1_000.0 }
      var m: Double { return self }
      var cm: Double { return self / 100.0 }
      var mm: Double { return self / 1_000.0 }
      var ft: Double { return self / 3.28084 }
  }
  let oneInch = 25.4.mm
  print("One inch is \(oneInch) meters")
  // Prints "One inch is 0.0254 meters"
  let threeFeet = 3.ft
  print("Three feet is \(threeFeet) meters")
  // Prints "Three feet is 0.914399970739201 meters"
  ```

+ 扩展可以添加新的计算属性，但是**不能添加存储属性**，也**不能向已有的属性添加属性观察者。**

### 3 初始化器

- 扩展可向已有的类型添加新的初始化器
- 扩展能为类添加新的便捷初始化器
- 不能为类添加指定初始化器或反初始化器

> 如果你使用扩展为一个值类型添加初始化器，且该值类型为其所有储存的属性提供默认值，而又不定义任何自定义初始化器时，你可以在你扩展的初始化器中调用该类型默认的初始化器和成员初始化器。

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
}

//值类型:默认的初始化器以及成员初始化器
let defaultRect = Rect()
let memberwiseRect = Rect(origin: Point(x: 2.0, y: 2.0),
                          size: Size(width: 5.0, height: 5.0))
//值类型:扩展初始化器，其内部可以直接调用默认的初始化器以及成员初始化器                          
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}                          
```



### 4 方法

- 扩展可以为已有的类型添加新的实例方法和类型方法

  ```swift
  extension Int {
      func repetitions(task: () -> Void) {
          for _ in 0..<self {
              task()
          }
      }
  }
  
  3.repetitions {
      print("Hello!")
  }
  // Hello!
  // Hello!
  // Hello!
  ```


#### 4.1 异变实例方法

- 增加了扩展的实例方法仍可以修改（或*异变*）实例本身

- `结构体`和`枚举类型`方法在`修改 self` 或本身的`属性`时必须标记实例方法为` mutating `，和原本实现的异变方法一样。

  ```swift
  extension Int {
      mutating func square() {
          self = self * self
      }
  }
  var someInt = 3
  someInt.square()
  ```

 ### 5 下标

- 扩展能为已有的类型添加新的下标

  ```swift
  extension Int {
      subscript(digitIndex: Int) -> Int {
          var decimalBase = 1
          for _ in 0..<digitIndex {
              decimalBase *= 10
          }
          return (self / decimalBase) % 10
      }
  }
  746381295[0]
  // returns 5
  746381295[1]
  // returns 9
  746381295[2]
  // returns 2
  746381295[8]
  // returns 7
  ```


### 6 内嵌类型

- 扩展可以为已有的类、结构体和枚举类型添加新的内嵌类型

  ```swift
  extension Int {
      enum Kind {
          case negative, zero, positive
      }
      var kind: Kind {
          switch self {
          case 0:
              return .zero
          case let x where x > 0:
              return .positive
          default:
              return .negative
          }
      }
  }
  
  func printIntegerKinds(_ numbers: [Int]) {
      for number in numbers {
          switch number.kind {
          case .negative:
              print("- ", terminator: "")
          case .zero:
              print("0 ", terminator: "")
          case .positive:
              print("+ ", terminator: "")
          }
      }
      print("")
  }
  printIntegerKinds([3, 19, -27, 0, -6, 0, 7])
  // Prints "+ + - 0 - 0 + "
  ```

  > 已知 number.kind 是 Int.Kind 类型。因此， switch 语句中的所有 Int.Kind 情况值都可以简写，例如用 .Negative 而不是Int.Kind.Negative 。



## 二十二 协议

+ 协议可被类、结构体、或枚举类型*采纳*以提供所需功能的具体实现

### 1 协议的语法

+ 语法格式

  ```swift
  protocol SomeProtocol {
      // protocol definition goes here
  }
  ```

+ 如何遵循一个协议

  ```swift
  struct SomeStructure: FirstProtocol, AnotherProtocol {
      // structure definition goes here
  }
  
  class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
      // class definition goes here
  }
  ```

### 2 属性要求

+ 协议要求类型提供特定名字和类型的实例属性或类型属性

+ 属性必须明确是可读的或可读写的。

+ 只定义属性的名称和类型，不管其是计算类型还是存储类型

+ 使用var声明变量属性，使用{get set}明确属性可读写，使用{get}明确属性只读

  ````swift
  protocol SomeProtocol {
      var mustBeSettable: Int { get set } //可读写属性
      var doesNotNeedToBeSettable: Int { get } //只读属性
  }
  ````

+ 使用static关键字声明类型属性

  ```swift
   protocol AnotherProtocol {
      static var someTypeProperty: Int { get set }
  }
  ```

### 3 方法要求

+ 协议可以要求采纳协议的类型实现指定的实例方法和类方法

+ 在协议的定义中，方法参数不能定义默认值。

  ```swift
  protocol RandomNumberGenerator {
      func random() -> Double
  }
  class LinearCongruentialGenerator: RandomNumberGenerator {
      var lastRandom = 42.0
      let m = 139968.0
      let a = 3877.0
      let c = 29573.0
      func random() -> Double {
          lastRandom = ((lastRandom * a + c).truncatingRemainder(dividingBy:m))
          return lastRandom / m
      }
  }
  let generator = LinearCongruentialGenerator()
  print("Here's a random number: \(generator.random())")
  // Prints "Here's a random number: 0.37464991998171"
  print("And another one: \(generator.random())")
  // Prints "And another one: 0.729023776863283"
  ```


### 4 异变方法要求 

+ 对于值类型的实例方法（即结构体或枚举）,在方法的 func 关键字之前使用 mutating 关键字，来表示在该方法可以改变其所属的实例，以及该实例的所有属性

+ 注意: 在协议中定义异变方法，需要用mutating关键字修饰。但是如果类也采纳该协议，在类中实现该方法时，不需要mutating关键字修饰。

  ```swift
  protocol Togglable {
      mutating func toggle()
  }
  
  enum OnOffSwitch: Togglable {
      case off, on
      mutating func toggle() {
          switch self {
          case .off:
              self = .on
          case .on:
              self = .off
          }
      }
  }
  var lightSwitch = OnOffSwitch.off
  lightSwitch.toggle()
  // lightSwitch is now equal to .on
  ```

### 5 初始化器要求

- 协议可以要求遵循协议的类型实现指定的初始化器

  ```swift
  protocol SomeProtocol {
      init(someParameter: Int)
  }
  ```

#### 5.1 协议初始化器要求的类实现

- 在类中必须使用 required 关键字修饰初始化器的实现

- required 修饰的使用保证了所有子类提供一个明确的继承实现

  ```swift
  class SomeClass: SomeProtocol {
      required init(someParameter: Int) {
          // initializer implementation goes here
      }
  }
  ```

- 由于 final 的类不会有子类，如果协议初始化器实现的类使用了 final 标记，你就不需要使用 required 来修饰了

- 如果一个子类重写了父类指定的初始化器，并且遵循协议实现了初始化器要求,则需要用required和override修饰

  ```swift
  protocol SomeProtocol {
      init()
  }
   
  class SomeSuperClass {
      init() {
          // initializer implementation goes here
      }
  }
   
  class SomeSubClass: SomeSuperClass, SomeProtocol {
      // "required" from SomeProtocol conformance; "override" from SomeSuperClass
      required override init() {
          // initializer implementation goes here
      }
  }
  ```

### 5.2 可失败初始化器要求

- 协议可以为遵循该协议的类型定义可失败的初始化器

### 6 将协议作为类型

+ 作为一个类型，可以这么使用

  + 在函数、方法或者初始化器里作为形式参数类型或者返回类型；

  + 作为常量、变量或者属性的类型；

  + 作为数组、字典或者其他存储器的元素的类型。

    ```swift
    class Dice {
        let sides: Int
        //generator即为任何遵守RandomNumberGenerator的协议的类型的实例
        let generator: RandomNumberGenerator
        init(sides: Int, generator: RandomNumberGenerator) {
            self.sides = sides
            self.generator = generator
        }
        func roll() -> Int {
            return Int(generator.random() * Double(sides)) + 1
        }
    }
    
    var d6 = Dice(sides: 6, generator: LinearCongruentialGenerator())
    for _ in 1...5 {
        print("Random dice roll is \(d6.roll())")
    }
    // Random dice roll is 3
    // Random dice roll is 5
    // Random dice roll is 4
    // Random dice roll is 5
    // Random dice roll is 4
    ```


### 7 委托

- **委托**是一个允许类或者结构体放手（或者说委托）它们自身的某些责任给另外类型实例的设计模式
- 由遵循了协议的类型（所谓的委托）来保证提供被委托的功能

### 8 在扩展里添加协议遵循

+ 为已经存在的类型扩展新的属性、方法和下标，即使你无法访问现有类型的源代码

  ```swift
  protocol TextRepresentable {
      var textualDescription: String { get }
  }
  
  extension Dice: TextRepresentable {
      var textualDescription: String {
          return "A \(sides)-sided dice"
      }
  }
  
  let d12 = Dice(sides: 12, generator: LinearCongruentialGenerator())
  print(d12.textualDescription)
  // Prints "A 12-sided dice"
  ```


#### 8.1 有条件地遵循协议 ???

+ 泛型类型可能只在某些情况下满足一个协议的要求 

  ```
  面的扩展让 Array 类型在存储遵循 TextRepresentable 协议的元素时遵循 TextRepresentable 协议。
  
  extension Array: TextRepresentable where Element: TextRepresentable {
      var textualDescription: String {
          let itemsAsText = self.map { $0.textualDescription }
          return "[" + itemsAsText.joined(separator: ", ") + "]"
      }
  }
  let myDice = [d6, d12]
  print(myDice.textualDescription)
  // Prints "[A 6-sided dice, A 12-sided dice]"
  ```


#### 8.2 使用扩展声明采纳协议

+ 如果一个类型已经遵循了协议的所有需求，但是还没有声明它采纳了这个协议，你可以让通过一个空的扩展来让它采纳这个协议

+ 注意类型不会因为满足需要就自动采纳协议。它们必须显式地声明采纳了哪个协议。

  ```swift
  struct Hamster {
      var name: String
      var textualDescription: String {
          return "A hamster named \(name)"
      }
  }
  extension Hamster: TextRepresentable {}
  
  //此时Hamster可以用在任何需要遵循TextRepresentable协议的类型的地方
  let simonTheHamster = Hamster(name: "Simon")
  let somethingTextRepresentable: TextRepresentable = simonTheHamster
  print(somethingTextRepresentable.textualDescription)
  // Prints "A hamster named Simon"
  
  ```

### 9 协议类型的集合

- 协议可以用作储存在集合比如数组或者字典中的类型

  ```swift
  //数组中的所有元素均为遵循了TextRepresentable协议的类型
  let things: [TextRepresentable] = [game, d12, simonTheHamster]
  for thing in things {
      print(thing.textualDescription)
  }
  // A game of Snakes and Ladders with 25 squares
  // A 12-sided dice
  // A hamster named Simon
  ```

### 10 协议继承

- 协议可以*继承*一个或者多个其他协议并且可以在它继承的基础之上添加更多要求

  ```swift
  protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
      // protocol definition goes here
  }
  ```

### 11 类专用的协议

- 通过添加 AnyObject 关键字到协议的继承列表，你就可以限制协议只能被类类型采纳。

  ```swift
  protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
      // class-only protocol definition goes here
  }
  ```


### 12 协议组合

- 你可以使用*协议组合*来复合多个协议到一个要求里

- 协议组合使用 SomeProtocol & AnotherProtocol 的形式

  ```swift
  protocol Named {
      var name: String { get }
  }
  protocol Aged {
      var age: Int { get }
  }
  struct Person: Named, Aged {
      var name: String
      var age: Int
  }
  //Named, Aged都是协议
  func wishHappyBirthday(to celebrator: Named & Aged) {
      print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
  }
  let birthdayPerson = Person(name: "Malcolm", age: 21)
  wishHappyBirthday(to: birthdayPerson)
  // Prints "Happy birthday, Malcolm, you're 21!"
  ```

- 用和符号连接（ & ）任意数量的协议

- 除了协议列表，协议组合也能包含类类型，这允许你标明一个需要的父类

  ```swift
  class Location {
      var latitude: Double
      var longitude: Double
      init(latitude: Double, longitude: Double) {
          self.latitude = latitude
          self.longitude = longitude
      }
  }
  class City: Location, Named {
      var name: String
      init(name: String, latitude: Double, longitude: Double) {
          self.name = name
          super.init(latitude: latitude, longitude: longitude)
      }
  }
  //Location是类, Named是协议
  func beginConcert(in location: Location & Named) {
      print("Hello, \(location.name)!")
  }
   
  let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
  beginConcert(in: seattle)
  // Prints "Hello, Seattle!"
  ```


### 13 协议遵循的检查

- 如同类型转换一样，可以使用的 is 和 as 运算符来检查协议遵循，还能转换为特定的协议

  1. 如果实例遵循协议is运算符返回 true 否则返回 false ；

  2. as? 版本的向下转换运算符返回协议的可选项，如果实例不遵循这个协议的话值就是 nil 

  3. as! 版本的向下转换运算符强制转换协议类型并且在失败是触发运行时错误。

     ```swift
     protocol HasArea {
         var area: Double { get }
     }
     class Circle: HasArea {
         let pi = 3.1415927
         var radius: Double
         var area: Double { return pi * radius * radius }
         init(radius: Double) { self.radius = radius }
     }
     class Country: HasArea {
         var area: Double
         init(area: Double) { self.area = area }
     }
     class Animal {
         var legs: Int
         init(legs: Int) { self.legs = legs }
     }
     
     let objects: [AnyObject] = [
         Circle(radius: 2.0),
         Country(area: 243_610),
         Animal(legs: 4)
     ]
     for object in objects {
         if let objectWithArea = object as? HasArea {
             print("Area is \(objectWithArea.area)")
         } else {
             print("Something that doesn't have an area")
         }
     }
     // Area is 12.5663708
     // Area is 243610.0
     // Something that doesn't have an area
     ```


### 14 可选协议要求

+ 可选要求使用 optional 修饰符作为前缀放在协议的定义中

+ 可选要求允许你的代码与 Objective-C 操作 , 协议和可选要求必须使用 @objc 标志标记

+ @objc 协议只能被继承自 Objective-C 类或其他 @objc 类采纳，它们不能被结构体或者枚举采纳。

+ 在可选要求中使用方法或属性是，它的类型自动变成可选项

  ```swift
  例如(Int) -> String 类型的方法会变成 ((Int) -> String)? 。
  注意是这个函数类型变成可选项，不是方法的返回值。
  ```

### 15 协议扩展

- 协议可以通过扩展来提供方法和属性的实现以遵循类型

- 通过给协议创建扩展，所有的遵循类型自动获得这个方法的实现而不需要任何额外的修改。

  ```swift
  extension RandomNumberGenerator {
      func randomBool() -> Bool {
          return random() > 0.5
      }
  }
  
  let generator = LinearCongruentialGenerator()
  print("Here's a random number: \(generator.random())")
  // Prints "Here's a random number: 0.37464991998171"
  print("And here's a random Boolean: \(generator.randomBool())")
  // Prints "And here's a random Boolean: true"
  ```

#### 15.1 提供默认实现

- 可以使用协议扩展来给协议的任意方法或者计算属性要求提供默认实现

- 如果遵循类型给这个协议的要求提供了它自己的实现，那么它就会替代扩展中提供的默认实现。

  ```swift
  extension PrettyTextRepresentable  {
      var prettyTextualDescription: String {
          return textualDescription
      }
  }
  ```

#### 15.2 给协议扩展添加限制？？？

- 定义协议扩展的时候，可以明确**遵循类型必须**在扩展的方法和属性可用之前**满足的限制**

- 可以在扩展协议名字后边使用 `where 分句`来写这些限制

  ```swift
  extension Collection where Iterator.Element: TextRepresentable {
      var textualDescription: String {
          let itemsAsText = self.map { $0.textualDescription }
          return "[" + itemsAsText.joined(separator: ", ") + "]"
      }
  }
  
  ```




## 二十三 泛型

### 1 泛型解决的问题

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}

//像上面的这几个函数，我们可以用泛型定义一个函数统一进行处理
```



### 2 泛型函数

- *泛型函数*可以用于任何类型。 泛型函数对上面问题的处理

  ```swift
  func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
      let temporaryA = a
      a = b
      b = temporaryA
  }
  ```

  - `<T>`告诉swift, `T`是一个占位符类型名,这样swift就不会去查找名为T的类型了。
  - 占位符类型名没有声明 T 必须是什么样的, 但是声明了a,b是同一个类型

### 3 类型形式参数

- 类型形式参数指定并且命名一个占位符类型，紧挨着写在函数名后面的一对尖括号里（比如 `<T> `

### 4 命名类型形式参数

- 大多数情况下，类型形式参数的名字要有描述性

  ```swift
  比如 Dictionary<Key, Value> 中的 Key  和 Value ，借此告知读者类型形式参数和泛型类型、泛型用到的函数之间的关系
  ```

- 类型形式参数如果没有实际意义时，一般按惯例用单个字母命名

  ```swift
  例如T 、 U 、 V 等
  ```

- 类型形式参数永远用大写开头的驼峰命名法（比如 T 和 MyTypeParameter ）命名，以指明它们是一个类型的占位符，不是一个值

### 5 泛型类型

- 定义一个 Stack 的泛型集合类型

  ```swift
  struct Stack<Element> {
      var items = [Element]()
      mutating func push(_ item: Element) {
          items.append(item)
      }
      mutating func pop() -> Element {
          return items.removeLast()
      }
  }
  
  var stackOfStrings = Stack<String>()
  stackOfStrings.push("uno")
  stackOfStrings.push("dos")
  stackOfStrings.push("tres")
  stackOfStrings.push("cuatro")
  // the stack now contains 4 strings
  
  let fromTheTop = stackOfStrings.pop()
  // fromTheTop is equal to "cuatro", and the stack now contains 3 strings
  ```



### 6 扩展一个泛型类型

- 原始类型定义的类型形式参数列表在扩展体里仍然有效

  ```swift
  extension Stack {
      var topItem: Element? {
          return items.isEmpty ? nil : items[items.count - 1]
      }
  }
  if let topItem = stackOfStrings.topItem {
      print("The top item on the stack is \(topItem).")
  }
  ```


### 7 类型约束

- 类型约束指出一个类型形式参数必须继承自特定类，或者遵循一个特定的协议、组合协议。

  ```swift
  例如Dictionary的Key必须遵循hashable协议
  
  public struct Dictionary<Key, Value> : Collection, ExpressibleByDictionaryLiteral where Key : Hashable
  ```

#### 7.1 类型约束语法

- 在一个类型形式参数名称后面放置一个类或者协议作为形式参数列表的一部分，并用冒号隔开。 语法为

  ```swift
  func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
      // function body goes here
  }
  ```

  - 第一个类型形式参数T是SomeClass的子类
  - 第二个类型形式参数U遵循SomeProtocol协议

#### 7.2 类型约束的应用

- 类型约束的一个示例

  ```swift
  func findIndex<T>(of valueToFind: T, in array:[T]) -> Int? {
      for (index, value) in array.enumerated() {
          if value == valueToFind {//编译报错,因为不是所有的类型都能用==
              return index
          }
      }
      return nil
  }
  
  //通过使泛型T遵循Equatable协议，从而让它们能够使用==
  func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
      for (index, value) in array.enumerated() {
          if value == valueToFind {
              return index
          }
      }
      return nil
  }
  ```

### 8 关联类型

- 定义一个协议时,可以通过associatedtype 关键字指定关联类型

#### 8.1 关联类型的应用 

-  Container 的示例协议，声明了一个叫做 ItemType 的关联类型

  ```swift
  protocol Container {
      associatedtype ItemType
      mutating func append(_ item: ItemType)
      var count: Int { get }
      subscript(i: Int) -> ItemType { get }
  }
  
  struct IntStack: Container {
      // original IntStack implementation
      var items = [Int]()
      mutating func push(_ item: Int) {
          items.append(item)
      }
      mutating func pop() -> Int {
          return items.removeLast()
      }
      // conformance to the Container protocol
      //*******不必显式指定ItemType的类型，swift能够通过类型推断出来****
      typealias ItemType = Int
      mutating func append(_ item: Int) {
          self.push(item)
      }
      var count: Int {
          return items.count
      }
      subscript(i: Int) -> Int {
          return items[i]
      }
  }
  ```

#### 8.2 给关联类型添加约束

- 要求协议中关联的类型满足约束

  ```swift
  protocol Container {
      associatedtype Item: Equatable
      mutating func append(_ item: Item)
      var count: Int { get }
      subscript(i: Int) -> Item { get }
  }
  ```

### 9 泛型Where分句

- 通过泛型Where分句，为关联类型定义要求

  ```swift
  func allItemsMatch<C1: Container, C2: Container>
      (_ someContainer: C1, _ anotherContainer: C2) -> Bool
      where C1.ItemType == C2.ItemType, C1.ItemType: Equatable {
          
          // Check that both containers contain the same number of items.
          if someContainer.count != anotherContainer.count {
              return false
          }
          
          // Check each pair of items to see if they are equivalent.
          for i in 0..<someContainer.count {
              if someContainer[i] != anotherContainer[i] {
                  return false
              }
          }
          
          // All items match, so return true.
          return true
  }
  ```

  - C1 必须遵循 Container 协议（写作 C1: Container ）；
  - C2 也必须遵循 Container 协议（写作 C2: Container ）；
  - C1 的 ItemType 必须和 C2 的 ItemType 相同（写作 C1.ItemType == C2.ItemType ）；
  - C1 的 ItemType 必须遵循 Equatable 协议（写作 C1.ItemType: Equatable ）。

### 10 带有泛型 Where 分句的扩展

- 可以使用泛型的 where 分句来作为扩展的一部分

  ```swift
  extension Stack where Element: Equatable {
      func isTop(_ item: Element) -> Bool {
          //扩展遵循Equatable协议，所以可以使用 == 
          guard let topItem = items.last else {
              return false
          }
          return topItem == item
      }
  }
  
  if stackOfStrings.isTop("tres") {
      print("Top element is tres.")
  } else {
      print("Top element is something else.")
  }
  // Prints "Top element is tres."
  ```

### 11 关联类型的泛型 Where 分句

- 可以在关联类型中包含一个泛型 where 分句

  ```swift
  protocol Container {
      associatedtype Item
      mutating func append(_ item: Item)
      var count: Int { get }
      subscript(i: Int) -> Item { get }
      
      associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
      func makeIterator() -> Iterator
  }
  ```

### 12 泛型下标

- 这个 Container 协议的扩展添加了一个接收一系列索引并返回包含给定索引元素的数组

  ```swift
  extension Container {
      subscript<Indices: Sequence>(indices: Indices) -> [Item]
          where Indices.Iterator.Element == Int {
              var result = [Item]()
              for index in indices {
                  result.append(self[index])
              }
              return result
      }
  }
  ```

  - 在尖括号中的泛型形式参数 Indices 必须是遵循标准库中 Sequence 协议的某类型；
  - 下标接收单个形式参数， indices ，它是一个 Indices 类型的实例；
  - 泛型 where 分句要求序列的遍历器必须遍历 Int 类型的元素。这就保证了序列中的索引都是作为容器索引的相同类型。
  - 

## 二十四 内存安全性

### 1 了解内存访问冲突 

- 内存访问冲突会在你的代码不同地方同一时间尝试访问同一块内存时发生

#### 1.1 典型的内存访问

- 一般访问冲突，发生在至少两个访问并满足下列条件时

  1. 至少一个是写入访问；
  2. 它们访问的是同一块内存；
  3. 它们的访问时间重叠。

- 即时访问: 一个访问在启动后其他代码不能执行直到它结束

  ```swift
  //下面普通的读写都是即时访问
  func oneMore(than number: Int) -> Int {
      return number + 1
  }
   
  var myNumber = 1
  myNumber = oneMore(than: myNumber)
  print(myNumber)
  // Prints "2"
  ```

- 长时访问: 一个访问开始后，在它结束之前其他代码依旧可以运行。这就造成了重叠。

- 重叠访问主要是出现在使用了输入输出形式参数的函数以及方法或者结构体中的异变方法

### 2 输入输出形式参数的访问冲突

- 对**输入输出形式参数的写入访问**会在所有非输入输出形式参数计算之后开始，并**持续到整个函数调用结束**

- 如果有多个输入输出形式参数，那么写入访问会以形式参数出现的顺序开始

- 在输入输出形式参数的函数中，任何对原变量的访问都会造成冲突

  ````swift
  var stepSize = 1
  func increment(_ number: inout Int) {
      number += stepSize
  }
   
  increment(&stepSize)
  // Error: conflicting accesses to stepSize
  
  //错误原因是: stepSize和number是引用的同一个内存地址
  读取和写入访问引用同一内存并且重叠，产生了冲突。
  
  //解决方案:---------
  // Make an explicit copy.
  var copyOfStepSize = stepSize
  //此时copyOfStepSize和number指向的不是同一块内存地址,所以不会冲突
  increment(&copyOfStepSize)
   
  // Update the original.
  stepSize = copyOfStepSize
  // stepSize is now 2
  // stepSize is now 2
  
  ````

- 输入输出形式参数的长时写入访问的另一个后果是传入一个单独的变量给同一个函数的多个输入输出形式参数产生冲突

  ```swift
  func balance(_ x: inout Int, _ y: inout Int) {
      let sum = x + y
      x = sum / 2
      y = sum - x
  }
  var playerOneScore = 42
  var playerTwoScore = 30
  balance(&playerOneScore, &playerTwoScore)  // OK
  //它尝试执行两个写入访问到同一个内存地址且是在同一时间执行，所以错误
  balance(&playerOneScore, &playerOneScore)
  // Error: Conflicting accesses to playerOneScore
  ```


### 3 在方法中对 self 的访问冲突

- 结构体中的异变方法可以在方法调用时对 self 进行写入访问

  ```swift
  struct Player {
      var name: String
      var health: Int
      var energy: Int
      
      static let maxHealth = 10
      mutating func restoreHealth() {
          health = Player.maxHealth
      }
  }
  
  extension Player {
      mutating func shareHealth(with teammate: inout Player) {
          balance(&teammate.health, &health)
      }
  }
   
  var oscar = Player(name: "Oscar", health: 10, energy: 10)
  var maria = Player(name: "Maria", health: 5, energy: 10)
  //其调用内部同时访问oscar,maria进行写入，操作的是不同的内存地址,所以没问题
  oscar.shareHealth(with: &maria)  // OK
  
  //其调用内部同时访问oscar进行写入，操作的是同一的内存地址，所以有问题
  oscar.shareHealth(with: &oscar)
  // Error: conflicting accesses to oscar
  ```


### 4 属性的访问冲突

+ 对于值类型, 改变任何一个值都会改变整个类型，意味着读或者写访问到这些属性就需要对整个值进行读写访问

  ```swift
  //对元组元素的写入访问需要整个元组的写入访问。也就是说 playerInformation 在调用过程中有两个写入访问重叠，导致冲突
  
  var playerInformation = (health: 10, energy: 20)
  balance(&playerInformation.health, &playerInformation.energy)
  // Error: conflicting access to properties of playerInformation
  ```

+ 对全局变量结构体属性的重叠写入访问，导致同样的错误。

  ```swift
  var holly = Player(name: "Holly", health: 10, energy: 10)
  balance(&holly.health, &holly.energy)  // Error
  ```

+ 结构体变成局部变量而不是全局变量，那么编译器就可以保证重叠访问结构体的存储属性是安全的

  ````swift
  func someFunction() {
      var oscar = Player(name: "Oscar", health: 10, energy: 10)
      balance(&oscar.health, &oscar.energy)  // OK
  }
  ````

+ 满足下面的条件就说明重叠访问结构体的属性是安全的

  - 你只访问实例的存储属性，不是计算属性或者类属性；
  - 结构体是局部变量而非全局变量；
  - 结构体要么没有被闭包捕获要么只被非逃逸闭包捕获。

## 二十五 访问控制

- **访问控制**限制其他源文件和模块对你的代码的访问



### 1 模块和源文件

+ 模块是单一的代码分配单元，一般指的框架和库，使用 import 关键字导入
+ 源文件是单个 Swift 源代码文件

> 简洁起见，代码中可以设置访问级别的部分(属性，类型，函数等)在下面的章节称为“实体”。

### 2 访问级别

+ Swift 为代码的实体提供个五个不同的访问级别

  1. `Open 访问` 和 `public 访问` 允许实体在模块内和在导入该模块的文件内被访问

  2. `Internal 访问 `允许实体在模块内被访问，但是不允许实体在模块外被访问

  3. `File-private 访问` 将实体的使用限制于当前定义源文件中

  4. `private 访问` 将实体的使用限制于封闭声明中

     ```
     open 访问是最高的（限制最少）访问级别，private 是最低的（限制最多）访问级别
     ```

+ open 访问仅适用于类和类成员，它与 public 访问区别如下

  - public 访问，或任何更严格的访问级别的类，只能在其定义模块中被继承。
  - public 访问，或任何更严格访问级别的类成员，只能被其定义模块的子类重写。
  - open 类可以在其定义模块中被继承，也可在任何导入定义模块的其他模块中被继承。
  - open 类成员可以被其定义模块的子类重写，也可以被导入其定义模块的任何模块重写。

#### 2.1 访问级别的指导准则

- 一个 public 的变量其类型的访问级别不能是 internal, file-private 或是 private，因为在使用 public 变量的地方可能没有这些类型的访问权限。
- 一个函数不能比它的参数类型和返回类型访问级别高，因为函数可以使用的环境而其参数和返回类型却不能使用

- 代码中的所有实体（以及本章后续提及的少数例外）默认为 internal 级别

#### 2.2 框架的访问级别

- 开发框架时，通过open 或 public，对外提供被访问能力。

#### 2.3 单元测试目标的访问级别 ???？

- 使用 @testable  属性标注了导入的生产模块并且用使能测试的方式编译了这个模块，单元测试目标就能访问任何 internal 的实体。



###  3 访问控制语法

- 通过在实体的引入之前添加 open ， public ， internal ， fileprivate ，或 private  修饰符来定义访问级别

  ```swift
  public class SomePublicClass {}
  internal class SomeInternalClass {}
  fileprivate class SomeFilePrivateClass {}
  private class SomePrivateClass {}
   
  public var somePublicVariable = 0
  internal let someInternalConstant = 0
  fileprivate func someFilePrivateFunction() {}
  private func somePrivateFunction() {}
  ```

### 4 自定类型

- 类型的访问控制级别也会影响它的成员的默认访问级别（它的属性，方法，初始化方法，下标）

  + 如果你将类型定义为 private 或 file private 级别，那么它的成员的默认访问级别也会是 private 或file private。

  + 如果你将类型定义为 internal 或 public级别（或直接使用默认级别而不显式指出），那么它的成员的默认访问级别会是 internal 。

  + 如果你想让pbulic类型的一个成员是 public 的，你必须用public明确指明该成员

    ```swift
    public class SomePublicClass {                          // explicitly public class
          public var somePublicProperty = 0               // explicitly public class member
          var someInternalProperty = 0                      // implicitly internal class member
          fileprivate func someFilePrivateMethod() {}   // explicitly file-private class member
          private func somePrivateMethod() {}           ／/ explicitly private class member
    }
     
    class SomeInternalClass {                                 // implicitly internal class
          var someInternalProperty = 0                      // implicitly internal class member
          fileprivate func someFilePrivateMethod() {}   // explicitly file-private class member
          private func somePrivateMethod() {}            // explicitly private class member
    }
     
    fileprivate class SomeFilePrivateClass {            // explicitly file-private class
          func someFilePrivateMethod() {}                 // implicitly file-private class member
          private func somePrivateMethod() {}           // explicitly private class member
    }
     
    private class SomePrivateClass {                     // explicitly private class
          func somePrivateMethod() {}                     // implicitly private class member
    }
    ```

#### 4.1 元组类型

- 元组类型的访问级别是其元素中最严格的访问级别决定

- 元组类型的访问级别会在使用的时候被自动推断出来，不需要显式指明。

  ```swift
  如果你将两个不同类型的元素组成一个元组，一个元素的访问级别是 internal，另一个是 private，那么这个元组类型是 private 级别的。
  ```

#### 4.2 函数类型

- 函数类型的访问级别由函数成员类型和返回类型中的最严格访问级别决定

- 函数的计算(实际)访问级别与上下文环境默认级别不匹配，你必须在函数定义时**显式指出**

  ```swift
  //错误😭😭😭，因为someFunction实际访问级别是private,而默认访问级别是internal
  func someFunction() -> (SomeInternalClass, SomePrivateClass) {
      // function implementation goes here
  }
  
  //正确😆😆😆
  private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
      // function implementation goes here
  }
  ```

#### 4.3 枚举类型 

- 枚举中的独立成员自动使用该枚举类型的访问级别

- 你不能给独立的成员指明一个不同的访问级别

  ```swift
  public enum CompassPoint {
        case north
        case south
        case east
        case west
  }
  ```

#### 4.3.1 原始值和关联值

- 枚举定义中的原始值和关联值使用的类型必须有一个不低于枚举的访问级别

  ```swift
  你不能使用一个 private 类型作为一个 internal 级别的枚举类型中的原始值类型。
  ```

#### 4.4 嵌套类型

- private 级别的类型中定义的嵌套类型自动为 private 级别
- fileprivate 级别的类型中定义的嵌套类型自动为 fileprivate 级别
- public 或 internal 级别的类型中定义的嵌套类型自动为 internal 级别
- 如果你想让嵌套类型是 public 级别的，你必须将其显式指明为 public。



#### 4.5 子类???

- 你可以继承任何类只要是在当前可以访问的上下文环境中。

- 但子类不能高于父类的访问级别，可以低于

  ```swift
  例如: 你不能写一个 internal 父类的 public 子类
  ```

- 可以重写任何类成员（方法，属性，初始化器或下标），只要是在确定的访问域中是可见的

  ```swift
  public class A {
           fileprivate func someMethod() {}
  }
  internal class B: A {
           override internal func someMethod() {}
  }
  ```

- xxxxx

### 5 常量，变量，属性和下标

- 常量、变量、属性不能拥有比它们类型更高的访问级别。例如

### 6 Getters 和 Setters

- 常量、变量、属性和下标的 getter 和 setter 自动接收它们所属常量、变量、属性和下标的访问级别。

- 你可以给 setter 函数一个比相对应 getter 函数更低的访问级别以限制变量、属性、下标的读写权限

- 你可以通过在var 和 subscript 的置入器之前书写 fileprivate(set) , private(set) , 或 internal(set) 来声明更低的访问级别。

  ```swift
  struct TrackedString {
           private(set) var numberOfEdits = 0
           var value: String = "" {
           didSet {
                 numberOfEdits += 1
           }
           }
  }
  
  //numberOfEdits在内部是可读写的，在外部是只读的
  
  var stringToEdit = TrackedString()
  stringToEdit.value = "This string will be tracked."
  stringToEdit.value += " This edit will increment numberOfEdits."
  stringToEdit.value += " So will this one."
  //在外部只能获取numberOfEdits的值，不能直接设置其值
  print("The number of edits is \(stringToEdit.numberOfEdits)")
  // Prints "The number of edits is 3"
  
  
  //numberOfEdits默认是internal,也可以通过public private(set),设置其getter方法是public级别的
  public struct TrackedString {
        public private(set) var numberOfEdits = 0
        public var value: String = "" {
              didSet {
                    numberOfEdits += 1
              }
        }
        public init() {}
  }
  
  ```


### 7 初始化器

- 我们可以给自定义初始化方法设置一个低于或等于它的所属的类的访问级别
- 但是必要初始化器必须和它所属类的访问级别一致
- 就像函数和方法的参数一样，初始化器的参数类型不能比初始化方法的访问级别还低。

#### 7.1 默认初始化器

- 默认初始化方法与所属类的访问级别一致
- 但如果一个类定义为 public ，那么默认初始化方法为 internal 级别
- 使用public修饰默认初始化器，以便使它能够在其他模块中被访问

#### 7.2 结构体的默认成员初始化器

+ 与7.1默认初始化一致

### 8 协议

- 协议定义中的每一个要求的访问级别都自动设为与该协议相同
- 你不能将一个协议要求的访问级别设为与协议不同。
- 这保证协议的所有要求都能被接受该协议的类型所见。
- 如果你定义了一个 public 的协议，该协议的规定要求在被实现时拥有一个 public 的访问级别

#### 8.1 协议继承

- 如果你定义了一个继承已有协议的协议，这个新协议最高与它继承的协议访问级别一致

  ```swift
  例如你不能写一个 public 的协议继承一个 internal 的协议。
  ```

#### 8.2 协议遵循

- 遵循了协议的类的访问级别取这个协议和该类的访问级别的最小者

  ```swift
  如果这个类型是 public 级别的，它所遵循的协议是 internal 级别，这个类型就是 internal 级别的。
  ```

### 9 扩展

- 扩展中添加的任何类型成员都有着被扩展类型相同的访问权限

  ```swift
  如果你扩展一个公开或者内部类型，你添加的任何新类型成员都拥有默认的内部访问权限。
  如果你扩展一个文件内私有的类型，你添加的任何新类型成员都拥有默认的私有访问权限。
  如果你扩展一个私有类型，你添加的任何新类型成员都拥有默认的私有访问权限。
  ```

- 你可以显式标注扩展的访问级别（例如， private extension ）已给扩展中的成员设置新的默认访问级别, 这个默认同样可以在扩展中为单个类型成员重写

- 你不能给用于协议遵循的扩展显式标注访问权限修饰符

- 扩展中使用协议自身的访问权限作为协议实现的默认访问权限。

#### 9.1 扩展中的私有成员

- 在同一文件中的扩展比如类、结构体或者枚举，可以写成类似多个部分的类型声明

  ```
  在原本的声明中声明一个私有成员，然后在同一文件的扩展中访问它；
  在扩展中声明一个私有成员，然后在同一文件的其他扩展中访问它；
  在扩展中声明一个私有成员，然后在同一文件的原本声明中访问它。
  ```

- 可以和组织代码一样使用扩展,从而使文件结构更为清晰

  ````swift
  protocol SomeProtocol {
      func doSomething()
  }
  
  struct SomeStruct {
      private var privateVariable = 12
  }
   
  extension SomeStruct: SomeProtocol {
      func doSomething() {
          print(privateVariable)
      }
  }
  
  ````

### 10 泛型

- 泛指类型和泛指函数的访问级别取泛指类型或函数以及泛型类型参数的访问级别的最小值。

### 11 类型别名

- 任何你定义的类型同义名都被视为不同的类型以进行访问控制
- 一个类型同义名的访问级别不高于原类型

## 二十六 高级运算符

### 1 位运算符

####  1.1 位取反运算符

- 位取反运算符（ ~ ）是对所有位的数字进行取反操作

  ```swift
  let initialBits: UInt8 = 0b00001111
  let invertedBits = ~initialBits  // equals 11110000
  ```

####  1.2 位与运算符

- 位与运算符（ & ）可以对两个数的比特位进行合并, ，只有当这两个数*都*是 1 的时候才能返回1

  ```swift
  let firstSixBits: UInt8 = 0b11111100
  let lastSixBits: UInt8 = 0b00111111
  let middleFourBits = firstSixBits & lastSixBits // equals 00111100
  ```

#### 1.3 位或运算符

- *位或运算符*（ | ）可以对两个比特位进行比较，然后返回一个新的数，只要两个操作位任意一个为 1 时，那么对应的位数就为 1 

  ```swift
  let someBits: UInt8 = 0b10110010
  let moreBits: UInt8 = 0b01011110
  let combinedbits = someBits | moreBits // equals 11111110
  ```

#### 1.4 位异或运算符 

- 位异或运算符，或者说“互斥或”（ ^ ）可以对两个数的比特位进行比较。它返回一个新的数，当两个操作数的对应位不相同时，该数的对应位就为 1 

  ```swift
  let firstBits: UInt8 = 0b00010100
  let otherBits: UInt8 = 0b00000101
  let outputBits = firstBits ^ otherBits // equals 00010001
  ```


### 2 位左移和右移运算符

- 位左移运算符（ << ）,把所有位数的数字向左移动一个确定的位数。
- 位右移运算符（ << ）,把所有位数的数字向右移动一个确定的位数。

#### 2.1 无符号整数的移位操作

- 对无符号整数的移位规则如下:

  1. 已经存在的比特位按指定的位数进行左移和右移

  2. 任何移动超出整型存储边界的位都会被丢弃。

  3. 用 0 来填充向左或向右移动后产生的空白位。

     ```swift
     1111 1111 << 1 变为 1 1111 1110 //右边末尾补0
     1111 1111 >> 1 变为   0111 1111 //左边开头补0
     
     let shiftBits: UInt8 = 4 // 00000100 in binary
     shiftBits << 1 // 00001000
     shiftBits << 2 // 00010000
     shiftBits << 5 // 10000000
     shiftBits << 6 // 00000000
     shiftBits >> 2 // 00000001
     ```


- 使用移位操作对其他的数据类型进行编码和解码

  ```swift
  let color: UInt32 = 0xCC6699aa 
  let redComponent = (color & 0xFF000000) >> 24
  //color & 0xFF000000为0xCC000000然后右移24位得到红色的值
  let greenComponent = (color & 0x00FF0000) >> 16 
  let blueComponent = (color & 0x0000FF00) >> 8
  let alphaComponent = (color & 0x000000FF) 
  ```

#### 2.2 有符号整型的位移操作

- 有符号整型使用它的第一位（所谓的*符号位*）来表示这个整数是正数还是负数, 其余的位数（所谓的*数值位*）存储了实际的值

- 号位为 0 表示为正数， 1 表示为负数。(以8进制进行演示)

- 正数: 符号位为0，另外7位则为值的二进制表示

  ```swift
  0 000 1000 = 4
  ```

- 负数: 符号位为0，它存储的是 2 的 n次方减去它的绝对值(n为数值位的位数，此时为7)

  ```swift
  1 111 1100 = -4 即为 -(2的7次方 - 124) = - (128 - 124) = - 4
  
  //负数相加 (-4) + (-1) = -5
  1 111 1100 = -4 
  1 111 1111 = -1
  1 111 1011 = -5 
  只需要将这两个数的全部八个比特位相加（包括符号位），并且将计算结果中超出的部分丢弃：
  
  ```

### 3 溢出运算符

- 当向一个整数赋超过它容量的值时，Swift 会报错而不是生成一个无效的数
- 这个行为给我们操作过大或着过小的数的时候提供了额外的安全性
- 溢出时，截断可用的位数，而不报错，可采用溢出运算符
  - 溢出加法 （ &+ ）
  - 溢出减法 （ &- ）
  - 溢出乘法 （ &* ）

#### 3.1 值溢出

- 数值可能出现向上溢出或向下溢出

  ```swift
  1. 无符号向上溢出
  var unsignedOverflow = UInt8.max
  // unsignedOverflow equals 255, which is the maximum value a UInt8 can hold
  unsignedOverflow = unsignedOverflow &+ 1
  // unsignedOverflow is now equal to 0
  
   1111 1111 UInt8.max
   0000 0001   1
  10000 0000   0 //溢出的位舍弃，所以结果为0->0000 0000
  
  2. 无符号向下溢出
  var unsignedOverflow = UInt8.min
  // unsignedOverflow equals 0, which is the minimum value a UInt8 can hold
  unsignedOverflow = unsignedOverflow &- 1
  // unsignedOverflow is now equal to 255
  
   0000 0000 UInt8.min
   0000 0001   1
   1111 1111   255 
  
  ```

### 4  优先级和结合性

- 注意运算符的优先级和结合性

### 5 运算符函数

- **运算符重载**: 类和结构体可以为现有的运算符提供自定义的实现

#### 5.1 中缀运算符

```swift
算术加法运算符(+)是一个二元运算符，同时也是一个中缀运算符

对一个结构体实现加法运算符的重载
struct Vector2D {
    var x = 0.0, y = 0.0
}
 
extension Vector2D {
    //该运算符函数被定义为一个全局函数
    static func + (left: Vector2D, right: Vector2D) -> Vector2D {
        return Vector2D(x: left.x + right.x, y: left.y + right.y)
    }
}

let vector = Vector2D(x: 3.0, y: 1.0)
let anotherVector = Vector2D(x: 2.0, y: 4.0)
let combinedVector = vector + anotherVector
// combinedVector is a Vector2D instance with values of (5.0, 5.0)
```



#### 5.2  前缀和后缀运算符

- 当运算符出现在目标之前，它就是前缀 （-a）

- 当它出现在操作目标之后时，它就是后缀运算符(比如b! )

- 要实现前缀运算符，需要在声明运算符函数的时候在 func  关键字之前指定 `prefix`限定符

- 要实现后缀运算符，需要在声明运算符函数的时候在 func  关键字之前指定 ` postfix`  限定符

  ```swift
  单目减运算符(-),也即前缀运算符
  extension Vector2D {
      static prefix func - (vector: Vector2D) -> Vector2D {
          return Vector2D(x: -vector.x, y: -vector.y)
      }
  }
  
  let positive = Vector2D(x: 3.0, y: 4.0)
  let negative = -positive
  // negative is a Vector2D instance with values of (-3.0, -4.0)
  let alsoPositive = -negative
  // alsoPositive is a Vector2D instance with values of (3.0, 4.0)
  
  ```


#### 5.3 组合赋值运算符

- **组合赋值运算符**将赋值运算符( = )与其它运算符进行结合

- 在实现的时候，需要把运算符的`左参数设置成 inout  类型`，因为这个参数的值会在运算符函数内直接被修改

  ```swift
  extension Vector2D {
      static func += (left: inout Vector2D, right: Vector2D) {
          left = left + right
      }
  }
  
  var original = Vector2D(x: 1.0, y: 2.0)
  let vectorToAdd = Vector2D(x: 3.0, y: 4.0)
  original += vectorToAdd
  // original now has values of (4.0, 6.0)
  ```

> 注意: 不能对默认的赋值运算符（ = ）进行重载。只有组合赋值运算符可以被重载。同样地，也无法对三元条件运算符 a ? b : c  进行重载.

#### 5.4 等价运算符

- 如果要使用 `==`,` !=`运算符检查你自己类型的等价，需要遵循Equatable协议

  ````swift
  extension Vector2D: Equatable {
      static func == (left: Vector2D, right: Vector2D) -> Bool {
          return (left.x == right.x) && (left.y == right.y)
      }
  }
  
  let twoThree = Vector2D(x: 2.0, y: 3.0)
  let anotherTwoThree = Vector2D(x: 2.0, y: 3.0)
  if twoThree == anotherTwoThree {
      print("These two vectors are equivalent.")
  }
  // Prints "These two vectors are equivalent."
  ````

### 6 自定义运算符

- 在 Swift 当中还可以声明和实现自定义运算符（custom operators）

- 新的运算符要在`全局作用域内`，使用` operator  `关键字进行声明

- 同时还要指定` prefix` 、 `infix`  或者` postfix`  限定符

  ````swift
  //声明自定义运算符 +++
  prefix operator +++ {}
  ````

  ```swift
  定义了一个新的名为 +++  的前缀运算符
  
  extension Vector2D {
      static prefix func +++ (vector: inout Vector2D) -> Vector2D {
          vector += vector
          return vector
      }
  }
   
  var toBeDoubled = Vector2D(x: 1.0, y: 4.0)
  let afterDoubling = +++toBeDoubled
  // toBeDoubled now has values of (2.0, 8.0)
  // afterDoubling also has values of (2.0, 8.0)
  ```

- 自定义中缀运算符的优先级和结合性

  1. 自定义的中缀（ infix ）运算符也可以指定优先级和结合性

  2. 结合性（ associativity ）可取的值有 left ， right  和 none 

  3. associativity 的默认值是 none ， precedence 默认为 100 

     ```swift
     //自定义运算符 +-
     infix operator +- { associativity left precedence 140 }
     extension Vector2D {
         static func +- (left: Vector2D, right: Vector2D) -> Vector2D {
             return Vector2D(x: left.x + right.x, y: left.y - right.y)
         }
     }
     let firstVector = Vector2D(x: 1.0, y: 2.0)
     let secondVector = Vector2D(x: 3.0, y: 4.0)
     let plusMinusVector = firstVector +- secondVector
     // plusMinusVector is a Vector2D instance with values of (4.0, -2.0
     
     //它的结合性和优先级被设置与 +  和 -  等默认的中缀加型运算符是相同的（ left  和 140 ）
     ```

  > 注意: 当定义前缀与后缀运算符的时候，我们并没有指定优先级。然而，如果对同一个操作数同时使用前缀与后缀运算符，则后缀运算符会先被应用。

