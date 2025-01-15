#  批处理基础

## 注释
- **`REM`** 或 **`::`** 用于添加注释。注释不会被执行。

  ```batch
  REM 这是一个注释
  :: 这也是一个注释
  ```

## 显示文本
- **`echo`** 显示文本。`@echo off` 可以关闭回显（即关闭命令本身的显示），但保留输出。

  ```batch
  echo Hello, World!
  @echo off
  echo 这行代码只显示输出，不显示命令本身
  ```

## 变量
- **`set`** 用于定义变量，**`%variable%`** 用于访问变量值。

  ```batch
  set myVar=Hello
  echo %myVar% World!
  ```

- **`set /p`** 从用户输入中获取变量值。

  ```batch
  set /p userInput=Please enter your name: 
  echo Hello, %userInput%!
  ```

## 条件语句
- **`if`** 用于执行条件检查，可以用于字符串比较、数值比较和文件存在性检查。

  ```batch
  if "%myVar%"=="Hello" echo It is Hello!
  if exist "file.txt" echo File exists!
  ```

  - `/i`：忽略大小写。

- 比较操作符：

  - 字符串比较：`==`
  - 数值比较：`equ（equal，等于）`、`neq（not equal，不等于）`、`lss（less than，小于）`、`leq（less than or equal to，小于等于）`、`gtr（greater than，大于）`、`geq（greater than or equal to)，大于等于`

  ```batch
  set /a num1=5
  set /a num2=10
  if %num1% leq %num2% echo num1 is less than or equal to num2
  ```

> [!TIP]
>
> 当一个判断（循环或其他）语句之后的操作有多行时，可以用圆括号括起来表示一个作用域

##  循环

- **`for`** 用于创建循环，用于遍历文件、字符串、命令输出等。`%%`表示的是脚本内部的变量

  ```batch
  for %%i in (1 2 3) do echo Loop number %%i
  ```

- **`/L`** 参数用于创建计数循环。从`start`开始，一次增加`step`，到`end`结束

  ```batch
  for /L %%variable in (start,step,end) do command
  for /L %%i in (1,1,5) do echo %%i
  ```

- **遍历目录中的文件名：**

  ```batch
  ::打印文件名
  for %%f in (*.txt) do echo Found file: %%f
  ::打印文件完整路径
  for %%f in (*.txt) do echo Found file: %%~f
  ```

  ```
  %%~ 修饰符
  在 for 循环中，%%~ 后面的字符可以用来获取文件的不同信息：
  
  假设现在循环的内部变量是f
  %%~f: 获取文件的完整路径。
  %%~dpf: 获取文件的驱动器和路径。
  %%~nxf: 获取文件的名称和扩展名。
  %%~dpnxf: 获取文件的完整路径、名称和扩展名。
  ```

- **遍历命令输出：**

  ```batch
  ::
  for /f ["options"] %%variable in (file) do command
  for /f ["options"] %%variable in ('command') do command
  for /f "tokens=*" %%i in ('dir /b *.txt') do echo %%i
  ```

## 跳转与标签
- **`goto`** 和 **`:`** 用于跳转到脚本中的指定位置。

  ```batch
  goto myLabel
  echo This won't be displayed.
  :myLabel
  echo Jumped to myLabel
  ```

## 调用其他脚本
- **`call`** 用于从一个批处理文件中调用另一个批处理文件或子例程。

  ```batch
  call otherScript.bat
  ```

## 暂停执行
- **`pause`** 用于暂停脚本执行，并提示用户按任意键继续。

  ```batch
  pause
  ```

## 退出脚本
- **`exit`** 用于退出批处理文件，并返回到命令提示符。

  ```batch
  exit
  ```

# 常用命令

## 文件操作

1. **`type`**：创建文件。

   ```bat
   type non > file.txt
   ```

2. **`copy`**: 复制文件。

   ```batch
   copy source_file destination_path
   ```

   - `/v`：验证文件是否正确写入。

   - `/y`：覆盖现有目标文件时不提示。

   - `/a`：将文件标记为 ASCII 文本模式。

   - `/b`：将文件标记为二进制模式。

3. **`del`**: 删除文件。

   ```batch
   del file_path
   ```

   - `/f`：强制删除只读文件。

   - `/q`：安静模式，删除时不提示确认。

   - `/s`：删除指定目录及所有子目录中的文件。

4. **`move`**: 移动（重命）文件。

   ```batch
   move source_file destination_path
   ```

5. **`xcopy`**: 复制文件和目录，包括子目录。
   ```batch
   xcopy source_path destination_path /e /i
   ```
   - `/i`: 如果目标不存在，假设目标是一个目录。
   - `/s`：复制所有非空子目录。
   - `/e`：复制所有子目录，包括空的子目录。
   - `/v`：验证文件是否正确写入目标位置。
   - `/y`：覆盖现有目标文件时不提示。

## 目录操作

