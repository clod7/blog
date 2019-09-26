# 多步构建python最小镜像

## 什么是多步构建
Docker 17.05版本以后，官方就提供了一个新的特性：Multi-stage builds（多阶段构建）。 使用多阶段构建，你可以在一个 Dockerfile 中使用多个 FROM 语句。每个 FROM 指令都可以使用不同的基础镜像，并表示开始一个新的构建阶段。你可以很方便的将一个阶段的文件复制到另外一个阶段，在最终的镜像中保留下你需要的内容即可。

## 为什么用python，不用golang
golang代码可以直接编译成二进制可执行文件，多步构建的优势不够明显。

## 多步构建的好处
- 镜像大小远小于之前
- 只需要一个Dockerfile定义整个构建过程
- 可以选用更适合的基础镜像

## Dockerfile
```
FROM python:3.5 AS build

RUN mkdir /install
WORKDIR /install
# 安装依赖
RUN python3 -m pip install --upgrade pip \
    && pip install lxml --install-option="--prefix=/install" -i  "https://pypi.tuna.tsinghua.edu.cn/simple/" 


FROM python:3.5-alpine
COPY --from=build /install /usr/local
COPY . /app
WORKDIR /app

ENTRYPOINT ["python3", "demo.py"]
```