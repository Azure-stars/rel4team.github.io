---
title: "Docker 环境安装"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 1. 简介

本文主要叙述如何使用 rel4-dev docker 镜像进行 sel4 和 rel4 的尝试。该镜像可以用于运行 sel4 和 rel4 例程，同时还支持 rust-sel4 使用 reL4 和 seL4 kernel。

使用 Docker 确保所有的 reL4 kernel 和 app 开发者使用相同的开发环境，避免因为环境问题造成无法同步。

## 2. 启动 Docker

### 2.1 进入 Docker 环境

> Start Docker 后会做一些配置，启动时间较长，因此尽量不要频繁的 Start Docker

```
# start_docker, only need execute once
# LOCAL WORKSPACE is the dir which map into docker container. Usually, it's you workspace, will map to /workspace in docker container docker. Please replace the {LOCAL WORKSPACE} with real dir path 
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/rel4team/build-scripts/refs/heads/mi_dev/scripts/start_docker.sh | bash -s -- {LOCAL WORKSPACE}

# enter docker with your current user
docker exec -u ${USER} -it rel4_dev bash

# You should be in the rel4_dev_env now
croak@rel4_dev_env:/workspace
```

### 2.2 Docker 启动脚本

如果你的网络不好，无法访问 raw.github, 可以将脚本 clone 下来再运行

```
git clone https://github.com/rel4team/build-scripts.git
cd build-scripts/scripts
bash start_docker.sh {LOCAL WORKSPACE}

# enter docker with your current user
docker exec -u ${USER} -it rel4_dev bash

# You should be in the rel4_dev_env now
croak@rel4_dev_env:/workspace
```

### 2.3 使用 Proxy

如果你需要使用 Proxy，请自行在 start_docker.sh 脚本 docker run 命令中加入 proxy 参数

```
    # -e HTTP_PROXY=http://127.0.0.1:7890 \
    # -e HTTPS_PROXY=http://127.0.0.1:7890 \
    # You can add proxy setting in docker run if you encounter network problem

    docker run -itd \
        --name "${CONTAINER_NAME}" \
        -e DOCKER_USER="${user}" \
        -e DOCKER_USER_ID="${uid}" \
        -e DOCKER_GRP="${group}" \
        -e DOCKER_GRP_ID="${gid}" \
        -v $workspace:/workspace \
        -e HTTP_PROXY=http://127.0.0.1:7890 \
        -e HTTPS_PROXY=http://127.0.0.1:7890 \
        -w /workspace \
        --hostname rel4_dev_env \
        --network host \
        ${IMAGE_NAME}:${IMAGE_VERSION} \
        /bin/bash

```

其中 http://127.0.0.1:7890 换成你的 proxy server 地址

## 3 使用编译环境

> 推荐将你所有 reL4 和 seL4 相关的项目放在一个文件夹下，然后将该文件夹映射到 docker container 中

### 3.1 rel4 kernel

在 Docker 环境中编译运行 rel4test 如下

```
# clone mi-dev kernel，you can also run follow command outside docker
mkdir mi-dev-repo && cd mi-dev-repo
repo init -u https://github.com/rel4team/mi-dev-repo.git -b main
repo sync

cd rel4_kernel
./build.py -p spike

cd build
./simulate

```

### 3.2 seL4 kernel

seL4 同样可以编译执行，按照官方指南中执行即可

### 3.3 rust-seL4

环境中可以运行 reL4team 修改后的 rust-seL4，使用 reL4 kernel

> 值得注意的是，目前 rust-seL4 使用的不是 reL4 最新的 kernel

由于我们已经在 Docker 中，无需进入 rust-seL4 的 Docker 环境，直接执行 make run 即可

```
git clone https://github.com/Huzhiwen1208/rust-root-task-demo.git
cd rust-root-task-demo
make run
```