# 数据类型

## vector

### 增加数据

在C++中，可以使用`std::vector`来动态管理数据。要向`std::vector`中增加数据，有几种常用的方法：

1. **使用`push_back`方法**：向`vector`的末尾添加一个元素。
   
    ```cpp
    myVector.push_back(10);
    ```
2. **使用`emplace_back`方法**：在`vector`的末尾原地构造一个元素。这通常比`push_back`更高效，尤其是在处理复杂对象时。
   
    ```cpp
    myVector.emplace_back(30);
    ```
    
3. **使用`insert`方法**：在`vector`的指定位置插入一个或多个元素。
    ```cpp
    myVector.insert(myVector.begin() + 1, 40);
    ```

4. **使用`resize`方法**：调整`vector`的大小，并对新增加的元素进行初始化。
    ```cpp
     // 将vector大小调整为5，并用0初始化新增的元素
    myVector.resize(5, 0);
    ```
    
5. **使用`assign`方法**：给`vector`分配新的内容，替换其当前内容。
    ```cpp
    myVector.assign({1, 2, 3, 4, 5});
    ```

## hash map

### 引入头文件

首先需要包含 `unordered_map` 头文件：

```cpp
#include <unordered_map>
```

### 基本用法

1. **定义一个 `unordered_map`：**

   ```cpp
   std::unordered_map<int, std::string> myMap;
   ```

   这里定义了一个 `unordered_map`，键是 `int` 类型，值是 `std::string` 类型。

2. **插入元素：**

   可以通过 `[]` 运算符或 `insert` 函数来插入元素。

   ```cpp
   myMap[1] = "Hello";
   myMap[2] = "World";

   // 使用 insert 函数
   myMap.insert({3, "C++"});
   ```

3. **访问元素：**

   可以使用键访问元素，如果键不存在则会插入一个默认值。

   ```cpp
   std::string value = myMap[1]; // "Hello"
   ```

   如果不确定键是否存在，可以使用 `at` 方法，这样在键不存在时会抛出 `std::out_of_range` 异常。

   ```cpp
   try {
       std::string value = myMap.at(4); // 会抛出异常
   } catch (const std::out_of_range& e) {
       std::cout << "Key not found" << std::endl;
   }
   ```

4. **查找元素：**

   可以使用 `find` 函数来查找元素，返回一个迭代器。

   ```cpp
   auto it = myMap.find(2);
   if (it != myMap.end()) {
       std::cout << "Found: " << it->second << std::endl; // 输出 "Found: World"
   } else {
       std::cout << "Not found" << std::endl;
   }
   ```

5. **遍历 `unordered_map`：**

   可以使用范围 `for` 循环或迭代器来遍历所有键值对。

   ```cpp
   for (const auto& pair : myMap) {
       std::cout << pair.first << ": " << pair.second << std::endl;
   }
   ```

6. **删除元素：**

   可以使用 `erase` 方法删除某个键值对。

   ```cpp
   myMap.erase(1); // 删除键为 1 的元素
   ```

7. **获取大小和检查是否为空：**

   ```cpp
   std::cout << "Size: " << myMap.size() << std::endl;
   if (myMap.empty()) {
       std::cout << "Map is empty" << std::endl;
   }
   ```













