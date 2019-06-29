# grpc安装

##安装pkg-config
```
sudo apt-get install pkg-config
```

##安装依赖文件
```
sudo apt-get install autoconf automake libtool make g++ unzip
sudo apt-get install libgflags-dev libgtest-dev
sudo apt-get install clang libc++-dev
```

## 下载grpc源码

```
git clone https://github.com/grpc/grpc.git
cd grpc
git submodule update --init  更新第三方源码
```

##此时应保证在grpc文件夹下
```
cd third_party/protobuf/
git submodule update --init --recursive 确保克隆子模块，更新第三方源码
sudo ./autogen.sh   生成配置脚本
sudo ./configure    生成Makefile文件，为下一步的编译做准备，可以加上安装路径：--prefix=path ，默认路径为/usr/local/
sudo make           从Makefile读取指令，然后编译
sudo make check     可能会报错，但是不影响,对于安装流程没有实质性用处，可以跳过该步
sudo make install 
sudo ldconfig       更新共享库缓存
which protoc        查看软件的安装位置
protoc --version    检查是否安装成功

###一下为对make命令的补充，不运行：
# 卸载命令：make uninstall
# 清除编译产生的可执行文件及目标文件：make clean
# 除了清除可执行文件和目标文件外，把configure所产生的Makefile也清除掉:make distclean
```

## 安装grpc
```
cd ../..  #到达grpc根文件夹下
make   #编译
```

## 错误情况
至此可能出现错误： 
/usr/bin/ld: warning: libprotobuf.so.15, needed by //usr/local/lib/libprotoc.so, may conflict with libprotobuf.so.9 
这是因为电脑安装了两个不同版本的protobuf(ubuntu16默认已经安装了protobuf.so.9这系列的，新装的是protobuf.so.15系列的）。 
解决办法： 
卸载掉老版本的protobu：
```
sudo apt-get remove libprotobuf-dev
```

## 继续安装
```
sudo make install  #编译安装，默认安装位置为/usr/local/
```

## 测试
```
cd examples/cpp/helloworld/
sudo make   #如果此处出错，可能就是安装有问题了
sudo ./greeter_server   #运行server，监听50051端口
#打开一个新的终端运行client
sudo ./greeter_client
#就可以看到返回结果：Greeter received: Hello world
```

