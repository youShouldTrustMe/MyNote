# 参考链接

[CSCI 2021 Quick Guide to gdb: The GNU Debugger (umn.edu)](https://www-users.cse.umn.edu/~kauffman/tutorials/gdb)

[GDB Documentation (sourceware.org)](https://sourceware.org/gdb/documentation/)

# 快速启动

使用`-g`命令生成debug的相关特征符号

```bash
gcc -Wall -g -c resource.c #这个时候会产生一个a.out文件
#或者使用
gcc -Wall -g -o target resource.c
```

使用TUI模式对代码进行调试

```bash
gdb -tui ./target
```

> [!NOTE]
>
> 推荐使用TUI模式，TUI模式即能看到源码，又能使用命令行