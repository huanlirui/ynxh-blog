---
cover: /img/linux.jpeg
---

# m1 芯片 mac 使用 docker 构建跨平台镜像

问题：

新换了 m1 的电脑，发现原先的构建脚本无法使用了
报错大概如下：
docker: Error response from daemon: image with reference registry.cn-chengdu.aliyuncs.com/gtkj/cce-web-im:test was found but does not match the specified platform: wanted linux/amd64, actual: linux/arm64.

好吧，看来就是构建的镜像 架构不同，不支持

## 1. Docker 默认的 builder 不支持同时指定多个架构，所以要新建一个

```bash
docker buildx create --use --name m1_builder
```

## 查看并启动 builder 实例

```
docker buildx inspect --bootstrap
```

```
Name:   m1_builder
Driver: docker-container

Nodes:
Name:      m1_builder0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Buildkit:  v0.11.0
Platforms: linux/arm64, linux/amd64, linux/amd64/v2, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6
```

其中 platforms 就是支持的架构，跨平台构建的底层是用 QEMU 实现的。

## 3.构建多架构 Docker 镜像

使用 buildx 构建：

```
docker buildx build \
  --platform linux/amd64,linux/arm64
  --push -t prinsss/google-analytics-hit-counter .
```

其中 -t 参数指定远程仓库，--push 表示将构建好的镜像推送到 Docker 仓库。如果不想直接推送，也可以改成 --load，即将构建结果加载到镜像列表中。

--platform 参数就是要构建的目标平台，这里我就选了本机的 arm64 和服务器用的 amd64。最后的 .（构建路径）注意不要忘了加。

我的命令

```
docker buildx build --platform=linux/amd64 -f dockerfile.test --load -t ${ImageName}:${ImageTag} .

```

> --load 这个参数必须加上。否则创建的镜像不会被直接保存下来

最后 docker run 的时候也加上参数 --platform linux/amd64
我的命令如下：

```
docker run --platform linux/amd64 -d -p ${ContainerPort}:${ContainerPort} --name ${ContainerName} ${ImageName}:${ImageTag}
```
