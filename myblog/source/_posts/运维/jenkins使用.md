---
cover: /img/linux.jpeg
---
# jenkins使用

### 前言

Jenkins是一个开源的持续集成和持续交付工具，可以帮助团队自动化构建、测试和部署软件项目。以下是使用Jenkins的基本流程

### 安装

1. docker 安装jenkins

```
docker pull jenkins/jenkins
```

2. 启动Jenkins容器：下载完Jenkins镜像后，可以使用以下命令启动一个Jenkins容器：

```
docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins
```

3. 访问Jenkins：在容器启动后，你可以通过浏览器访问Jenkins的Web界面，在浏览器中输入以下地址：<http://localhost:8080>
   这个命令会打开Jenkins的Web界面，你可以在这里进行Jenkins的初始化设置，例如创建管理员账户等。
4. 创建Jenkins项目：在Jenkins的管理界面中，你可以创建一个新的项目，并配置项目的源代码仓库、构建脚本、测试脚本等信息。
5. 配置项目，增加git仓库配置
   配置Git仓库的Deploy Key可以使得Jenkins可以自动从Git仓库中拉取代码，进行自动构建和部署。以下是配置Git仓库Deploy Key的步骤：
6. 生成SSH key：在Jenkins服务器上，打开终端窗口，并执行以下命令生成SSH key：

  ```ssh-keygen -t rsa -b 4096 -C "your_email@example.com"```
  在执行该命令时，需要将"your_email@example.com"替换为你的Email地址。

  注意，执行这个命令需要进入到jenkins挂载的目录下执行，比如宿主主机有个/home/jenkins 目录，指的就是容器内jenkins使用的目录，在这里执行ssh key生成。这里的公钥私钥就是jenkins服务的秘钥。

7. 添加SSH key到Git仓库：将刚刚生成的SSH key添加到Git仓库的Deploy Key中。首先，复制公钥（id_rsa.pub）的内容。然后，打开Git仓库的设置页面，在Deploy Keys选项卡中，点击"Add deploy key"按钮，将公钥内容粘贴到"Key"字段中，并为该Deploy Key设置一个描述名称。

8. 在Jenkins中配置SSH key：在Jenkins的管理界面中，打开"Credentials"页面，点击"Global credentials"选项卡，然后点击"Add Credentials"按钮。在弹出的对话框中，选择"SSH Username with private key"选项，并填写以下信息：

   "Username"字段：填写你在Git仓库中使用的用户名。
   "Private Key"字段：点击"Enter directly"按钮，将刚刚生成的私钥（id_rsa）的内容粘贴到文本框中。
   "ID"字段：填写一个描述性的名称，用于识别该SSH key。

9. 在Jenkins项目中配置Git仓库：在Jenkins项目的配置页面中， git模块。地址是git@xxxx ssh地址

完成以上步骤后，Jenkins就可以使用Deploy Key从Git仓库中拉取代码了。

