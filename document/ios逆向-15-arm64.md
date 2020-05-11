## arm64

### iOS汇编

- 真机: arm64, GNU
- 模拟器: x86, AT&T

### 寄存器

- 通用寄存器
  - 64位的: x0~x28
  - 32位的: w0~w28 （属于x0~x28的低32位）
  - x0~x7通常用来存放函数的参数，更多的参数使用堆栈来传递
  - x0通常拿来存放函数的返回值
- 程序计数器
  - pc (Program Counter)
  - 记录CPU将要执行的是哪一条指令
  - 存储着CPU将要执行的指令的地址
  - 类似于8086汇编的ip寄存器
- 堆栈指针
  - sp (Stack Pointer):栈顶指针
  - fp (Frame Pointer): 栈底指针, 也就是x29
- 链接寄存器
  - lr (Linker Register), 也就是x30
  - 存储着函数的返回地址
- 程序状态寄存器
  - cpsr (Current Program Status Register)
  - spsr (Savad Program Status Register) , 异常状态下使用
- 零寄存器
  - wzr (32位， Word Zore Register)
  - xzr (64位)
  - 里面存储的值为0， 为了方便的将内存中的数据变为0

### LLDB指令操作寄存器

- 查看寄存器：` register read `

  ![](./images/arm0.png)

- 读/写某个寄存器: 

  - register read 寄存器

  - register write 寄存器  值

    ![](./images/arm1.png)

###  常见指令

![](./images/arm17.png)

![](./images/arm18.png)

### mov

- 数据传送指令

- mov 目的寄存器, 源操作数 : mov x0, x1 将x1寄存器的数据加载到x0寄存器中

  ![](./images/arm2.png)

+ 简单用法

  1. 建立arm.s文件，代码如下

     ![](./images/arm3.png)

     - test函数变成汇编代码时，会自动在前面加`_`
     - `.text`是告诉汇编器将编写汇编代码放在`__TEXT`的`__text`区域中
     - `.global`告诉汇编器_test函数是全局函数
     - 当我们进行数据操作时，具体操作多少个字节，不是根据源操作数来决定的。而是根据目的寄存器决定。
       * x0值为0时，mov x0 0x1111111122222222 ,读8个字节 此时x0为0x1111111122222222
       * x0值为0时，mov w0 0x1111111122222222,读4个字节, 此时x0为0x0000000022222222
     - 立即数前面要加`#`,并且立即数最好使用16进制
     - 注释用分号`;`

  2. 运行，查看x0寄存器值的变化

     ![](./images/arm4.png)

### add

- 加法指令

![](./images/arm5.png)

- 简单用法

  ![](./images/arm10.png)

  - x1, x0相加放到x2中

### sub

- 减法指令

![](./images/arm6.png)

- 简单用法

  ![](./images/arm11.png)

  - x0减x1的值放在x2中

### cpsr

- 存储程序当前的状态信息

  ![](./images/arm12.png)

  ![](./images/arm13.png)

### cmp

- 将2个寄存器相减

- 相减的结果会影响到cpsr寄存器的标志位

  ![](./images/arm7.png)

+ 简单使用

  ![](./images/arm14.png)
  
  ![](./images/arm15.png)

### b 

+  跳转指令

+ 可以带条件跳转，一般跟cmp配合使用

  ![](./images/arm8.png)

+ 指令的条件域

  - EQ: equal，相等
  - NE：not equal, 不相等
  - GT: great than , 大于
  - GE: greate equal ,大于等于
  - LE： less than, 小于
  - 简单按字面意思理解即可

  ![](./images/arm16.png)

+ 简单用法

  - 不带条件的跳转

    ![](./images/arm19.png)

    ![](./images/arm20.png)

  - 带条件的跳转

    ![](./images/arm21.png)

### bl

- 带返回的跳转指令

- 会在从当前函数跳转到其他函数之前，将返回时要执行的指令的地址存储到**lr(x30)**寄存器中

- 然后再跳转到标记处开始执行代码

