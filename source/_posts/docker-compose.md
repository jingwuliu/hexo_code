---
title: docker-compose.yml的基础使用
date: 2021-02-07 17:20:58
tags: [Docker, Docker-compose]
---

docker-compose.yml文件用来做项目编排，负责实现对Docker容器集群的快速编排，部署分布式应用，通过一个单独的yaml文件为一个项目来定义一组相关联的应用容器。文件中主要包含了使用各种镜像创建的容器服务，使用的镜像可以是网络上的，也可以是根据Dockerfile文件生成的。

一份标准docker-compose.yml配置文件应包含**version**、**service**、**networks**三部分，其中最关键的就是*service*和*networks*两部分。

* services标签下的第二级标签的名字由用户自定义，其就为服务名称。
```
services:
    vue-client:
        ...
```

services下的服务里仍然有需要方法可以执行，下面我们将选择部分进行介绍。

## image
    image可以直接指定服务的镜像名称或ID。如果镜像在本地不存在，compose将会尝试拉取这个镜像。格式如下： 
    ```
    services: 
        vue-client: 
            image: hello-world 
            ///image: ubuntu:16.04 
            ///image: a4bc65fd   
    ```
## build
    服务也可以使用build基于一份Dockerfile，在使用docker-compose up启动时执行构建任务。Compose会利用提供的路径自动构建该镜像，然后使用这个镜像启动服务容器。如果你同时指定了image和build两个标签，那么Compose会构建镜像并且把镜像命名为image提供的名称。
```
    services: 
        vue-client:
            build: ./build/client
```
## command
    使用command可以覆盖容器启动后默认执行的命令。
    ```
    services:
        vue-client:
            command: ["--replSet", "rs0", "--bind_ip_all"]
            ///command: --replSet rs0 --bind_ip_all
    ```
## container_name
    该指令用于指定生成的容器的标签名。
    ```
    services:
        vue-client:
            container_name: vue-client
    ```
## env_file
    在docker-compose.yml中可以定义一个专门存放变量的文件。也可以通过docker-compose -f FILE的方式指定docker-compose.yml配置文件，相应的配置文件中的env_file也会使用配置文件路径。
    - docker-compose.yml
    ```
    services:
        vue-client:
            env_file: .env
    ```
    - .env
    ```
    MONGODB_DB_NAME=release-test
    MEDA_DATA_PATH=/meda_data/release_test/images/
    CLIENT_PORT=3001:80
    ```
## environment
    添加环境变量，可以使用数组或字典。作用就是为镜像设置变量，将变量保存到镜像中，由此镜像生成的容器同样也会包含这些变量设置。
    ```
    services:
        node-server:
            environment:
                MONGODB_URL: "mongodb://mongo1:27017"
                MONGODB_DB_NAME: "release-test-1"
    ```
## healthcheck
    用于检查测试服务使用的容器是否正常。
    ```
    services:
        mongo:
           healthcheck:
                test: ["CMD","curl","-f","http://localhost"]
                interval: 15s
                timeout: 10s
                retries: 3
                start_period: 10s
    ```
    其中interval、timeout、start_period都定位持续时间。test中的内容必须是字符串或列表。
## ports
    映射端口。可以使用HOST:CONTAINER的方式指定端口，可以指定容器和主机端口。
    ```
    services:
        file-server:
            ports:
                - "9950:80"
                - "8090:8090"
    ```
## volumes
    挂载一个目录或已存在的数据卷容器，直接使用HOST:CONTAINER的格式。
    ```
    - services:
        engine-center:
            volumes:
                - "/meda_data/release_test/images:/app/meda_output/"

        mongo:
            volumes:
                - "dbdata1:/data/db"

    - volumes:
        dbdata1:
    ```

# docker compose常用命令

## dockers-compose (-f) docker-compose.yml up -d  
    在后台启动服务。

## docker-compose ps
    查看启动的服务。

## docker-compose stop
    停止目前处于运行状态的容器，但不删除。

## docker-compose down
    停用移除所有容器以及相关网络。

## docker-compose start
    启动已经存在的服务容器。


