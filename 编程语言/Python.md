# Python基础

>  [!IMPORTANT]
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

好的，下面是关于Python中模块和包的详细说明：

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















