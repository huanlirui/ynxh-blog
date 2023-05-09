
### 大致流程

1. pr推送
2. 触发jenkins构建任务
3. 清空jenkins工作目录
4. 拉取git代码
5. 在工作区创建/Public目录
6. 新建并写入git log 到result.html
7. 使用Maven进行构建打包
8. 产物在工作区的/ruoyi-admin/target/ruoyi-admin.jar
9. 将产物和Dockerfile 传到远程服务器的/home/xxx/xxx-server 目录
10. 在远程服务器端 执行docker build 打包成xxx-server镜像
11. 在远程服务端 执行 docker run 运行在20111端口，并把日志挂载到 /home/xxx/xxx-server/logs
12. 向slack推送

实现方式：

1. 安装maven ，Publish Over SSH , docker 插件 ，使用maven project 流程来构建
2. 使用pipeline 和 jenkinsfile 自定义流程 的方式实现。

### 尝试方式一

- 安装maven 插件，配置自动安装
- 安装docker 插件，配置自动安装
- 安装 Publish Over SSH 插件，配置时发现，ssh测试一直不能成功。
   步骤如下：  
   1. 将jenkins 容器中的.ssh 下的id_rsa.pub 公钥拷贝到 远程服务器的authorized_keys 文件
   2. 将jenkins 容器中的.ssh 下的id_rsa 私钥拷贝到jenkins 的SSH 插件全局配置
   3. test 报错
<img width="1243" alt="image" src="https://user-images.githubusercontent.com/40491203/236610686-32ffdb4b-b6e0-42f8-8d04-8e215aea6d19.png">

> 然而，直接在容器内，ssh 是可以免密直接连接到远程的，就是这个插件的test过不去。

- 新建maven 项目构建流程
- 在jenkins容器中 .ssh目录 生成密钥对

  ```
  ssh-keygen -t rsa -b 4096 -C "xxx-server@troy.com"
  ```

- 配置git 地址与deploy key
- 配置webhook
- 配置build 命令

  ```
  clean package -Dmaven.test.skip=true
  ```

- 构建成功后 执行 Send build artifacts over SSH 把jar包和dockerfile 丢过去
- 构建成功后 再执行  shell 脚本 进行docker build 和 run

> 由于卡在了  Publish Over SSH 插件 配置，而且这个方式似乎不够灵活，还需要添加一些shell脚本来实现git log 写入和slack推送等，而且还得去学习各种插件配置，无法把已有的jenkinsfile 中的流程一键迁移（也得一个一个加shell脚本搞），遂暂时放弃。去搞方式二

### 尝试方式二

1. 项目根目录创建 jenkinsfile 推到仓库
2. 在jenkinsfile 语法中完成下面步骤👇🏻
3. 调用sh 命令完成打包步骤
4. 把产物和dockerfile丢到远程服务器
5. ssh 到远程服务器打包镜像和运行镜像
6. 运行成功后，slack推送消息

### 这里有几个坑记录下

1. 由于jenkins容器中是没有安装过docker和maven的，必须手动去安装一下。

   ```
   apt update
   apt install maven
   ```

2. 安装docker是个坑，apt源没有docker

#### 以下回答来自gpt，亲测可用

   要在Debian GNU/Linux 11上安装Docker，您可以按照以下步骤操作：

   1. 更新软件包列表：

      ```
      sudo apt-get update
      
      ```

   2. 安装依赖包，以便您可以通过HTTPS使用Docker存储库：

      ```
      sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
      
      ```

   3. 添加Docker官方GPG密钥：

      ```
      curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
      ```

   4. 添加Docker存储库到您的APT源列表：

      ````
      echo \
        "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      ```
      ````

   5. 更新APT软件包列表：

      ```
      sudo apt-get update
      ```

   6. 安装Docker Engine和docker-compose：

      ```
      sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose
      ```

   7. 启动Docker服务：

      ```
      sudo systemctl start docker
      ```

   8. 验证Docker是否已成功安装：

      ```
      sudo docker run hello-world
      
      如果一切正常，您应该会看到一个欢迎消息，表示Docker已经成功安装。
      ```

   3. docker build 必须要进入到远程部署目录去执行，这个点没注意就会在当前工作目录执行了。

   4. 语法坑，注意转义符号，双引号内的双引号，这样才能保证在 ssh 后的远程目录执行docekr build

  ```
 sh "ssh ${env.DEPLOY_REMOTE_SERVER} \"cd ${env.DEPLOY_DIR_MAIN} && pwd && docker build -t xxx-server .\""
  ```

