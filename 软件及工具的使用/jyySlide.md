# 下载、安装

## 参考链接

[sunnysky29/jyy-slides: jyy OS PPT (github.com)](https://github.com/sunnysky29/jyy-slides)

## 工具的安装

首先需要安装python，python版本需要在`3.10`及以上

安装poetry

```shell
pip install poetry
```

克隆库

```bash
git clone https://github.com/zweix123/jyyslide-md.git
cd jyyslide-md
```

安装第三方库

```bash
poetry install
```

## 转换

转换可以使用两种方法：

1. > 使用虚拟环境
   >
   > ```bash
   > poetry shell
   > python main.py 需要转换的文件地址
   > ```

2. > 直接转换
   >
   > ```
   > poetry run python main.py 需要转换的文件地址
   > ```

windows的快捷转换：

1. 首先需要先给md文件设置一个右键命令

   1. > 按下 `Win + R` 打开“运行”窗口，输入 `regedit`，然后按回车进入注册表编辑器。
      >
      > 导航到以下路径：
      >
      > ```
      > 复制代码
      > HKEY_CLASSES_ROOT\*\shell
      > ```
      >
      > 右键点击 `shell` 文件夹，选择“新建” > “项”，将其命名为 `Convert to HTML`。
      >
      > 在 `Convert to HTML` 项下，右键点击右侧窗格中的 `(默认)`，选择“修改”，将“数值数据”设置为你希望显示的右键菜单项的名称，比如“Convert Markdown to HTML”。
      >
      > 右键点击 `Convert to HTML` 项，选择“新建” > “项”，将其命名为 `command`。
      >
      > 选择 `command` 项，然后右键点击右侧窗格中的 `(默认)`，选择“修改”。在“数值数据”中输入以下内容：
      >
      > ```
      > "D:\jyyPPT\md2html.bat" "%1"
      > ```
      >
      > 请确保路径替换为你实际的批处理文件路径。

   2. 这个时候右键的时候就能发现已经创建成功了

2. 设置一个bat文件

   1. > ```batch
      > @echo off
      > cd /d "D:\jyyPPT\jyyslide-md\"
      > poetry run python main.py "%1"
      > pause
      > ```

# 语法

水平分割使用`---`

垂直分割使用`----`

渐变垂直使用`++++`

- 在同一张幻灯片中依次出现的页面使用`--`

作者信息页使用`+++++`，语法格式使用json或yaml

- > ```json
  > {
  >     "author": {
  >         "name": "蒋炎岩",
  >         "url": "https://ics.nju.edu.cn/~jyy/"
  >     },
  >     "departments": [
  >         {
  >             "name": "  南京大学  ",
  >             "url": "https://www.nju.edu.cn/main.htm",
  >             "img": "./img/nju-logo.jpg"
  >         },
  >         {
  >             "name": "计算机科学与技术系",
  >             "url": "https://cs.nju.edu.cn/main.htm",
  >             "img": "./img/njucs-logo.jpg"
  >         },
  >         {
  >             "name": "计算机软件研究所",
  >             "url": "https://www.nju.edu.cn/main.htm",
  >             "img": "./img/ics-logo.png"
  >         }
  >     ]
  > }
  > ```

- > ```yaml
  > author:
  >   name: 蒋炎岩
  >   url: https://ics.nju.edu.cn/~jyy/
  > 
  > departments:
  >   - name: "  南京大学  "
  >     url: https://www.nju.edu.cn/main.htm,
  >     img: ./img/nju-logo.jpg
  > 
  >   - name: 计算机科学与技术系
  >     url: https://cs.nju.edu.cn/main.htm,
  >     img: ./img/njucs-logo.jpg
  > 
  >   - name: 计算机软件研究所
  >     url: https://www.nju.edu.cn/main.htm,
  >     img: ./img/ics-logo.png
  > ```
  >
  > 

















