## 开机启动Docker：
    systemctl enable docker

## Docker配置文件
    vim /etc/docker/daemon.json
    {
    //配置仓库镜像地址
    "registry-mirrors": ["https://kzflb.mirror.aliyuncs.com"],
    //默认http私有仓库不能访问，设置后才可以
    "insecure-registries": ["http://192.168.2.196"],
    //开启docker-API远程访问
    "hosts": ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"]
    }

## 更新配置文件后
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    
## 我们运行docker容器的时候，使用了-d参数，把容器在后台运行后。
    1.启动docker容器
    docker run -d -i -t <imageID> /bin/bash 
    上述命令执行某些命令回报错Failed to get D-Bus connection: Operation not permitted
    可使用如下命令启动
    docker run -d -i -t <imageID> /usr/sbin/init
    这个时候，我们使用docker ps命令，我们就可以知道哪些程序在后台运行。

    2.我们要怎么进入到docker容器呢？
    （1）使用docker attach命令
    我们使用
    docker attach db3 或者 docker attach d48b21a7e439 
    db3是后台容器的NAMES,d48b21a7e439是容器的进程ID  CONTAINER ID
    然后就进去了这个容器的ssh界面。
    但是它有一个缺点，只要这个连接终止，或者使用了exit命令，容器就会退出后台运行

    （2）使用docker exec命令
    这个命令使用exit命令后，不会退出后台，一般使用这个命令，使用方法如下
    docker exec -it db3 /bin/sh 或者 docker exec -it d48b21a7e439 /bin/sh
    db3是后台容器的NAMES,d48b21a7e439是容器的进程ID  CONTAINER ID
    /bin/sh 是固定写法
    exit  #直接让容器停止并退出
    Ctrl + P + Q #容器不停止退出

## 删除镜像
    1.停止所有的container，这样才能够删除其中的images：
    docker stop $(docker ps -a -q)
    如果想要删除所有container的话再加一个指令：
    docker rm $(docker ps -a -q)
    2.查看当前有些什么images
    docker images
    3.删除images，通过image的id来指定删除谁
    docker rmi <image id>
    删除untagged images，也就是那些id为<None>的image
    docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
    删除全部image
    docker rmi $(docker images -q)

## 查看 Docker 容器历史运行日志
    $ docker logs {容器名}
    实时监听 Docker 容器运行日志
    $ docker logs -f {容器名}
    Docker 数据卷命令
    创建 Docker 数据卷
    $ docker volume create {数据卷名}
    列出所有 Docker 数据卷
    $ docker volume ls
    删除指定 Docker 数据卷
    $ docker volume rm {数据卷名}
    删除未关联（失效） Docker 数据卷
    $ docker volume prune
    $ docker volume rm $(docker volume ls -qf dangling=true)
    Docker 文件操作命令
    从主机复制文件到 Docker 容器中
    $ sudo docker cp {主机内文件路径} {容器ID}:{容器内文件存储路径}
    从 Docker 容器中复制文件到主机中
    $ sudo docker cp {容器ID}:{容器内文件路径} {主机内文件存储路径}