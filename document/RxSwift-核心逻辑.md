## RxSwift-核心逻辑

### 创建序列并订阅

```swift
// 1: 创建序列
let observable = Observable<String>.create { 
              (observer) -> Disposable in
             // 3:发送信号
            observer.onNext("哈哈哈")
            return Disposables.create()  // 这个销毁不影响我们这次的解读
      }
//2: 订阅序列
observable.subscribe(onNext: { (text) in
                print("订阅到:\(text)")
})
            
        
```

### create方法

```swift
//Create.swift
extension ObservableType {
    public static func create(_ subscribe: @escaping (AnyObserver<E>) -> Disposable) -> Observable<E> {
        //匿名的可观察序列
        return AnonymousObservable(subscribe)
    }
}
```

+ 其内部创建了一个匿名的可观察序列，并且将subscribe这个闭包保存起来， 以便后面发送信号

+ subscribe这个闭包需要传入一个观察者(AnyObserver)  ，当这个observer观察新的消息时，通过不同的事件，将消息传递出去。

  ```swift
  //obserber通过next事件，将"哈哈哈"传递出去
  obserber.onNext("哈哈哈")
  ```

+ 那么后面的关键在于: 
  +  什么时候调用这个subscribe闭包 ? 
  + 怎么给这个闭包传递一个Observer?

### 匿名可观察序列AnonymousObservable

```swift
//Create.swift
//匿名可观察序列
final private class AnonymousObservable<Element>: Producer<Element> {
    typealias SubscribeHandler = (AnyObserver<Element>) -> Disposable

    let _subscribeHandler: SubscribeHandler
    //保存创建匿名观察者时传入的闭包，该闭包其接收一个Observer，用来发送观察到的事件消息
    init(_ subscribeHandler: @escaping SubscribeHandler) {
        self._subscribeHandler = subscribeHandler
    }
    .....
}
```

### subscribe-onNext:onError:onCompleted:onDisposed

```swift
 public func subscribe(onNext: ((E) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
```