10. 在项目的根目录增加一个Jenksfile 文件大小写敏感

    ```
    pipeline {
        agent any
    
        environment {
            // 目录隶属用户用户组
            DEPLOY_DIR_USER = 'drg'
            // 项目
            PROJECT_MAIN = 'main_project'
            // pnpm打包好的主目录
            TARGET_HOME_DIR = "${env.WORKSPACE}"
            // 远程主机ip
            DEPLOY_REMOTE_SERVER = 'drg@10.0.0.143'
            // 远程部署目录
            DEPLOY_DIR_MAIN = "${params.DEPLOY_DIR_MAIN}"
        }
    
        stages {
            stage('Clean') {
                steps {
                    // clean workspace
                    cleanWs()
                }
            }
    
            stage('Checkout') {
                dir('{env.PROJECT_MAIN}') {
                        checkout scm
                }
            }
            stage('Add env') {
                steps {
                    echo "${params.environmental_main}"
                    dir('{env.PROJECT_MAIN}') {
                        sh "git log -20 --grep=\"^Merge pull request\" --pretty=format:\"%H<br>\" -20 | awk '{printf \"%s\", \$0}' > ${env.TARGET_HOME_DIR}/${WORKSPACE}/public/result.html"
                    }
                    echo "version:${env.BUILD_NUMBER}"
                    script {
                        def packageJSON = readJSON file: "${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/package.json"
                        def packageJSONVersion = packageJSON.version
                        println "version:${packageJSONVersion}"
                        // 主项目环境变量
                        write_file_path = "${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/.env"
                        file_content = "${params.environmental_main}\nICE_VERSION=${packageJSONVersion}-${env.BUILD_NUMBER}\nICE_PR_HASH_URL=${params.deploySite1}result.html"
                        // write file
                        writeFile file: write_file_path, text: file_content, encoding: 'UTF-8'
                        // read file
                        fileContents = readFile file:write_file_path, encodeing: 'UTF-8'
                        println fileContents
    
                        // 修改build.json的ts检查
                        buildJsonPath = "${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/build.json"
                        def buildJson = readJSON file: buildJsonPath
                        def tsChecker = buildJson.tsChecker
                        println "tsChecker: ${tsChecker}"
                        buildJson.tsChecker = false
                        writeJSON file: buildJsonPath, json: buildJson
                    }
                }
            }
            stage('Build') {
                steps {
                    nodejs('nodejs19.6.0') {
                        dir("${env.PROJECT_MAIN}") {
                            sh '''
                                pwd
                                npm --version
                                node --version
                                pnpm i --frozen-lockfile
                                pnpm codegen
                                pnpm run build
                                '''
                        }
                    }
                }
            }
    
            stage('Deploy') {
                steps {
                 // 脚本生成
                // sshPublisher(publishers: [sshPublisherDesc(configName: 'nginx_server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: 'build', sourceFiles: 'build/**')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
    
                    echo "清空部署目录 [${env.DEPLOY_DIR_MAIN}] ... start"
    
                    sh "ssh ${env.DEPLOY_REMOTE_SERVER} rm -rf ${env.DEPLOY_DIR_MAIN}"
    
                    sh "ssh ${env.DEPLOY_REMOTE_SERVER} mkdir ${env.DEPLOY_DIR_MAIN}"
    
                    echo "清空部署目录 [${env.DEPLOY_DIR_MAIN}] ... ok"
    
                    echo "复制部署目录 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/build] to [${env.DEPLOY_DIR_MAIN}] ... start"
    
                    sh "scp -r ${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/build/* ${env.DEPLOY_REMOTE_SERVER}:${env.DEPLOY_DIR_MAIN}"
    
                    echo "复制部署目录 [${env.TARGET_HOME_DIR}/${env.PROJECT_MAIN}/build] to [${env.DEPLOY_DIR_MAIN}] ... ok"
    
                    echo "\n修改部署目录权限及用户组 [${env.DEPLOY_DIR_MAIN}] ... start"
    
                    sh "ssh ${env.DEPLOY_REMOTE_SERVER} chown -R ${env.DEPLOY_DIR_USER}:${env.DEPLOY_DIR_USER} ${env.DEPLOY_DIR_MAIN}"
    
                    sh "ssh ${env.DEPLOY_REMOTE_SERVER} chmod -R 755 ${env.DEPLOY_DIR_MAIN}"
    
                    echo "\n修改部署目录权限及用户组 [${env.DEPLOY_DIR_MAIN}] ... ok"
    
                    //调用方法得到日志 并 输出
    
                    script {
                        def changeString = getChangeString()
                        echo "$changeString"
                        // 发送消息
                        slackSend channel: '#drg-verify', color: '#7CFC00', message: "DRG-verify-打包部署成功 部署地址：${params.deploySite1} \n Build Started: ${env.JOB_NAME} ${env.BUILD_NUMBER} \n 提交信息: \n $changeString", teamDomain: 'troy-e1l2741', tokenCredentialId: 'COItrECTK3yvrbLtvtEMwoo2'
                    }
    
                    echo '\n--------------- 部署完成 ---------------'
                }
            }
        }
    }
    
    ```

11. 大概能看懂上述的过程，具体需要自己微调。

    有几个地方特殊注意下：

    - 可以在项目中配置环境变量。用params. 来引入
    - 也可以自己写入环境变量，env使用
    - 还有slack推送的配置，需要到jenkins下载插件，配合slack的jenkins插件，配置通信密钥。
    - jenkins在自己容器内会有工作区，dir用于指定在那个目录执行下面的流程，默认env.WORKSPACE 可以得到当前工作区目录

12. 目前流程需要自己手动执行

13. 如果需要git仓库发生变动时候就执行，还需要配置webhook👇

14. 去git仓库打开setting - webhook

15. 填写<https://> your jenkins site .com/github-webhook/

16. 去jenkins后台，项目配置中勾选GitHub hook trigger for GITScm polling

17. 可以了，试试把
