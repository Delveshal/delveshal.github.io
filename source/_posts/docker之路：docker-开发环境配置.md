---
title: docker之路：docker 开发环境配置
date: 2018-06-04 15:12:10
tags: docker
categories: docker
---
# docker之路：docker 开发环境配置

可以直接通过[官方教程](https://github.com/moby/moby/blob/master/docs/contributing/set-up-dev-env.md)进行搭建，但是如果没有翻墙是很难搭建成功的。
这里我采用另外一种办法，使用[dockercore/docker](https://hub.docker.com/r/dockercore/docker/)，这个镜像包含了所需要用到的依赖。镜像大概2G。

# git clone
路径最好是`$GOPATH/src/github.com/docker`，`$GOPATH`可以是项目独有的。
``` bash
git clone git@github.com:docker/docker.git
# 由于dockercore/docker的版本的17.03.2-ce(目前最新release版本)，所有我们要checkout一下
git checkout 17.03.2-ce
```

# docker pull
```
docker pull dockercore/docker
docker run --rm -i --privileged -e BUILDFLAGS -e KEEPBUNDLE -e DOCKER_BUILD_GOGC -e DOCKER_BUILD_PKGS -e DOCKER_CLIENTONLY -e DOCKER_DEBUG -e DOCKER_EXPERIMENTAL -e DOCKER_GITCOMMIT -e DOCKER_GRAPHDRIVER=devicemapper -e DOCKER_INCREMENTAL_BINARY -e DOCKER_REMAP_ROOT -e DOCKER_STORAGE_OPTS -e DOCKER_USERLANDPROXY -e TESTDIRS -e TESTFLAGS -e TIMEOUT -v "/home/delveshal/GoglandProjects/src/github.com/docker/docker:/go/src/github.com/docker/docker" -t "dockercore/docker:latest" bash
# 编译二进制文件，生成目录是bundles
./hack/make.sh binary
```
现在就可以编辑源码然后使用上面的 make.sh binary 就可以进行编译