5. 权限坑 ，xxx用户是没有docker 命令运行权限的。

```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
```

   需要切换到root用户执行，把xxx用户添加到docker用户组

  ```
sudo usermod -aG docker xxx  
  ```

6. 运行在docker 中的java程序想要链接另一个容器的redis，不能使用127.0.0.1 或者localhost

- 可以在运行时候，redis，java应用都使用 -network 组同一个网，然后使用容器名称连接即可
- 或者使用命令查看下redis 的容器地址。使用容器ip地址来连接

  ```
  docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
  ```

7. docker run 运行的时候一定要记得加上-d 后台运行，不然无法执行到下一步。

9. java项目Captcha生成验证码，docker部署时字体包空指针异常
原因是制作镜像时使用的 openjdk:8-jdk-alpine 没有相关字体包，调用createImage()方法时，会报空指针异常
解决方案就是在dockerfile中增加以下代码：

```
FROM openjdk:8-jre-alpine

#captcher 字体包
RUN set -xe \
&& apk --no-cache add ttf-dejavu fontconfig

```

最终dockerfile ：

```
# 构建镜像的第二阶段

FROM openjdk:8-jre-alpine

# captcher 字体包
RUN set -xe \
&& apk --no-cache add ttf-dejavu fontconfig

# 设置工作目录

WORKDIR /app

# 将 jar 文件复制到容器中

COPY ruoyi-admin.jar /app/

# 暴露的端口，根据您的实际项目端口进行调整

EXPOSE 20111

# 设置容器启动时执行的命令

# ENTRYPOINT ["java", "-jar", "ruoyi-admin.jar"]
ENTRYPOINT ["java", "-Xms256m", "-Xmx1024m", "-XX:MetaspaceSize=128m", "-XX:MaxMetaspaceSize=512m", "-jar", "ruoyi-admin.jar"]

# 添加下面这一行，将宿主机的日志目录挂载到容器中

VOLUME /home/xxxx/logs:/app/logs

```

jekinsfile:

