# 编译原理

## 1. 绪论
### 1.1 高级语言
翻译高级语言的程序--编译程序/编译器  
高级语言--编译器--机器码  

高级语言:  
- 直观, 自然, 易于理解
- 易读, 易写, 易于交流/出版和存档
- 一般独立于机器, 易于移植

### 1.2 强制式语言
语言分类:  
- 强制式语言(命令式语言, 基于命令)
- 函数式语言(基于数学函数)
- 逻辑式语言(基于数理逻辑的谓词演算)
- 对象式语言(基于抽象数据类型)

语言发展:  
- 第一代语言(机器语言, 依赖于机器指令系统)
- 第二代语言(汇编语言, 机器语言的符号化)
- 第三代语言(通常的高级语言, 命令式语言)
- 第四代语言(说明性语言, 告诉机器做什么)
- 新一代语言(理论基础和风格与高级语言不同, 函数式/逻辑式语言)

强制式语言的基础: 冯-诺伊曼体系结构  

冯-诺伊曼体系结构的特点:  
- 数据或指令以**二进制形式**存储
- 使用**存储程序**的工作方式(程序要执行必须先存储到存储器中)
- 程序**顺序**执行
- 存储器的内容可以被修改

冯-诺伊曼模型在高级语言中的呈现形式:  
- 变量: 存储单元由变量的概念来代替, 可以代表一个或者一组单元, 可以修改(变量是存储的抽象, 存储不仅存储指令, 还会存储数据)
- 赋值: 计算结果必须存储(赋值是对存储器的内容进行修改)
- 重复: 语句顺序执行, 指令存储在有限的存储器中, 完成复杂计算必须重复执行某些指令序列(循环语句)

高级语言的特征:  
绑定(binding): 实体与其属性建立联系的过程  
> 实体在声明(创建)的时候未必含有属性, 可能在编译或执行的时候才会明确指定实体的类型或属性.  
> 一个实体在什么时候拥有什么样的属性, 是一个很重要的时间节点.  

binding 的概念:
- 描述符: 编译程序中用来描述实体属性的表格的统称, 是实体到属性的映像.
- 静态绑定: 在编译时能确定的属性, 成为静态属性. 如果绑定在编译时完成, 运行时不改变, 称为静态绑定.
- 动态绑定: 实体的某些属性在运行时才能确定的属性. 绑定在运行时完成, 称为动态绑定.
> 在编译中, 静态仅指在编译时能确定的信息; 动态仅指运行时才能确定的信息.  
> 如: 动态数组和静态数组

**变量**:
> 冯-诺伊曼体系结构在高级语言中最重要的特性.
变量是一个或若干个存储单元的抽象. 赋值语句是修改存储单元内容的抽象.  
变量属性:
- 名称
- 作用域(空间概念)
  - 静态作用域绑定: 按照程序语法结构静态地定义变量的作用域
  - 动态作用域绑定: 按照程序的执行动态地定义变量的作用域
- 生存期(时间概念)
  - 数据对象: 存储区和它保存的值
  - 分配: 变量获得存储区的活动
- 值: 变量所对应存储区单元的内容
  - 匿名变量的访问通过指针实现
  - 变量与它的值的绑定是动态的
  - 符号常数的值不能修改
  - 变量的初始化, 几种处理方法:
    - 不初始化则出错
    - 随机
    - 缺省值 0
- 类型: 与变量相关联的值的类
  - 可以用来解释变量绑定的存储区的内容的意义
  - 语言定义时, 类型名通常绑定于某一个值类和某一组操作
  - 语言实现是, 值和操作绑定于某种机器的二进制表示及机器指令

虚拟机: 由软件实现的机器  

程序单元(Program Unit): 程序执行过程中的独立调用单位. 若干个编译好的单元结合起来, 组成一个完整的可执行程序.  

活动记录: 存储程序单元正常运行所需的所有信息.  

## 2. 数据类型
### 2.1 引言
数据类型是对存储器中所存储的数据进行的抽象. 包含了一组**值的集合**和一组**操作**.  

数据类型的作用:  
- **实现了数据抽象**: 不需要关注数据的底层实现.
- 是程序员从机器的具体特征中解脱出来.
- 提高了编程效率.

