# 参考链接

[从零到一搭建Kconfig配置系统-CSDN博客](https://blog.csdn.net/wenbo13579/article/details/127464764)

# 安装

由于在window上使用Kconfig需要依赖Curses，所以要先安装window-curses，但是curses本身是应用在Linux平台上的，所以想要安装在window上，就需要使用whl包进行安装。

> curses的下载地址：[windows-curses · PyPI](https://pypi.org/project/windows-curses/#files)

在安装的时候需要检查一下安装的python版本，然后根据python版本安装相应的whl文件

```bash
python --version
```