```
pipeline {
    agent any

    environment {
        // 目录隶属用户用户组
        DEPLOY_DIR_USER = 'xxx'
        // 项目
        PROJECT_MAIN = 'main_project'
        // 打包好的主目录
        TARGET_HOME_DIR = "${env.WORKSPACE}"
        // 远程主机ip
        DEPLOY_REMOTE_SERVER = 'xxx@10.0.0.远程'
        // 远程部署目录
        DEPLOY_DIR_MAIN = "${params.DEPLOY_DIR_MAIN}"
        // jar包路径
        JAR_DIR = 'ruoyi-admin/target/ruoyi-admin.jar'

        JAR_NAME = 'ruoyi-admin.jar'

        CONTAINER_NAME = 'xxx-server'
    }

    stages {
        stage('Clean') {
            steps {
                // clean workspace
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                dir("${env.PROJECT_MAIN}") {
                    checkout scm
                }
            }
        }

        stage('Create Directories') {
            steps {
                sh "mkdir -p ${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/public"
            }
        }
        stage('Generate log HTML ') {
            steps {
                echo "${params.environmental_main}"
                dir("${env.PROJECT_MAIN}") {
                    sh "git log -20 --grep=\"^Merge pull request\" --pretty=format:\"%H<br>\" -20 | awk '{printf \"%s\", \$0}' > ${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/public/result.html"
                }
                echo "version:${env.BUILD_NUMBER}"
            }
        }
        stage('mvn clean package') {
            steps {
                dir("${env.PROJECT_MAIN}") {
                    sh 'mvn clean package -Dmaven.test.skip=true'
                }
            }
        }
        stage('Deploy') {
            steps {
             // 脚本生成
            // sshPublisher(publishers: [sshPublisherDesc(configName: 'nginx_server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: 'build', sourceFiles: 'build/**')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])

                echo "清空部署目录下的jar文件 [${env.DEPLOY_DIR_MAIN}/${env.JAR_NAME}] ... start"

                sh "ssh ${env.DEPLOY_REMOTE_SERVER} rm  ${env.DEPLOY_DIR_MAIN}/${env.JAR_NAME}"

                echo "清空部署目录下的jar文件 [${env.DEPLOY_DIR_MAIN}/${env.JAR_NAME}]  ... ok"

                echo "清空部署目录下的DockerFile文件 [${env.DEPLOY_DIR_MAIN}/Dockerfile] ... start"

                sh "ssh ${env.DEPLOY_REMOTE_SERVER} rm  ${env.DEPLOY_DIR_MAIN}/Dockerfile"

                echo "清空部署目录下的jar文件 [${env.DEPLOY_DIR_MAIN}/Dockerfile]  ... ok"

                echo "复制部署jar文件 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/${env.JAR_DIR}] to [${env.DEPLOY_DIR_MAIN}] ... start"

                sh "scp -r ${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/${env.JAR_DIR} ${env.DEPLOY_REMOTE_SERVER}:${env.DEPLOY_DIR_MAIN}"

                echo "复制部署jar文件 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/${env.JAR_DIR}] to [${env.DEPLOY_DIR_MAIN}] ... ok"

                echo "复制Dockerfile文件 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/Dockerfile] to [${env.DEPLOY_DIR_MAIN}] ... start"

                sh "scp -r ${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/Dockerfile ${env.DEPLOY_REMOTE_SERVER}:${env.DEPLOY_DIR_MAIN}"

                echo "复制Dockerfile文件 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/Dockerfile] to [${env.DEPLOY_DIR_MAIN}] ... ok"

                echo 'docker 打包  ...start'

                // sh "ssh ${env.DEPLOY_REMOTE_SERVER} cd ${env.DEPLOY_DIR_MAIN} && pwd && docker build -t xxx-server ."

                sh "ssh ${env.DEPLOY_REMOTE_SERVER} \"cd ${env.DEPLOY_DIR_MAIN} && pwd && docker build -t xxx-server .\""

                echo 'docker 打包  ...ok'

                script {
                    try {
                        echo "判断下有没有${env.CONTAINER_NAME}容器"
                        def containerExists = sh(
                        returnStatus: true,
                        script: "docker ps -aqf name=${env.CONTAINER_NAME}"
                    ) == 0
                        if (containerExists) {
                            echo "${env.CONTAINER_NAME}容器 存在"

                            sh "docker stop ${env.CONTAINER_NAME}"

                            echo "${env.CONTAINER_NAME}容器 停止"

                            sh "docker rm ${env.CONTAINER_NAME}"

                            echo "${env.CONTAINER_NAME}容器 删除"
                        }
                        echo 'docker 后台启动  ...start'

                        sh "ssh ${env.DEPLOY_REMOTE_SERVER} docker run -d --name xxx-server -p 20111:20111 -v /home/xxx/xxx-server/logs:/app/logs -v xxx-server-data:/data xxx-server"

                        echo 'docker 启动  ...ok'
                        //调用方法得到日志 并 输出

                        def changeString = getChangeString()
                        echo "$changeString"
                        // 发送消息
                        slackSend channel: '#xxx-verify', color: '#7CFC00', message: "xxx-server-打包部署成功 swagger文档地址：${params.deploySite1} \n Build Started: ${env.JOB_NAME} ${env.BUILD_NUMBER} \n 提交信息: \n $changeString", teamDomain: 'xxxx', tokenCredentialId: 'xxx'

                        echo '\n--------------- 部署完成 ---------------'
                } catch (err) {
                        echo "部署失败: ${err}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
}

@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ''

    echo 'Gathering SCM changes'
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }
    if (!changeString) {
        changeString = ' - No new changes'
    }
    return changeString
}

```