数据类型的分类:  
- 内部类型(build-in): 语言定义的
- 自定义类型(user-define): 用户定义的. 使用更多, 更重要.
  
### 2.2 内部类型
内部类型的特点:  
- 反映基本硬件特性(不同类型的机器指令不同).
- 在语言级, 内部类型标志共用某些操作的数据对象的抽象表示(值 + 操作的 数据对象的抽象).

内部类型的优势:  
- 基本表示的不可见性.
- **编译**时, 能检查变量使用的正确性.
- 编译时, 可以确定无二义的操作(整形的加法和浮点型的加法不同).
- 可以进行精度控制.

### 2.3 用户定义的类型
#### 2.3.1 笛卡尔积
在高级语言中, 笛卡尔积数据类型通常体现为记录或者结构, 由若干个域组成, 每个域有一个唯一的名字. 一般用域名来选取域, 对其进行修改.

#### 2.3.2 有限映像
从定义类型 DT(domain type) 的值的有限集合, 到值域类型 RT(range type) 的值的有限集合的函数称为有限映像.
- 在高级语言中, 通常体现为数组构造.
- 值域对象通过下标选取.
- 下标越界会出错, 动态检查.
- 下标可以用于选取值域的多个元素.
- 某些语言允许值域中的元素不是同一类型.
- DT 到相应值的特定子集的绑定策略:
  - 编译时绑定: 静态数组
  - 对象建立时绑定: 执行到分程序时, 动态数组
  - 对象处理时绑定: 子集范围可变
  
#### 2.3.3 序列
序列由任意多个数据项组成, 这些数据项称为该序列的成分, 且类型相同, 记为 CT.
> - 在高级语言中, 序列的体现是字符串和文件.
> - 序列和有限映像的区别在于序列长度是无限的.

#### 2.3.4 递归
如果数据类型包含属于同一类型 T 的成分, 那么类型 T 称为递归类型.
- 允许在类型定义中使用被定义类型的名字
- 指针是建立递归数据类型的重要手段

#### 2.3.5 判定或
判定或是一种选择对象结构的构造机制, 规定在两个不同选择对象之间做出适当的选择; 每一个选择对象结构称为**变体**.  
域数量或者类型都可能发生变化.  

#### 2.3.6 幂集
类型 T 的元素所有子集的集合, 称为幂集. 记为 Powerset(T), T 为基类型.
> 幂集类型的基本操作是集合的操作.

### 2.4 PASCAL 语言数据类型结构
#### 2.4.1 非结构类型
- 内部类型: integer, real, boolean, char
- 有序类型: 每一个元素都有唯一的前驱和后继(可以进行比较), 如: 整形, 布尔型, 字符型
- 定义新的有序类型的方法
  - 枚举型: 其值不能直接读写.
  - 子界型: 动态检查范围.

#### 2.4.2 聚合构造
数组构造: 构造符 `ARRAY` 允许定义有限映像; 数组构造一般形式: `array[t1] of [t2]`. `t1` 指定义域的类型, `t2` 为值域的类型.
```pascal
var arr1 = [1..50] of integer
var arr2 = [1..70] of integer
```  

#### 2.4.3 记录构造
使用构造符 `record` 来定义笛卡尔积:
```pascal
record field_1:type_1;
       field_2:type2;
       ...
       field_n:type_n;
end
```

也可以由变体记录(判定或的体现)

可以使用 `.` 来访问记录中的域.  

#### 2.4.4 文件构造
- PASCAL 文件是任意类型的多个元素的序列
- 只能顺序处理
- 只能进行 PUT 和 GET 操作
  
#### 2.4.5 指针
指针是 PASCAL 的第三类数据类型, 是非结构的, 可以用来构造递归结构.
- 指针可引用匿名数据对象, 这类对象由建立语句 `NEW` 显示分别配在堆上.
- 空指针的值是 `nil`
- 指针的操作: 赋值, 比较
- PASCAL 指针只能指向匿名数据对象, 不能指向在堆栈上分配的单元.

### 2.5 C 语言数据类型结构
#### 2.5.1 非结构类型
- 内部类型
- 用户自定义类型
- 非结构内部类型有整形, 实型(浮点型)和字符型.

#### 2.5.2 用户自定义的非结构类型
即枚举类型

#### 2.5.3 聚合构造
- 数组: 有限映像