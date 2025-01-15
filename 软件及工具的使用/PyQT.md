# 参考链接

[Python 小白从零开始 PyQt5 项目实战（8）汇总篇（完整例程）_pyqt项目实战教程-CSDN博客](https://blog.csdn.net/youcans/article/details/120925109)

[免费学习编程 - Python、JavaScript、Java、Git 等 (freecodecamp.org)](https://www.freecodecamp.org/chinese/learn)

# UI

## fluent-widgets

参考链接：[快速上手 - PyQt-Fluent-Widgets](https://pyqt-fluent-widgets.readthedocs.io/zh-cn/latest/quick-start.html)

## silicon 

参考链接：[ChinaIceF/PyQt-SiliconUI: A powerful and artistic UI library based on PyQt5，基于 PyQt5 的UI框架，灵动、优雅而轻便 (github.com)](https://github.com/ChinaIceF/PyQt-SiliconUI)

# 安装

*`安装`python*

从官网下载：[Welcome to Python.org](https://www.python.org/)

*`IDE`的选择*

- ==VSCODE==（❤❤❤）：[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
- ==Cursor==(AI级选手，面向自然语言编程，amazing❗❗❗)：[Cursor](https://www.cursor.com/)
- ==Pycharm==（python专用工具）：[PyCharm: the Python IDE for data science and web development (jetbrains.com)](https://www.jetbrains.com/pycharm/)

*`python包管理工具`PIP*

`pip` 是 Python 包管理工具，用于安装和管理 Python 库和依赖。通过 `pip`，你可以方便地从 Python Package Index (PyPI) 下载和安装第三方库，并自动解决依赖问题。

常见的 `pip` 使用命令：
1. **安装包**：`pip install 包名`
2. **卸载包**：`pip uninstall 包名`
3. **查看已安装的包**：`pip list`
4. **升级包**：`pip install --upgrade 包名`
5. **查看包信息**：`pip show 包名`

例如，安装 NumPy 库的命令是：
```bash
pip install numpy
```

*`安装`Pyqt5*

使用PIP来安装pyqt

```bash
pip install pyqt5 -i https://mirrors.aliuyun.com/pypi/simple
```

> [!tip]
>
> 建议先不换源下载试试，如果不可以在使用-i参数换源

*`安装`QtTool*

Qt Tools 包含：

- 图形界面设计工具 Qt Designer，用于设计图形界面，生成 .ui文件，以 xml 格式存储界面和控件的属性；
- UI 文件转换工具 PyUic，用于将 .ui 文件解析为 .py 文件的工具。

```bash
pip install pyqt5-tools  -i https://mirrors.aliuyun.com/pypi/simple
```

> [!TIP]
>
> 注意：这里使用的是python版本需要低于3.10，如果高于3.10，那么请分步安装软件：
>
> ```bash
> pip install PyQt5Designer
> ```

# Python基础

> [!IMPORTANT]
>
> Python语言有着严格的缩进模式，同一个Tab表示在同一个作用域（和Makefile同样的Tab控制）
>
> 在Python中的缩进相当于是C语言中的`{}`，用于控制作用域
>
> 注意缩进❗❗❗❗❗❗
>
> 否则会导致程序报错或者出现很多奇奇怪怪的bug🙉

## 数据类型

在Python中无需定义变量的数据类型，编译器会自动解析当前定义的变量是什么数据类型。

###  整数 (int)
- **定义**：表示整数值。
- **基本用法**：
  ```python
  # 定义整数
  age = 25
  print(age)  # 输出：25
  
  # 基本运算
  sum = age + 5
  print(sum)  # 输出：30
  ```

###  浮点数 (float)
- **定义**：表示带小数的数字。
- **基本用法**：
  ```python
  # 定义浮点数
  temperature = 36.6
  print(temperature)  # 输出：36.6
  
  # 基本运算
  area = 3.14 * (5 ** 2)
  print(area)  # 输出：78.5
  ```

### 字符串 (str)
- **定义**：表示文本数据，使用单引号或双引号包裹。
- **基本用法**：
  ```python
  # 定义字符串
  name = "Alice"
  print(name)  # 输出：Alice
  
  # 字符串拼接
  greeting = "Hello, " + name + "!"
  print(greeting)  # 输出：Hello, Alice!
  
  # 字符串方法
  upper_name = name.upper()
  print(upper_name)  # 输出：ALICE
  ```

### 布尔值 (bool)
- **定义**：表示真或假，只有两个值：`True` 和 `False`。
- **基本用法**：
  ```python
  # 定义布尔值
  is_student = True
  print(is_student)  # 输出：True
  
  # 布尔运算
  has_passed = True
  is_adult = False
  print(has_passed and is_adult)  # 输出：False
  ```

### 列表 (list)
- **定义**：有序可变集合，可以存储不同类型的元素。（C++中的vector）
- **基本用法**：
  
  ```python
  # 定义列表
  fruits = ["apple", "banana", "cherry"]
  print(fruits)  # 输出：['apple', 'banana', 'cherry']
  
  # 访问元素
  first_fruit = fruits[0]
  print(first_fruit)  # 输出：apple
  
  # 添加元素
  fruits.append("orange")
  print(fruits)  # 输出：['apple', 'banana', 'cherry', 'orange']
  
  # 删除元素
  fruits.remove("banana")
  print(fruits)  # 输出：['apple', 'cherry', 'orange']
  ```

### 元组 (tuple)
- **定义**：有序不可变集合，类似于列表，但不能修改元素。
- **基本用法**：
  ```python
  # 定义元组
  coordinates = (10.0, 20.0)
  print(coordinates)  # 输出：(10.0, 20.0)
  
  # 访问元素
  x = coordinates[0]
  print(x)  # 输出：10.0
  ```

### 字典 (dict)
- **定义**：无序可变集合，用键值对存储数据。（C++中的hash map）
- **基本用法**：
  ```python
  # 定义字典
  person = {"name": "Alice", "age": 30}
  print(person)  # 输出：{'name': 'Alice', 'age': 30}
  
  # 访问值
  name = person["name"]
  print(name)  # 输出：Alice
  
  # 添加新键值对
  person["city"] = "New York"
  print(person)  # 输出：{'name': 'Alice', 'age': 30, 'city': 'New York'}
  ```

### 集合 (set)
- **定义**：无序不重复元素的集合，适用于需要唯一性的数据。（C++中的set）
- **基本用法**：
  ```python
  # 定义集合
  unique_numbers = {1, 2, 3, 3}
  print(unique_numbers)  # 输出：{1, 2, 3}
  
  # 添加元素
  unique_numbers.add(4)
  print(unique_numbers)  # 输出：{1, 2, 3, 4}
  
  # 删除元素
  unique_numbers.remove(2)
  print(unique_numbers)  # 输出：{1, 3, 4}
  ```

## 控制结构

### 条件语句
条件语句用于根据特定条件执行不同的代码块。

#### if 语句
```python
# 定义变量
score = 85

# 使用 if 语句
if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 70:
    print("中等")
else:
    print("需要努力")
```

> [!TIP]
>
> 逻辑表达式有`or`、`not`、`and`,对应c中的`||`、`！`、`&&`

#### 其他条件语句

- **短路条件**：
  
  ```python
  is_pass = True
  is_registered = True
  
  if is_pass and is_registered:
      print("可以参加考试")
  ```
  
- **条件表达式 (三元运算符，行迭代器)**：
  
  ```python
  age = 18
  status = "成人" if age >= 18 else "未成年人"
  print(status)  # 输出：成人
  ```

### 循环语句
循环语句用于重复执行特定代码块，直到满足退出条件。

#### for 循环
用于遍历序列（如列表、字符串等）。
```python
# 遍历列表
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# 使用 range 函数
for i in range(5):
    print(i)  # 输出：0, 1, 2, 3, 4
```

#### while 循环
在满足条件时重复执行代码块。
```python
count = 0
while count < 5:
    print(count)
    count += 1  # 输出：0, 1, 2, 3, 4
```

###  跳转语句
跳转语句用于改变程序的执行顺序。

#### break
用于立即退出循环。
```python
for i in range(10):
    if i == 5:
        break
    print(i)  # 输出：0, 1, 2, 3, 4
```

#### continue
用于跳过当前循环的剩余部分，进入下次迭代。
```python
for i in range(5):
    if i == 2:
        continue
    print(i)  # 输出：0, 1, 3, 4
```

#### pass
用于占位，表示一个空的代码块，通常用于结构体。
```python
if True:
    pass  # 这里什么都不做，但可以保持代码结构完整
```

函数是组织和重用代码的基本构件，允许你将特定的任务封装成一个可重复调用的块。Python 中的函数可以有参数和返回值。以下是函数的详细介绍：

## 函数

### 定义函数

使用 `def` 关键字来定义函数。

```python
def greet(name):
    print(f"Hello, {name}!")
```

### 调用函数
定义函数后，可以通过函数名和括号调用它。

```python
greet("Alice")  # 输出：Hello, Alice!
```

### 函数参数
函数可以接受参数，支持位置参数和关键字参数。

#### 位置参数
```python
def add(a, b):
    return a + b

result = add(3, 5)
print(result)  # 输出：8
```

#### 关键字参数
可以在调用时指定参数名称。
```python
def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type} named {pet_name}.")

describe_pet(animal_type="dog", pet_name="Buddy")
```

### 默认参数
可以为参数指定默认值，如果调用时不传入该参数，则使用默认值。

```python
def power(base, exponent=2):
    return base ** exponent

print(power(4))      # 输出：16
print(power(4, 3))   # 输出：64
```

### 可变参数
使用 `*args` 和 `**kwargs` 来处理不确定数量的参数。

#### *args
接收任意数量的位置参数。
```python
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3))  # 输出：6
```

#### **kwargs
接收任意数量的关键字参数。
```python
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30)
```

###  返回值
使用 `return` 语句返回值。如果没有 `return`，函数默认返回 `None`。

```python
def multiply(x, y):
    return x * y

result = multiply(4, 5)
print(result)  # 输出：20
```

###  文档字符串
可以为函数添加文档字符串，用于描述函数的功能。

```python
def factorial(n):
    """计算 n 的阶乘"""
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)

print(factorial.__doc__)  # 输出：计算 n 的阶乘
```

###  函数作为对象
函数是第一类对象，可以赋值给变量，作为参数传递，或者作为返回值。

```python
def square(x):
    return x * x

def apply_function(func, value):
    return func(value)

result = apply_function(square, 5)
print(result)  # 输出：25
```

## 面向对象编程

### 类与对象
- **类**是对象的蓝图或模板，定义了一组属性和方法。
- **对象**是类的实例，包含类定义的属性和方法的具体值。

#### 定义类
```python
class Dog:
    # 属性
    species = "Canis lupus familiaris"

    def __init__(self, name, age):
        self.name = name  # 实例属性
        self.age = age

    # 方法
    def bark(self):
        return f"{self.name} says Woof!"
```

#### 创建对象
```python
my_dog = Dog("Buddy", 3)
print(my_dog.name)  # 输出：Buddy
print(my_dog.bark())  # 输出：Buddy says Woof!
```

### 属性
属性是对象的特征，分为实例属性和类属性。
- **实例属性**：特定于某个对象的属性。
- **类属性**：属于类本身的属性，所有对象共享。

```python
print(my_dog.species)  # 输出：Canis lupus familiaris
```

### 方法
方法是定义在类中的函数，用于执行与对象相关的操作。

```python
class Circle:
    pi = 3.14  # 类属性

    def __init__(self, radius):
        self.radius = radius  # 实例属性

    def area(self):
        return Circle.pi * (self.radius ** 2)  # 方法访问类属性
```

###  继承
继承是面向对象编程的重要特性，可以创建一个新类，从现有类继承属性和方法。

```python
class Labrador(Dog):  # 继承自 Dog 类
    def fetch(self):
        return f"{self.name} is fetching!"

my_lab = Labrador("Rex", 2)
print(my_lab.fetch())  # 输出：Rex is fetching!
```

###  多态
多态允许不同类的对象以相同的方式调用相同的方法，具体的行为由对象的类型决定。

```python
class Cat:
    def bark(self):
        return "Meow!"

def make_sound(animal):
    print(animal.bark())

make_sound(my_dog)  # 输出：Buddy says Woof!
make_sound(Cat())   # 输出：Meow!
```

### 封装
封装是将对象的状态（属性）与行为（方法）结合在一起，限制外部直接访问对象的内部状态。可以通过命名约定和 getter/setter 方法实现。

```python
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance  # 私有属性

    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount

    def get_balance(self):
        return self.__balance

account = BankAccount()
account.deposit(100)
print(account.get_balance())  # 输出：100
```

## 模块

1. **定义**：模块是一个包含Python代码的文件，文件名以`.py`结尾。模块可以包含函数、类和变量，方便代码的组织和重用。

2. **导入模块**：

   - 使用`import`语句导入模块，例如：

     ```python
     import math
     print(math.sqrt(16))  # 输出 4.0
     ```

   - 使用`from ... import ...`导入特定的函数或变量，例如：

     ```python
     from math import pi
     print(pi)  # 输出 3.141592653589793
     ```

3. **模块的创建**：创建模块只需将相关的函数和变量写入一个`.py`文件中。例如，创建一个名为`mymodule.py`的文件，内容如下：

   ```python
   def greet(name):
       return f"Hello, {name}!"
   ```

4. **模块的作用域**：模块内的变量和函数具有模块作用域，避免与其他模块的命名冲突。

## 包

1. **定义**：包是一个包含多个模块的文件夹，用于组织相关模块。包通过`__init__.py`文件标识，可以是空文件，也可以包含包的初始化代码。

2. **创建包**：创建一个包只需创建一个文件夹并在其中添加模块和一个`__init__.py`文件。例如，创建一个名为`mypackage`的文件夹，里面有`module1.py`和`module2.py`，并添加一个空的`__init__.py`文件。

3. **导入包**：

   - 导入整个包：

     ```python
     import mypackage
     ```

   - 导入包中的特定模块：

     ```python
     from mypackage import module1
     ```

   - 导入包中的特定函数或类：

     ```python
     from mypackage.module1 import my_function
     ```

4. **命名空间**：包提供了命名空间，可以避免模块之间的命名冲突。

### 示例

假设我们有如下目录结构：

```
mypackage/
    __init__.py
    module1.py
    module2.py
```

在`module1.py`中：

```python
def func1():
    return "Function 1 from module 1"
```

在`module2.py`中：

```python
def func2():
    return "Function 2 from module 2"
```

在主程序中使用包：

```python
from mypackage import module1, module2

print(module1.func1())  # 输出 "Function 1 from module 1"
print(module2.func2())  # 输出 "Function 2 from module 2"
```

# 小栗子

## 新建

打开designer，依次点击*==新建窗口>Main window==*创建一个软件页面

![动画](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_19_35_15_202410231935990.gif)

软件页面

![软件页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_19_34_58_202410231934330.png)

## 操作步骤

==信号与槽的原理==：

简单地说，将信号与槽函数连接后，当信号被触发时，槽函数将被自动调用。

分析这个过程，涉及到几个基本概念和关系：

- 信号：信号可以是一个动作，也可以是对象的一种状态，用于触发所连接的槽。

- 槽：槽就是一个函数，可以由连接的信号触发。
- 发射信号：信号被发射时，自动调用信号连接的槽函数。通常，在对象的状态改变时发射信号。
- 信号可以带有参数，但必须与槽函数的参数相对应。
- 一个信号可以连接多个槽函数，多个信号也可以连接到一个槽函数。
- 一个信号可以连接到另一个信号上。
- 很多窗口部件（控件）内置了一下信号和槽，可以直接调用。也可以按需求自定义信号和槽。

==信号发送者与槽的接收者==：

- 信号的发送者通常是一个控件对象，在控件对象的状态发生变化时发送信号。常见的发送者是图形窗口中的各种控件对象，但也可以是动作对象。
- 槽的接收者通常也是控件对象。槽函数是一个自定义的槽函数，或控件内置的槽函数。一般地，槽函数也有一个对象作为主体，即对于接受者这个控件对象执行函数定义的操作。例如槽函数执行的功能是关闭，哪么究竟是关闭那个控件呢？关闭对象就是接受者。


> [!NOTE]
>
> 举个例子，在一个十字路口，信号灯变成了绿色，对面的汽车看到后就启动了。
>
> - 信号灯就是发送信号的对象，绿灯亮是它发送的信号 (signal)。
> - 汽车是接收对象，汽车行驶是汽车对信号的响应，也叫槽 (slot)。

### 需求

> 1. 点击一下按钮，输入框中的文字就消失
> 2. 点击一下按钮，输入框中就显示“不要再点啦”

### 实现

需求1：

> 首先解析需求：
>
> > ==按钮是信号的发送者，输入框是接收者。按钮按下是信号，清空输入框中的文字是槽。==
>
> 首先从左侧的Widget Box中拉取一个push button，双击该push button即可修改button的文字。
>
> 之后从左侧的Widget Box中拉取一个Line Edit。
>
> ![拉去控件](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_19_58_30_202410231958540.gif)
>
> 将发送者和接收者连在一起：
>
> > 方法1：
> >
> > 1. 点击菜单栏的*==Edit》编辑信号\槽（F4）==*
> >
> > 2. 首先选中信号发送者，鼠标左键按住拖动会发现出现一个箭头，当拖动箭头到另外一个组件的时候，会将两个组件连接在一起，同时会出现配置连接界面，用于控制信号和槽
> >
> > 3. 此时需要选择信号和槽，左边是信号（按压动作，也就是clicked），右边栏是槽（清空函数，也就是clear）
> >
> > 4. 完成之后点击ok即可
> >
> >    ![连接信号和槽](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_20_9_21_202410232009568.gif)
> >
> > 方法2：
> >
> > 1. 点击右侧的信号槽，点击左上角加号增加一个连接
> >
> > 2. 依次选择发送者、信号、接收者、槽即可
> >
> >    ![选择式](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_20_11_36_202410232011449.png)
>
> 完成UI设计之后保存。将保存的ui文件转换为python文件，使用以下指令：
>
> ```
> python -m PyQt5.uic.pyuic .\mainWindow.ui -o mainWindow.py
> ```
>
> 使用完命令之后就会发现在同级目录下生成了py文件。
>
> 新建一个python文件，用来运行，代码如下：
>
> ```python
> import sys
> from PyQt5 import QtWidgets
> from mainWindow import Ui_MainWindow  # 导入你的UI类
> 
> class MainWindow(QtWidgets.QMainWindow):
>     def __init__(self):
>         super().__init__()	#调用父类的构造函数
>         self.ui = Ui_MainWindow()
>         self.ui.setupUi(self)
> 
> if __name__ == '__main__':
>     app = QtWidgets.QApplication(sys.argv)
>     mainWin = MainWindow()
>     mainWin.show()
>     sys.exit(app.exec_())
> 
> ```
>
> > [!TIP]
> >
> > super用于调用父类的函数，这里的`super().__init__()`就是调用父类的`__init__（）`
> >
> > 当然，也可以调用父类的函数。现考虑以下程序会输出什么？
> >
> > ```python
> > class A:
> >     def greet(self):
> >         print("Hello from A")
> > 
> > class B(A):
> >     def greet(self):
> >         print("Hello from B")
> >         super().greet()  # 调用父类 A 的 greet 方法
> > 
> > class C(A):
> >     def greet(self):
> >         print("Hello from C")
> >         super().greet()  # 调用父类 A 的 greet 方法
> > 
> > class D(B, C):
> >     def greet(self):
> >         print("Hello from D")
> >         super().greet()  # 这样会依照方法解析顺序调用 B 和 C 的 greet 方法
> > 
> > d = D()
> > d.greet()
> > ```
> >
> > ```
> > Hello from D
> > Hello from B
> > Hello from C
> > Hello from A
> > ```
> >
> > 在函数调用时，遵从C3线性算法：
> >
> > 1. **保证优先级**：C3线性化确保了子类在调用方法时，父类的顺序是可信的，子类总是在父类之前得到调用。
> > 2. **无歧义性**：如果一个类有多个父类，C3算法会生成一个唯一的线性顺序，从而消除方法解析过程中的歧义。
> > 3. **遵循先父后子的原则**：在确定顺序时，确保父类在子类之前，且继承树的任何变化都会反映在新的线性化顺序中。
> >
> > C3算法的基本思想是为每个类维护一个序列，这个序列从当前类开始，依次添加类的父名单，并遵循以下原则：
> >
> > 1. 如果类A继承类B和类C，则在确定类A的MRO（Method Resolution Order）时，类B和类C的优先级要遵循它们的MRO，可以使用`D.mro()`来查看类的调用顺序。
> > 2. 类B的所有父类在类A的MRO中必须在类B之后出现。
> > 3. 类C的所有父类在类A的MRO中必须在类C之后出现。
>
> > [!TIP]
> >
> > 构造函数是用于初始化对象的一种特殊方法。在Python中，构造函数是`__init__`方法。当一个对象被创建时，Python会自动调用这个方法，以便为对象的属性赋初始值并进行必要的设置。
> >
> > 构造函数的特点:
> >
> > 1. **名称**：构造函数在Python中总是被命名为`__init__`，这是一个特殊的方法名。
> > 2. **参数**：构造函数可以接受参数，用于传递需要初始化的值。第一个参数通常是`self`，它代表实例本身，后面的参数则可以根据需要自定义。
> > 3. **自动调用**：当创建对象时，例如`obj = ClassName()`, Python会自动调用相应的构造函数。
> >
> > 示例：
> >
> > ```python
> > class Car:
> >     def __init__(self, make, model, year):
> >         self.make = make   # 品牌
> >         self.model = model # 型号
> >         self.year = year   # 年份
> > 
> >     def display_info(self):
> >         print(f"Car: {self.year} {self.make} {self.model}")
> > 
> > # 创建对象时将调用构造函数
> > my_car = Car("Toyota", "Camry", 2020)
> > my_car.display_info()
> > ```
> >
> > 输出
> >
> > ```
> > Car: 2020 Toyota Camry
> > ```



需求2：

> 由于显示特定文字，接收者（Line edit）就是Main window，所以我们的指向应该指向“地”。
>
> 在建立连接之前，需要先申明一个我们自定义的函数，在`对象查看器`中选中QMainWindow，右键，选择`改变信号\槽`，在槽中新建一个自己的函数
>
> ![新建槽函数](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_20_39_8_202410232039010.gif)
>
> 然后从按钮出发接地即可
>
> ![动画](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/10/23_20_41_27_202410232041717.gif)
>
> 和上面一样，先将文件转化为python文件，之后再新建一个python文件，此时我们需要实现刚刚定义的函数：
>
> ```python
> import sys
> from PyQt5 import QtWidgets
> from mainWindow import Ui_MainWindow  # 导入你的UI类
> 
> class MainWindow(QtWidgets.QMainWindow):
>  def __init__(self):
>      super().__init__()
>      self.ui = Ui_MainWindow()
>      self.ui.setupUi(self)
> 
>  def showing(self):
>      print("showing")
>      self.ui.lineEdit.setText("不要再点啦")
> 
> if __name__ == '__main__':
>  app = QtWidgets.QApplication(sys.argv)
>  mainWin = MainWindow()
>  mainWin.show()
>  sys.exit(app.exec_())
> ```
>
> > [!NOTE]
> >
> > ```python
> > import sys
> > from PyQt5 import QtWidgets
> > from mainWindow import Ui_MainWindow  # 导入你的UI类
> > 
> > class MainWindow(Ui_MainWindow,QtWidgets.QMainWindow):
> >      def __init__(self):
> >            super().__init__()
> >            self.setupUi(self)
> >     
> >      def showing(self):
> >            print("showing")
> >            self.lineEdit.setText("不要再点啦")
> > 
> > if __name__ == '__main__':
> >      app = QtWidgets.QApplication(sys.argv)
> >      mainWin = MainWindow()
> >      mainWin.show()
> >      sys.exit(app.exec_())
> >  
> > ```
> >
> > 

最后打包软件：

```bash
pyinstaller -D --noconsole  main.py  
```



#  技巧

将ui文件转换为python文件

```bash
python -m PyQt5.uic.pyuic .\mainWindow.ui -o mainWindow.py
```

打包软件

```bash
pyinstaller -D -i .\resource\windowIcon.ico --noconsole  main.py  
```

设置字体能跟随分辨率

```python
 QtWidgets.QApplication.setAttribute(QtCore.Qt.AA_EnableHighDpiScaling, True)
 QtGui.QGuiApplication.setHighDpiScaleFactorRoundingPolicy(QtCore.Qt.HighDpiScaleFactorRoundingPolicy.PassThrough)   
```

设置下拉框中选项的距离

```
# m_sectionShape是组件名称
self.m_sectionShape.setStyleSheet("QAbstractItemView::item { height: 20px}")
self.m_sectionShape.setView(QListView())
```































































