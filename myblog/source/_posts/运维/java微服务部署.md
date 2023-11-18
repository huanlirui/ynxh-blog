记录一个 shell 脚本的坑

比如如下脚本，执行的时候提示

```bash
./restart.sh: 行 1: $'\r': 未找到命令
```

出现这样的错误，是因为 Shell 脚本在 Windows 系统编写时，每行结尾是\r\n，而在 Linux 系统中行每行结尾是\n，所以在 Linux 系统中运行脚本时，会认为\r 是一个字符，导致运行错误。
虽然我是 mac 编写的，但应该也是这个意思吧。

解决方法
去除 Shell 脚本的\r 字符：

方法 1

```bash
sed -i 's/\r//' restart.sh
```

方法 2

```bash
dos2unix one-more.sh
```

如果权限不足：

```bash
chmod +x restart.sh
```

```bash

#!/bin/bash
# 停止容器
echo "Stopping container ruoyi-cloud-gen..."
docker stop ruoyi-cloud-gen
wait
if [ $? -eq 0 ]; then
  echo "Container stopped successfully."
else
  echo "Failed to stop container."
fi

# 删除容器
echo "Removing container ruoyi-cloud-gen..."
docker rm ruoyi-cloud-gen
wait
if [ $? -eq 0 ]; then
  echo "Container removed successfully."
else
  echo "Failed to remove container."
fi

# 删除镜像
echo "Removing image ruoyi-cloud-gen..."
docker rmi ruoyi-cloud-gen
wait
if [ $? -eq 0 ]; then
  echo "Image removed successfully."
else
  echo "Failed to remove image."
fi

# 构建镜像
echo "Building image ruoyi-cloud-gen..."
docker build -f dockerfile -t ruoyi-cloud-gen .
wait
if [ $? -eq 0 ]; then
  echo "Image built successfully."
else
  echo "Failed to build image."
fi

# 运行容器
echo "Running container ruoyi-cloud-gen..."
docker run -d --name ruoyi-cloud-gen -p 9202:9202 -v /home/xxx/ruoyi_could_drg/server/modules/gen/logs:/app/xxx/ruoyi_could_drg/gen/logs --restart=always ruoyi-cloud-gen
wait
if [ $? -eq 0 ]; then
  echo "Container started successfully."
else
  echo "Failed to start container."
fi

# 查看容器日志
echo "Showing container logs..."
docker logs -f --tail 200 ruoyi-cloud-gen
if [ $? -eq 0 ]; then
  echo "Container logs displayed successfully."
else
  echo "Failed to display container logs."
fi
```