1. **`dir`**:显示目录内容

   ```bat
   dir
   ```

   - **`/p`**：分页显示结果。
   - **`/w`**：以宽列表格式显示结果。
   - **`/s`**：递归显示目录及其所有子目录中的文件。
   - **`/b`**：使用简洁格式显示结果，仅显示文件和目录名。
   - **`/o`**：按指定顺序排序显示文件和目录。
      - `N`：按名称排序。
      - `S`：按大小排序。
      - `E`：按扩展名排序。
      - `D`：按日期/时间排序。
      - `G`：首先列出目录，然后是文件。
      - `-`：逆序排序。
   - **`/q`**：显示文件的所有者。
   - **`/a`**：显示具有特定属性的文件。
      - `R`：只读文件。
      - `H`：隐藏文件。
      - `S`：系统文件。
      - `D`：目录。
      - `A`：准备归档的文件。
      - `L`：符号链接。
   - **`/c`**：显示文件大小时包含千位分隔符（默认行为）。
   - **`/n`**：以默认格式显示文件名（与 `/o` 结合使用）。
   - **`/x`**：显示 8.3 短文件名。
   - **`/t`**：指定用于排序或显示的时间字段。
     - `C`：创建时间。
     - `A`：上次访问时间。
     - `W`：上次写入时间。

2. **`mkdir`**: 创建目录。

   ```batch
   mkdir directory_name
   ```

3. **`rmdir`**: 删除目录。
   ```batch
   rmdir directory_name /S /Q
   ```
   - `/S`: 删除目录及其所有子目录和文件。
   - `/Q`: 安静模式，不提示确认。

4. **`cd`**: 更改当前目录。

   ```batch
   cd directory_path
   ```
   - `cd ..` 返回上一级目录。

5. 获取文件路径

   - `%~f1`：获取文件的完整路径。

   - `%~d1`：获取文件的驱动器号。

   - `%~p1`：获取文件的路径（不包括文件名）。

   - `%~n1`：获取文件名（不包括扩展名）。

   - `%~x1`：获取文件扩展名。

     ```bat
     set filepath=%~f1
     echo File path: %filepath%
     ```

     

## 流程控制

1. **`if`**: 条件判断。
   ```batch
   if condition (command1) else (command2)
   ```
   - Example:
     ```batch
     if exist file.txt (
         echo File exists.
     ) else (
         echo File does not exist.
     )
     ```

2. **`for`**: 循环执行命令。
   ```batch
   for %%variable in (set) do command
   ```
   - Example:
     ```batch
     for %%f in (*.txt) do (
         echo %%f
     )
     ```

3. **`goto`**: 跳转到指定标签。
   ```batch
   goto label_name
   ```
   - Example:
     ```batch
     goto end
     
     :end
     echo Script finished.
     ```

4. **`call`**: 调用另一个批处理文件或标签。
   ```batch
   call script.bat
   ```

5. **`exit`**: 退出批处理脚本。
   ```batch
   exit
   ```
   - `exit /b`: 退出子批处理文件并返回父脚本。

## 输入/输出

1. **`echo`**: 输出文本。
   ```batch
   echo Hello, World!
   ```
   - `echo.` 输出空行。

2. **`pause`**: 暂停执行并等待用户按键。
   ```batch
   pause
   ```

3. **`set /p`**: 获取用户输入并赋值给变量。
   ```batch
   set /p variable_name=Enter value: 
   ```

##  字符串处理

- **获取子字符串：** `:~` 用于从变量中提取子字符串。

  ```batch
  set string=HelloWorld
  echo %string:~0,5%  REM 输出 "Hello"
  ```

- **字符串替换：** `:<old>=<new>` 用于替换字符串中的子串。

  ```batch
  set string=HelloWorld
  echo %string:World=Universe%  REM 输出 "HelloUniverse"
  ```

## 处理错误

- **检查错误代码：** 通过 `if errorlevel` 检查上一个命令的退出代码。

  ```batch
  some_command
  if errorlevel 1 echo Error occurred
  ```

- **设置退出代码：** 使用 `exit /b` 设置批处理脚本的退出代码。

  ```batch
  exit /b 0
  ```

# 特殊符号

- **`%`**：用于引用变量（如 `%variable%`）。

- **`&`**：用于将多条命令放在同一行上执行。

  ```batch
  echo First command & echo Second command
  ```

- **`|`**：管道符号，用于将前一命令的输出作为下一命令的输入。

  ```batch
  dir /b | find "keyword"
  ```

- **`>`** 和 **`>>`**：重定向符号，用于将输出写入文件（覆盖或追加）。

  ```batch
  echo Hello > output.txt ::覆盖
  echo World >> output.txt ::追加
  ```

- **`<`**：输入重定向，用于将文件内容作为命令的输入。

  ```batch
  findstr "keyword" < input.txt
  ```

- **`^`**：转义符号，用于转义特殊字符。

  ```batch
  echo This is a caret ^ symbol
  ```

- **`&&`** 和 **`||`**：条件执行符号，分别表示“如果成功则执行”和“如果失败则执行”。

  ```batch
  some_command && echo Success || echo Failure
  ```

# 综合示例

以下是一个更复杂的 Batch 脚本示例，它演示了各种操作：

```batch
@echo off
setlocal

REM 获取当前日期和时间
set currentDate=%date%
set currentTime=%time:~0,8%

echo Starting backup process at %currentDate% %currentTime%

REM 创建备份目录
set backupDir=C:\Backup\%date:~-4,4%-%date:~-10,2%-%date:~-7,2%
if not exist "%backupDir%" mkdir "%backupDir%"

REM 复制重要文件
xcopy C:\ImportantFiles\* "%backupDir%" /s /e /y
if errorlevel 1 (
    echo Failed to copy files. Exiting.
    exit /b 1
)

REM 删除旧备份（超过7天）
forfiles /p "C:\Backup" /d -7 /c "cmd /c rmdir /s /q @path"

REM 结束备份
echo Backup completed successfully.
pause
```

