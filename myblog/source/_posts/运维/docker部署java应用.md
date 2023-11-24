## java应用部署

> 记录一下部署公司java项目的过程。顺便写了个简单脚本。

1. maven clean
2. maven package
3. 产物拷贝到服务器
4. dockerfile拷贝到服务器
5. 连接远程服务器
6. 清理原有的容器，镜像
7. 执行镜像构建命令
8. 启动容器
9. 查看日志

#### dockerFile

```dockerfile
# 基础镜像
FROM docker.io/library/openjdk:8-jre-alpine@sha256:f362b165b870ef129cbe730f29065ff37399c0aa8bcab3e44b51c302938c9193
# author
MAINTAINER project

# 暴露的端口，根据您的实际项目端口进行调整
EXPOSE 9401

# 挂载目录
VOLUME /app/xxx/project
# 创建目录
RUN mkdir -p /app/xxx/project
# 指定路径
WORKDIR /app/xxx/project
# 将 jar 文件复制到容器中
COPY project.jar /app/xxx/project
# 时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone

# 启动代码生成服务
ENTRYPOINT ["java", "-Xms256m", "-Xmx2048m", "-XX:MetaspaceSize=128m", "-XX:MaxMetaspaceSize=512m", "-jar", "project.jar"]
```

#### 一键部署脚本

```sh
# !/bin/bash

# 打包当前的java项目
#mvn clean package

# 传递产物到远程服务器
scp target/project.jar user@remoteServer:/home/xxx/project
scp Dockerfile user@remoteServer:/home/xxx/project


# 登录远程服务器执行docker命令
ssh user@remoteServer << EOF
set -e

cd /home/xxx/project
# 停止容器
if docker ps -a --format '{{.Names}}' | grep -q "^xxx-project$"; then
  echo "Stopping container xxx-project..."
  docker stop xxx-project
  wait
  if [ $? -eq 0 ]; then
    echo "Container stopped successfully."
  else
    echo "Failed to stop container."
    exit 1
  fi
else
  echo "Container xxx-project not found, skipping stop operation."
fi

# 删除容器
if docker ps -a --format '{{.Names}}' | grep -q "^xxx-project$"; then
  echo "Removing container xxx-project..."
  docker rm xxx-project
  wait
  if [ $? -eq 0 ]; then
    echo "Container removed successfully."
  else
    echo "Failed to remove container."
    exit 1
  fi
else
  echo "Container xxx-project not found, skipping remove operation."
fi

# 删除镜像
if docker images --format '{{.Repository}}' | grep -q "^xxx-project$"; then
  echo "Removing image xxx-project..."
  docker rmi xxx-project
  wait
  if [ $? -eq 0 ]; then
    echo "Image removed successfully."
  else
    echo "Failed to remove image."
    exit 1
  fi
else
  echo "Image xxx-project not found, skipping remove operation."
fi

# 构建镜像

echo "Building image xxx-project..."
docker build -f Dockerfile -t xxx-project .
wait
if [ $? -eq 0 ]; then
  echo "Image built successfully."
else
  echo "Failed to build image."
fi

# 运行容器

echo "Running container xxx-project..."
docker run -d --name xxx-project -p 9401:9401 -v /home/xxx/project/logs:/app/xxx/project/logs --restart=always xxx-project
wait
if [ $? -eq 0 ]; then
  echo "Container started successfully."
else
  echo "Failed to start container."
fi

# 查看容器日志

echo "Showing container logs..."
docker logs -f --tail 200 xxx-project
if [ $? -eq 0 ]; then
  echo "Container logs displayed successfully."
else
  echo "Failed to display container logs."
fi

EOF


```

记录下遇到的小问题：
容器启动后，日志显示

```
no main manifest attribute, in xxx.jar
no main manifest attribute, in xxx.jar
no main manifest attribute, in xxx.jar
no main manifest attribute, in xxx.jar
```

解决方案：

pom中增加这个 打包插件

```xml

<build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```

好了接下来，你可能会得到另外一个错误：

```
org/springframework/boot/maven/RepackageMojo has been compiled by a more recent version of the Java Runtime (class file version 61.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

不要慌，解决方案在这里：
指定打包插件的版本,最终如下

```xml
 <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```