- 在执行完函数后，ret指令会取出**lr(x30)**的地址赋值给pc寄存器

  ![](./images/arm9.png)

### ret

+ 函数返回

+ 将**lr (x30)**寄存器的值赋值给**pc**寄存器


### 寻址方式

+ 立即寻址

  ![](./images/arm22.png)

+ 寄存器寻址

  ![](./images/arm23.png)

+ 寄存器间接寻址

  ![](./images/arm24.png)

+ 基址变址寻址

  ![](./images/arm25.png)

+ 多寄存器寻址

  ![](./images/arm26.png)

+ 相对寻址

  ![](./images/arm27.png)

+ 堆栈寻址

  ![](./images/arm28.png)


### ldr

![](./images/arm33.png)

+ 从内存中读取数据

  - `ldr 寄存器  cpu寻址`

+ 简单使用

  ![](./images/arm29.png)

  ![](./images/arm30.png)

  - 预先定义` int a = 10`

### ldur

+ ldur的功能和ldr相同，都是从内存读取数据

+ 但从系统自动生成的汇编可知，当偏移量为负时，使用ldur

  ![](./images/arm31.png)

### ldp

+ 从内存读取数据，放到一对寄存器中
+ p是pair的简称，也就是一对
+ 指令格式示例： `ldp w0, w1, cpu寻址`
  - w0放低32位
  - w1放高32位

+ 简单使用

  ![](./images/arm32.png)

  - 预先定义` long a = 10;`



### str

![](./images/arm34.png)

![](./images/arm35.png)

+ 往内存中存储数据

+ 简单使用

  ![](./images/arm36.png)

  ![](./images/arm37.png)

  ![](./images/arm38.png)

### stur

+ 与str功能相同，但当偏移量为负时，使用stur

### stp

+ 将一对寄存器的数据，存储到内存中

+ 指令格式示例: `stp w0 ,w1, cpu寻址`

  - w0放在低32位，w1放在高32位

+ 简单使用

  ![](./images/arm39.png)

  ![](./images/arm40.png)

  ![](./images/arm41.png)

### orr

+ orr r0,r1,r2 
  -  将r1和r2或运算，然后将结果赋值给r0
  - r0=r1 | r2
+ orr r0,r1, #0xFF
  - 将r1和0xFF或运算，然后将结果赋值给r0
  - r0=r1 |0xFF

### 零寄存器

+ wzr (32位， Word Zore Register)

+ xzr (64位)

+ 里面存储的值为0， 为了方便的将内存中的数据变为0

  - 因为str不能直接将立即数赋值给内存，所以用零寄存器可以直接将内存中的赋值为0

    ```
    ### 将内存中数据赋值为0
    方案1 
    str #0,[x1] 错误，不能直接使用立即数
    
    方案2
    mov x2, #0
    str x2 [x1]
    
    方案3
    str xzr [x1]
    
    ### 方案3是最好的
    ```




### 程序计数器

+ pc寄存器

+ 简单演示

  ![](./images/arm42.png)

  - 断点在`0x1010da858`处,代码将要执行的汇编指令就是 `str x0, [sp, #0x20]`
  - 此时查看pc寄存器，其值为`0x1010da858`。说明pc寄存器里放着将要执行的指令。

### 链接寄存器

+ lr (Linker Register), 也就是x30

+ 存储着函数的返回地址

+ pc，lr， bl, ret的联系

  ![](./images/arm43.png)

  ![](./images/arm44.png)

  ![](./images/arm45.png)

  ![](./images/arm46.png)

  

### 函数的分类

+ 叶子函数:函数内部不会再调用其他函数
+ 非叶子函数:函数内部还会调用其他函数

### 叶子函数

+ 函数内部不会再调用其他函数

+ 对其堆栈过程进行分析

  1. 测试代码如下

     ![](./images/arm47.png)

  2. 编译生成汇编代码,代码精简后如下

     ```shell
     $ xcrun -sdk iphoneos clang -arch arm64 -S CTest.c -o CTest.s
     ```

     ![](./images/arm48.png)

  3. 通过汇编代码分析， 其堆栈平衡

     ![](./images/arm49.png)

