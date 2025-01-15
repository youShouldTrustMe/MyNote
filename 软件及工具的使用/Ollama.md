# 大模型的本地部署
## ollama的安装

下载官网

[Ollama](https://ollama.com/)

![ollama官方页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_10_2_13_202408031002660.png)

## oepn WebUI

下载官网

[🏡 Home | Open WebUI](https://docs.openwebui.com/)

有两种方法安装openWebUI

1. 使用docker安装(*`推荐`*)
2. 使用python安装（pip指令，且python版本需要在3.11及以上）

### docker



下载官网

[Docker: Accelerated Container Application Development](https://www.docker.com/)

![image-20240803101517610](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_10_15_17_202408031015701.png)

安装完成之后，确保docker在运行状态，然后打开cmd输入以下命令

```shell
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

> [!WARNING]
>
> 注意，这里一定要确保docker在运行状态，否则会报错

安装完成之后，docker会出现以下页面，点击port下的网址即可访问open-ui了

![image-20240803105455441](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_10_54_55_202408031054562.png)

首次运行会需要注册账号（这里的注册账号是随便输入账号名称都可以使用，首次注册的账号是管理员账号）

![注册页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_10_57_26_202408031057350.png)

注册完成之后即可进入聊天页面

![聊天页面](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_11_1_50_202408031101274.png)

设置模型，在拉取模型的右下角有个链接，可以直接查看当前最受欢迎的语言模型，需要使用的话就输入模型名称即可

![image-20240803110226966](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/08/3_11_2_27_202408031102050.png)

# OpenRouter























