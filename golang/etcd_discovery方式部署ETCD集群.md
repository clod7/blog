# etcd_discovery方式部署ETCD集群

## 使用docker来模拟多台服务器
```
docker安装后，默认会创建三种网络类型，bridge、host和none
可以使用docker network ls 查看
```
### 创建固定ip容器
```
docker network create --subnet=172.18.0.0/16 staticnet
通过docker network ls可以查看到网络类型中多了一个staticnet

使用新的网络类型创建并启动容器
docker run -it --name userserver --net staticnet --ip 172.18.0.2
// 不指定ip将顺序的分配ip给容器
```

## etcd discovery方式选择
etcd discovery集群方式分为两种：自定义的etcd discovery和公共 etcd discovery。
没有已创建好的集群建议使用公共etcd discovery。
公共的discovery就是通过CoreOS提供的公共discovery服务申请token。
```
curl https://discovery.etcd.io/new?size=3  // 获取token
如果实际启动的etcd节点个数大于discovery token创建时指定的size，多余的节点会自动变为proxy节点。
```

## 自定义docker镜像
```
1. docker pull ubuntu  // 拉去ubuntu镜像
2. docker run -it --name myubuntu ubuntu
3. apt-get update
4. apt-get install -y ca-certificates  //解决docker中不能使用https情况
5. docker cp ./etcd myubuntu:/home   // 将本地的etcd文件拷贝到docker 
6. docker commit ubuntu-1 myetcd:1.0  // 生成自定义镜像
```

## 容器中运行etcd
```
docker run -it --name etcd00 --net staticnet etcd:1.0
cd /home/etcd
docker inspect etcd00 查看ip
运行etcd00(etcd01,etcd02同理)
./etcdser --name etcd00 \
--data-dir /home  \
--listen-peer-urls http://172.18.0.2:2380 \
--listen-client-urls http://172.18.0.4:2379,http://172.18.0.2:4001 \
--initial-advertise-peer-urls http://172.18.0.2:2380 \
--advertise-client-urls=http://172.18.0.2:2379,http://172.18.0.2:4001 \
--discovery https://discovery.etcd.io/f329f829a446004d144888e7618d34d1 \

使用./etcdctl member list查看集群成员
```


