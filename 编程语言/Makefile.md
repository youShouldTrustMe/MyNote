# 参考链接

[跟我一起写Makefile — 跟我一起写Makefile 1.0 文档 (seisman.github.io)](https://seisman.github.io/how-to-write-makefile/index.html)

# 书写规则

```makefile
targets: prerequisites 
    commands
targets:prerequisties;commands
```

*`targets`*

- 可以是一个object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。
- targets是文件名，以空格分开，可以使用通配符。一般来说，我们的目标基本上是一个文件，但也有可能是多个文件。

*`prerequisites`*

- 目标所依赖的文件（或依赖目标）。如果其中的某个文件要比目标文件要新，那么，目标就被认为是“过时的”，被认为是需要重生成的。

*`commands`*

- 该target要执行的命令（任意的shell命令）。
- commands是命令行，如果其不与“target:prerequisites”在一行，那么，必须以 `Tab` 键开头。
- 如果和prerequisites在一行，那么可以用分号做为分隔。（见上）
- 如果命令太长，可以使用反斜杠（ `\` ）作为换行符。make对一行上有多少个字符没有限制。

>  [!NOTE]
>
> 这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:
>
> ==prerequisites中如果有一个以上的文件比target文件要新的话，commands所定义的命令就会被执行。==

>  [!IMPORTANT]
>
> 注意：makefile有着严格的缩进格式，命令前面需要使用tab键对齐

## 通配符

如果想定义一系列比较类似的文件，则可使用通配符。make支持三个通配符：

1. `*`: 匹配任意数量的字符（包括零个字符），但不匹配路径分隔符 `/`。
2. `?`: 匹配单个任意字符，但不匹配路径分隔符 `/`。
3. `[...]`: 匹配括号内的任意单个字符。例如，`[abc]` 匹配 `a`、`b` 或 `c`。

```
clean:
    rm -f *.o
```

其实在这个clean:后面可以加上你想做的一些事情，如果你想看到在编译完后看看main.c的源代码，你可以在加上cat这个命令，例子如下：

```
clean:
    cat main.c
    rm -f *.o
```

其结果你试一下就知道的。 上面这个例子我不不多说了，这是操作系统Shell所支持的通配符。这是在命令中的通配符。

```
print: *.c
    lpr -p $?
    touch print
```

上面这个例子说明了通配符也可以在我们的规则中，目标print依赖于所有的 `.c` 文件。其中的 `$?` 是一个自动化变量，我会在后面给你讲述。

```
objects = *.o
```

上面这个例子，表示了通配符同样可以用在变量中。并不是说 `*.o` 会展开，不！objects的值就是 `*.o` 。Makefile中的变量其实就是C/C++中的宏。如果你要让通配符在变量中展开，也就是让objects的值是所有 `.o` 的文件名的集合，那么，你可以这样：

```
objects := $(wildcard *.o)
```

另给一个变量使用通配符的例子：

1. 列出一确定文件夹中的所有 `.c` 文件。

   ```
   objects := $(wildcard *.c)
   ```

2. 列出(1)中所有文件对应的 `.o` 文件，在（3）中我们可以看到它是由make自动编译出的:

   ```
   $(patsubst %.c,%.o,$(wildcard *.c))
   ```

3. 由(1)(2)两步，可写出编译并链接所有 `.c` 和 `.o` 文件

   ```
   objects := $(patsubst %.c,%.o,$(wildcard *.c))
   foo : $(objects)
       cc -o foo $(objects)
   ```

这种用法由关键字“wildcard”，“patsubst”指出，关于Makefile的关键字，我们将在后面讨论。



# 书写命令

# 使用变量



























































