# 使用docker部署项目

```
镜像为goalng:latest  
将项目go build 后再发布到docker中，可以避免外部库导入失败的情况。
使用数据卷来连接mysql,redis
将镜像发布到阿里云docker管理中

等下使用镜像alpine:latest部署项目
使用alpine时，会有很多c的库不能使用，需要go build -v -installsuffix cgo -o
可能还需要导入其他的包或库...
```


## 使用mysql镜像
```
拉取镜像  docker pull mysql
查看镜像  docker images|grep mysql
启动镜像  
docker run -p 33060(本地端口):3306(容器端口) -e MYSQL_ROOT_PASSWORD=root --name mysql 
-it -v /tmp/docker-mysql(本地目录):/var/lib/mysql(容器目录) mysql
查看容器进程 docker ps -a
进入mysql容器 docker exec -it dockerID /bin/bash
停止启动重启容器  docker start|stop|restart dockerID
删除容器 docker rm -f dockerID    -f 强制删除
删除镜像 docker rmi dockerID
```

## 单机容器间的关联
```
docker run --link mysql(容器名):mysql -p 3001:3001 --name testname test
```

## 远程连接mysql容器
```
进入容器: docker exec -it mysql /bin/bash
连接mysql: mysql -u root -p
查看信息: select host,user,plugin,authentication_string from mysql.user; 
备注：host为 % 表示不限制ip localhost表示本机使用 plugin非mysql_native_password 则需要修改密码
修改加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges; 
连接mysql即可
```