````swift
//ObservableType + Extensions.swift
//只展示核心逻辑
//闭包可以理解为函数
extension ObservableType {
   ...
   ...
    public func subscribe(onNext: ((E) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
        -> Disposable {
            
          ...
          ...
            //创建了匿名的观察者,其接收一个闭包，并且闭包接收Event类型
            let observer = AnonymousObserver<E> { event in
                
                #if DEBUG
                 synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                switch event {
                //这个闭包接收到.next事件时，将对应的消息通过onNext传递给外界
                case .next(let value):
                    //
                    onNext?(value)
                //这个闭包接收到.error事件时，将对应的消息通过oneError传递给外界
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    //这个闭包接收到.completed事件时，将对应的消息通过onCompleted传递给外界
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
                //核心逻辑，通过这句代码，将observer传递给create可观察序列时传入的闭包
                self.asObservable().subscribe(observer),
                disposable
            )
    }
}

````

+ 创建了匿名的观察者AnonymousObserver

+ 其保存了一个闭包，对不同的事件,调用外界传递不同的闭包，将消息传递给外界

  ```swift
  
  onNext: ((E) -> Void)? = nil
  onError: ((Swift.Error) -> Void)? = nil
  onCompleted: (() -> Void)? = nil
  
  这个闭包接收到.next事件时，将对应的消息通过外界传入的onNext闭包传递给外界
  这个闭包接收到.error事件时，将对应的消息通过外界传入的oneError闭包传递给外界
  这个闭包接收到.completed事件时，将对应的消息通过外界传入的onCompleted闭包传递给外界
  ```

### 匿名观察者AnonymousObserver

```swift
//AnonymousObserver.swift
//匿名观察者
final class AnonymousObserver<ElementType> : ObserverBase<ElementType> {
    //重命名泛型ElementType为Element
    typealias Element = ElementType
   
     //外界传递的闭包的类型
    typealias EventHandler = (Event<Element>) -> Void
    
    private let _eventHandler : EventHandler
    
    init(_ eventHandler: @escaping EventHandler) {
#if TRACE_RESOURCES
       //自己手动维护一个类似引用计数的机制
        _ = Resources.incrementTotal()
#endif
        self._eventHandler = eventHandler
    }

    //通过不同的event，来调用初始化时传入的闭包
    override func onCore(_ event: Event<Element>) {
        return self._eventHandler(event)
    }
    
#if TRACE_RESOURCES
    deinit {
       //自己手动维护一个类似引用计数的机制
        _ = Resources.decrementTotal()
    }
#endif
}

//枚举类型
public enum Event<Element> {
    /// Next element is produced.
   //一下个元素已经产生, Element数据和next事件是绑定在一起的，传递事件本身就是传递数据
    case next(Element)
  
    /// Sequence terminated with an error.
    //因一个错误，序列中止
    case error(Swift.Error)
  
    /// Sequence completed successfully.
    /// 序列成功结束
    case completed
}
```

### subscribe-observer

```swift
override func subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == Element 
```

```swift
//匿名可观察序列继承Producer
final private class AnonymousObservable<Element>: Producer<Element>
   ...
   ...
   override func run<O : ObserverType>(_ observer: O, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where O.E == Element {
        //匿名可观察序列管道: AnonymousObservableSink
        let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
        let subscription = sink.run(self)
        return (sink: sink, subscription: subscription)
    }
}
class Producer<Element> : Observable<Element> {
    override init() {
        super.init()
    }

    override func subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == Element {
        if !CurrentThreadScheduler.isScheduleRequired {
            // The returned disposable needs to release all references once it was disposed.
            let disposer = SinkDisposer()
            let sinkAndSubscription = self.run(observer, cancel: disposer)
            disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

            return disposer
        }
        else {
             //核心逻辑
            return CurrentThreadScheduler.instance.schedule(()) { _ in
                let disposer = SinkDisposer()
                //调用run方法,这个方法由子类实现    
                //sinkAndSubscription是一个元组,调用子类的run方法                                                 
                let sinkAndSubscription = self.run(observer, cancel: disposer)
                disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)
                return disposer
            }
        }
    }
  
  //子类重写该run方法
  func run<O : ObserverType>(_ observer: O, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where O.E == Element {
        rxAbstractMethod()
    }
}
```

### AnonymousObservableSink

+ 建立AnonymousObservable和observer的关系
+ AnonymousObservable也遵守ObserverType协议

```swift

final private class AnonymousObservableSink<O: ObserverType>: Sink<O>, ObserverType {
    typealias E = O.E
    //重定义AnonymousObservable<E>为Parent
    typealias Parent = AnonymousObservable<E>

    // state
    private let _isStopped = AtomicInt(0)

    #if DEBUG
        fileprivate let _synchronizationTracker = SynchronizationTracker()
    #endif

    override init(observer: O, cancel: Cancelable) {
        super.init(observer: observer, cancel: cancel)
    }
    ....
    ....
    //parent即为AnonymousObservable
    //parent._subscribeHandler即为create该匿名可观察序列时保存的闭包。
    //AnyObserver(self)创建了一个observer对象，传递给AnonymousObservable的闭包进行调用
    func run(_ parent: Parent) -> Disposable {
        return parent._subscribeHandler(AnyObserver(self))
    }
}

```

+ parent即为AnonymousObservable

+ parent._subscribeHandler即为create该匿名可观察序列时保存的闭包。

  ```swift
   final private class AnonymousObservable<Element>: Producer<Element> {
      typealias SubscribeHandler = (AnyObserver<Element>) -> Disposable
      let _subscribeHandler: SubscribeHandler
      init(_ subscribeHandler: @escaping SubscribeHandler) {
          self._subscribeHandler = subscribeHandler
      }
  }
  ```

+ AnyObserver(self)创建了一个observer对象，传递给AnonymousObservable的闭包进行调用

###  AnyObserver

```swift
public struct AnyObserver<Element> : ObserverType {
    /// The type of elements in sequence that observer can observe.
    public typealias E = Element
    
    /// Anonymous event handler type.
    public typealias EventHandler = (Event<Element>) -> Void
    //observer为闭包
    private let observer: EventHandler
    //接着上面的流程，此时observer即为AnonymousObservableSink
    public init<O : ObserverType>(_ observer: O) where O.E == Element {
        //将AnonymousObservableSink的on方法传递给AnyObserver的observer变量，作为闭包
        self.observer = observer.on
    }
}


final private class AnonymousObservableSink<O: ObserverType>: Sink<O>, ObserverType {
    ...
    ...
    //AnonymousObservableSink的on方法，作为闭包传递给AnyObserver
    func on(_ event: Event<E>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        switch event {
        case .next:
            if load(self._isStopped) == 1 {
                return
            }
            self.forwardOn(event)
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.forwardOn(event)
                self.dispose()
            }
        }
    }
}
```

### observer通过事件，来传递消息

```swift
let observable = Observable<String>.create { 
              (observer) -> Disposable in
             //observer通过next事件，消息
            observer.onNext("哈哈哈")
            return Disposables.create()  // 这个销毁不影响我们这次的解读
}

//ObserverType.swift
extension ObserverType {
    
    //通过on方法, 触发next事件, 此时根据上面的流程on方法即为AnonymousObservableSink的on方法
    public func onNext(_ element: E) {
        self.on(.next(element))
    }
    //通过on方法,触发completed事件
    public func onCompleted() {
        self.on(.completed)
    }
    //通过on方法,触发error事件
    public func onError(_ error: Swift.Error) {
        self.on(.error(error))
    }
}

```



### AnonymousObservableSink的on方法

````swift
final private class AnonymousObservableSink<O: ObserverType>: Sink<O>, ObserverType {
    ...
    ...
    //AnonymousObservableSink的on方法，作为闭包传递给AnyObserver
    func on(_ event: Event<E>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        switch event {
        case .next:
            if load(self._isStopped) == 1 {
                return
            }
            //调用父类Sink的forwardOn方法
            self.forwardOn(event)
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.forwardOn(event)
                self.dispose()
            }
        }
    }
}

class Sink<O : ObserverType> : Disposable {
    fileprivate let _observer: O
    fileprivate let _cancel: Cancelable
    fileprivate let _disposed = AtomicInt(0)
    ...
    ...
    //_observer即为匿名观察者AnonymousObserver
    init(observer: O, cancel: Cancelable) {
        self._observer = observer
        self._cancel = cancel
    }

    final func forwardOn(_ event: Event<O.E>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        if isFlagSet(self._disposed, 1) {
            return
        }
        //触发AnonymousObserver的on方法
        self._observer.on(event)
    }
}

````

### AnonymousObserver的on方法

```swift
//AnonymousObserver.swift
//AnonymousObserver继承ObserverBase，从而调用ObserverBase的on方法
final class AnonymousObserver<ElementType> : ObserverBase<ElementType> {
    typealias Element = ElementType
    
    typealias EventHandler = (Event<Element>) -> Void
    
    private let _eventHandler : EventHandler
    
    init(_ eventHandler: @escaping EventHandler) {
#if TRACE_RESOURCES
        _ = Resources.incrementTotal()
#endif
        self._eventHandler = eventHandler
    }
    override func onCore(_ event: Event<Element>) {
         //调用创建AnonymousObserver时传入的闭包, 从而触发onNext, onError
        return self._eventHandler(event)
    }
    
#if TRACE_RESOURCES
    deinit {
        _ = Resources.decrementTotal()
    }
#endif
}

//ObserverBase.swift
class ObserverBase<ElementType> : Disposable, ObserverType {
    typealias E = ElementType

    private let _isStopped = AtomicInt(0)
    //调用ObserverBase的on方法
    func on(_ event: Event<E>) {
        switch event {
        case .next:
            if load(self._isStopped) == 0 {
                //调用子类的onCore方法，在此时即为调用AnonymousObserver的onCore方法
                self.onCore(event)
            }
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.onCore(event)
            }
        }
    }

    func onCore(_ event: Event<E>) {
        rxAbstractMethod()
    }

    func dispose() {
        fetchOr(self._isStopped, 1)
    }
}

```

1. AnonymousObserver继承ObserverBase，从而调用ObserverBase的on方法

2. 在on方法中, 调用子类的onCore方法，在此时即为调用AnonymousObserver的onCore方法

3. 调用创建AnonymousObserver时传入的闭包, 从而触发subscribe时，传入的onNext, onError, onCompleted闭包

   ```swift
    public func subscribe(onNext: ((E) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
   ```

### 简单总结

1. 通过`create`返回一个可观察序列`Observable`， 其实质是一个`AnonymousObservable`，并保存一个接收`AnyObserver`对象的闭包。 通过`AnyObserver`来观察`Observable序列`变化，从而在合适的时机，通过不同的事件，将消息(数据)传递给外界

2. 通过 `subscribe(onNext: onError:  onCompleted: onDisposed:)`方法进行订阅时，其内部创建了一个匿名的观察者(`AnonymousObserver`),并最终生成了一个`AnyObserver`对象，并将`AnyObserver`对象对象传递给`Observable`的 闭包。 该`AnyObserver`对象通过不同事件来传递数据时，其内部实质上是通过`AnonymousObserver`来使用对应的事件来传递数据，并触发创建AnonymousObserver时传入的闭包，从而根据不同的事件触发`subscribe(onNext: onError:  onCompleted: onDisposed:)`执行时，传入的对应的闭包。

   ```swift
   observer.onNext("哈哈哈")
   1. 此时observer为AnyObserver
   2. 当调用onNext方法，通过next事件传递数据"哈哈哈"时，其内部实际上是AnonymousObserver通过next事件传递数据"哈哈哈"
   3. 最终触发AnonymousObserver的onCore方法,调用创建AnonymousObserver传递的闭包, 根据不同的事件，将数据传递给外界
   ```

   ### RxSwift如何使用MVVM

   rxswift的好处就是实现了数据的双向流动，数据就流一样，我们可以对流进行中间加工处理，合并，最终得到我们想要的数据。

   

   rxswift怎么使用双向绑定进行mode - viewmodel - view之间的流动?

   

   对于view层的点击，输入，下拉等操作，作为可观察的序列，在viewModel中定义对应的观察者。数据从view层流动到viewModel进行处理。

   

   例如登录界面，我们把账号和密码，流动到viewModel。在viewModel中进行数据请求，并把请求完的数据，作为可观察的序列。最终被view层的观察者接收到。 这就实现了数据的双向绑定。

   

   例如点击的跳转，viewModel的Observer观察到tap事件进行跳转逻辑。

   

   这样把所有的业务逻辑放在viewModel层。在controller里我们只进行observable和observer的绑定。

   同时因为我们的UI界面是可变的，通过Protocol统一定义界面的行为。把UI层可观察的部分，例如tap，下拉等操作，都定义为可观察序列，然后通过接口来进行访问。这样就隔离了UI的变化，我们的业务层逻辑都放在viewModel中，保证了业务层逻辑的复用。controller层只做订阅， UI层订阅viewModel层， viewModel层订阅UI层

   

   我们同时支持iphone和ipad，同一个界面只有UI布局不一样，其业务逻辑完全一样，这样我只需要定义好UI层的Protocol，然后写两套布局并实现接口即可。

   

   

   

   

   

