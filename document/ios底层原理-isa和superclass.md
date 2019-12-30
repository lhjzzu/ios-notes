## isa和superclass

### isa的作用

![](./images/isa和superclass0.png)

+ instance的isa指向class
  - 当调用对象方法时，通过instance的isa找到class，最后找到对象方法的实现进行调用
+ class的isa指向meta-class
  - 当调用类方法时，通过class的isa找到meta-class，最后找到类方法的实现进行调用

### class的superclass指针

![](./images/isa和superclass1.png)

+ 当Student的instance对象要调用Person的对象方法时，会先通过isa找到Student的class，然后通过superclass找到Person的class，最后找到对象方法的实现进行调用

### meta-class的superclass指针

![](./images/isa和superclass2.png)

+ 当Student的class要调用Person的类方法时，会先通过isa找到Student的meta-class，然后通过superclass找到Person的meta-class，最后找到类方法的实现进行调用



###  isa和superclass小结

![](./images/isa和superclass3.png)

+ instance的isa指向class

+ class的isa指向meta-class

+ meta-class的isa指向基类的meta-class

+ class的superclass指向父类的class

  - 如果没有父类，superclass指针为nil

+ meta-class的superclass指向父类的meta-class

  - 基类的meta-class的superclass指向基类的class

  - 因此如果一个类对象调用方法，但是没有实现该方法时，最终会找到NSObject对象的实例方法

    ![](./images/isa和superclass4.png)

+ instance调用对象方法的轨迹

  - isa找到class，方法不存在，就通过superclass找父类

+ class调用类方法的轨迹

  - isa找meta-class，方法不存在，就通过superclass找父类

### unrecognized selector

+ 找不到方法时的crash信息

  ![](./images/isa和superclass5.png)

  



### isa细节

![](./images/isa和superclass6.png)



+ 从64bit开始，isa需要进行一次位运算，才能计算出真实地址

  ```shell
  # if __arm64__
  #   define ISA_MASK        0x0000000ffffffff8ULL
  # elif __x86_64__
  #   define ISA             0x00007ffffffffff8ULL
  # endif
  ```

+ 在x86_64环境下

  ![](./images/isa和superclass7.png)

### class和meta-class的结构



### isa和superclass答疑