### 非叶子函数

+ 函数内部不会再调用其他函数

+ 对其堆栈过程进行分析

  1. 测试代码如下

     ![](./images/arm54.png)

  2. 通过汇编代码分析， 其堆栈平衡

     ![](./images/arm50.png)

     ![](./images/arm51.png)

     ![](./images/arm52.png)

     ![](./images/arm53.png)

     - 上图来自于`./document/函数堆栈平衡.xlsx`文件

### 用hopper修改Mac汇编代码

1. 简单创建测试工程如下

   ![](./images/arm55.png)

2. 拿到可执行文件，在终端中执行结果如下

   ![](./images/arm56.png)

3. 利用hopper打开Test，假如经过一系列的分析，我们确认该段逻辑在main函数中

   ![](./images/arm57.png)

4. 利用hopper修改汇编代码

   ![](./images/arm58.png)

   ![](./images/arm59.png)

5. 生成新的可执行文件

   ![](./images/arm60.png)

6. 测试新的可执行文件

   ![](./images/arm61.png)

### Hopper修改App汇编代码

> 1. 去掉弹框
> 2. age改为20
> 3. 修改后只能装到越狱的手机中，因为签名已经被破坏。要装到非越狱手机中需要进行重签名

1. 建立iOS测试工程代码如下

   ```objective-c
   #import "ViewController.h"
   #import <objc/runtime.h>
   #import <malloc/malloc.h>
   
   @interface ViewController ()
   @property (weak, nonatomic) IBOutlet UILabel *label;
   @property (nonatomic, assign) int age;
   @property (nonatomic, weak) UIAlertAction *action;
   @end
   
   @implementation ViewController
   
   /*
    1.不再弹框
    2.my age is 20
    */
   
   int age = 10;
   
   - (void)viewDidLoad {
       [super viewDidLoad];
   
       self.label.text = [NSString stringWithFormat:@"my age is %d", age];
   }
   
   - (void)show
   {
       __weak typeof(self) weakSelf = self;
       
       UIAlertController *ac = [UIAlertController alertControllerWithTitle:@"提示" message:@"请输入密码" preferredStyle:UIAlertControllerStyleAlert];
       [ac addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
           [[NSNotificationCenter defaultCenter] addObserver:weakSelf selector:@selector(textFieldChanged:) name:UITextFieldTextDidChangeNotification object:textField];
       }];
       UIAlertAction *action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:nil];
       action.enabled = NO;
       [ac addAction:action];
       self.action = action;
       [self presentViewController:ac animated:YES completion:nil];
   }
   
   - (void)viewDidAppear:(BOOL)animated
   {
       [super viewDidAppear:animated];
       
       [self show];
   }
   
   
   - (void)didReceiveMemoryWarning {
       [super didReceiveMemoryWarning];
       // Dispose of any resources that can be recreated.
   }
   
   - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
   {
       
   }
   
   #pragma mark - UITextFieldDelegate
   - (void)textFieldChanged:(NSNotification *)note
   {
       self.action.enabled = [((UITextField *)note.object).text isEqualToString:@"12345"];
   }
   @end
   ```

2. 运行到越狱手机上，然后从手机上导出可执行文件

   ![](./images/arm62.png)

3. hopper打开应用，假如经过分析定位到弹窗代码如下

   ![](./images/arm63.png)

   ![](./images/arm64.png)

4. 利用MachOView定位全局变量的位置

   ![](./images/arm65.png)

   - 偏移量为0x90D8， 其在hopper中的地址为0x1000090D8

5. 在hopper中定位age,并修改其值

   ![](./images/arm66.png)

   ![](./images/arm67.png)

   ![](./images/arm68.png)

   ![](./images/arm69.png)

6. 生成新的可执行文件, 然后将新的可执行文件装到手机中即可

   ![](./images/arm70.png)