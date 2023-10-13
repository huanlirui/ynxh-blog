# Docker 安装 ElasticSearch

## 1 安装说明

在平时工作的时候，开发环境大多数会安装单机`ElasticSearch`，但生产环境基本会安装`ElasticSearch`集群版，所以我们接下来实现一下`ElasticSearch`单机安装，下一节实现集群安装，但安装也大多数采用`Docker`安装。不过中文搜索，会实现分词器集成，可以采用 IK 分词器。`ElasticSearch`采用`Kibana`实现数据可视化分析也是当前主流，所以我们除了安装`ElasticSearch`和`IK`分词器外，还需要安装`Kibana`。

安装实践:

`1:ElasticSearch单机安装
2:IK分词器安装
3:Kibana安装`

## 2 Docker 安装 ElasticSearch

当前`ElasticSearch`已经到了`8.0`，新版本都有很多新特性，性能和功能都有大幅提升，我们建议使用较高版本，这里将采用`7.12.1`版本。

### 2.1 网络创建

高版本安装`Kibana`的时候需要和`ElasticSearch`在同一网段内，所以采用`docker`安装首先要确认网段，为了方便操作，我们直接创建一个网络，创建脚本如下:

```bash
docker network create itmentu-net
```

### 2.2 ElasticSearch 安装

安装`ElasticSearch`脚本如下:

```bash
docker run -d \
 --name elasticsearch \
 -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
 -e "discovery.type=single-node" \
 -v es-data:/usr/share/elasticsearch/data \
 -v es-plugins:/usr/share/elasticsearch/plugins \
 --privileged \
 --network itmentu-net \
 -p 9200:9200 \
 -p 9300:9300 \
elasticsearch:7.12.1
```

命令说明：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定`elasticsearch`的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定`elasticsearch`的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定`elasticsearch`的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network itmentu-net` ：加入一个名为`itmentu-net`的网络中
- `-p 9200:9200`：端口映射配置

`Docker`安装`ElasticSearch`下载可能会比较慢，需要耐心等待，效果如下：

![image-20220102123013210](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffa4c684eb7f40eaaebe149053f3c04e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

安装完成后，在浏览器中输入：[http://192.168.211.130:9200/](https://link.juejin.cn/?target=http%3A%2F%2F192.168.211.130%3A9200%2F "http://192.168.211.130:9200/") 即可看到`elasticsearch`的响应结果：

![image-20220102122738908](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe45cf37b43c49839538683f5ddf0f15~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 3 安装 Kibana

我们可以基于`Http`请求操作`ElasticSearch`，但基于`Http`操作比较麻烦，我们可以采用`Kibana`实现可视化操作。

### 3.1 Kibana 介绍

![image-20220102122520286](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bdb2b2c21f44a18af80bfd15a8bfd79~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

`Kibana` 是一个免费且开放的用户界面，能够让您对 `Elasticsearch` 数据进行可视化，并让您在 `Elastic Stack` 中进行导航。您可以进行各种操作，从跟踪查询负载，到理解请求如何流经您的整个应用，都能轻松完成。

`Kibana` 让您能够自由地选择如何呈现自己的数据。不过借助 `Kibana` 的交互式可视化，您可以先从一个问题出发，看看能够从中发现些什么。

可视化界面如下：

![image-20220102122626497](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2d7f768a8184c90a5112ef05985423d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 3.2 Kibana 安装

使用`Docker`安装`Kibana`非常简单，只需要执行如下命令即可，但是执行命令需要注意`Kibana`操作的`ElasticSearch`地址，因为`Kibana`是需要连接`ElasticSearch`进行操作的，命令如下:

```bash
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://192.168.211.130:9200 \
--network itmentu-net \
-p 5601:5601  \
kibana:7.12.1
```

命令说明：

- `--network itmentu-net` ：加入一个名为`itmentu-net`的网络中，与`elasticsearch`在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://192.168.211.130:9200"`：设置`elasticsearch`的地址，因为`kibana`已经与`elasticsearch`在一个网络，因此可以用容器名直接访问`elasticsearch`，也可以写 IP 地址实现访问。
- `-p 5601:5601`：端口映射配置

安装的时候如果没有镜像，会下载镜像，效果如下：

![image-20220102123516823](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a1161c1c5094472bc5fa1ccf5736d80~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

`kibana`安装会比较耗时间，也需要耐心等待下载安装完成，如果想实时知道服务安装运行的状态，可以通过查看日志实现，查看日志如下：

复制代码

```bash
docker logs -f kibana
```

日志中如果出现了`http://0.0.0.0:5601`即可访问`Kibana`后台服务，日志如下：

![image-20220102123646972](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccc91500ccaf46e79afb4faf37d7505a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

访问`http://192.168.211.130:5601`效果如下：

![image-20220102123829118](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/549cd772a85648d987d4aea267f96248~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

可以点击`Add data`，添加示例数据，如下图，随意选一个即可，不选其实也是可以的。

![image-20220102123929231](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08d1c881b1944ea294348708f7beadc5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 3.3 Kibana 中文配置

我们发现`Kibana`是英文面板，看起来不是很方便，但`Kibana`是支持中文配置，所以我们可以把`Kibana`配置成中文版，便于我们操作。

切换中文操作如下:

复制代码

```bash
#进入容器
docker exec -it kibana /bin/bash
#进入配置文件目录
cd /usr/share/kibana/config
#编辑文件kibana.yml
vi kibana.yml
#在最后一行添加如下配置
i18n.locale: zh-CN
#保存退出
exit
#并重启容器
docker restart kibana
```

等待`Kibana`容器启动，再次访问[http://192.168.211.130:5601/效果如下：](https://link.juejin.cn/?target=http%3A%2F%2F192.168.211.130%3A5601%2F%25E6%2595%2588%25E6%259E%259C%25E5%25A6%2582%25E4%25B8%258B%25EF%25BC%259A "http://192.168.211.130:5601/%E6%95%88%E6%9E%9C%E5%A6%82%E4%B8%8B%EF%BC%9A")

![image-20220102124446277](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f0f6300557340e4b3adb6e37473f147~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 4 IK 分词器安装

我们打开`Kibana`，点击开发工具，操作如下：

![image-20220102130545516](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/813ac36dc02e4ef1a9b0fb0fae69294b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

输入如下操作，用于查询分词：

![image-20220102130749934](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/567f9030d0c4414db43b9f21851cdbc4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

上图测试代码如下:

复制代码

```
GET /_analyze
{
 "analyzer": "standard",
 "text": "IT门徒，带你打开通往梦想的门！"
}
```

表示使用`standard`对`IT门徒，带你打开通往梦想的门！`进行分词。

`分词`：提取一句话或者一篇文章中的词语。

我们在使用`ElasticSearch`的时候，默认用`standard`分词器，但`standard`分词器使用的是按空格分词，这种分词操作方法不符合中文分词标准，我们需要额外安装中文分词器。

### 4.1 IK 分词器介绍

`IK Analyzer`是一个开源的，基于 java 语言开发的轻量级的中文分词工具包。从 2006 年 12 月推出 1.0 版开始， `IKAnalyzer`已经推出了多个大版本。最初，它是以开源项目`Luence`为应用主体的，结合词典分词和文法分析算法的中文分词组件。`IK Analyzer`则发展为面向`Java`的公用分词组件，独立于`Lucen`e 项目，同时提供了对`Lucene`的默认优化实现。

`ElasticSearch`内核其实就是基于`Lucene`，所以我们可以直接在`ElasticSearch`中集成 IK 分词器，`IK`分词器集成`ElasticSearch`下载地址：[github.com/medcl/elast…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmedcl%2Felasticsearch-analysis-ik%2Freleases "https://github.com/medcl/elasticsearch-analysis-ik/releases")

![image-20220102125305738](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96f25eb5318a40ec96e5b4ab8caf287d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 4.2 IK 分词器配置

下载安装包`elasticsearch-analysis-ik-7.12.1.zip`后，并解压，目录如下：

![image-20220102125602923](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef21870531ec423a8dfe4f1badb51671~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们只需要将上面`elasticsearch-analysis-ik-7.12.1`拷贝到`ElasticSearch`的`plugins`目录中即可，但由于当前服务采用的是`docker`安装，所以需要将文件拷贝到`docker`容器的`plugins`目录才行。

操作如下：

复制代码

```
#为了方便配置，我们将elasticsearch-analysis-ik-7.12.1改成ik文件夹
mv elasticsearch-analysis-ik-7.12.1 ik
#将ik文件夹拷贝到elasticsearch容器中
docker cp ik elasticsearch:/usr/share/elasticsearch/plugins
#重启容器
docker restart elasticsearch
```

操作效果如下：

![image-20220102130217434](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aac9883b11b947aca79e6df463322d87~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 4.3 分词测试

`IK`分词器包含两种模式：

- `ik_smart`：最少切分
- `ik_max_word`：最细切分

前面使用默认的`standard`分词器，对中文分词非常难用，安装 IK 分词器后，我们可以使用 IK 分词器测试，测试代码如下：

复制代码

```
GET /_analyze
{
 "analyzer": "ik_max_word",
 "text": "IT门徒，带你打开通往梦想的门！"
}
```

测试效果如下：

![image-20220102131106766](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2910ef87aad430097bc7e32fa620e31~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们可以发现对中文的分词效果是比较不错的，但也存在一些不足，比如`梦想的门`我们希望它是一个词，而`带你`我们希望它不被识别一个词，又该如何实现呢？

### 4.4 IK 自定义词典

IK 分词器支持自定义词典，包括自定义分词，也包含自定义停用分词，操作起来也非常简单。我们接下来实现一下自定义词典和停用词典。

#### 4.4.1 自定义词典

自定义词典，需要先创建自己的词典，再引用自己的词典即可。

复制代码

`1:创建词典
2:引用词典`

1)创建词典

在`config`中创建自己的词典，例如叫`itmentu_ext.dic`，在文件中添加自定义的词语，操作如下：

![image-20220102133144102](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbc91a1096ce4be6a6baa031d4ed01b5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们把自定义的词典`梦想的门`添加到了`itmentu_ext.dic`中了，这就是创建词典，如果后多个自定义次，需要换行加入，这里一定要注意中文分词设置编码格式为`UTF-8`。

2)引用词典

修改`config/IKAnalyzer.cfg.xml`引用自己创建的`itmentu_ext.dic`词典，配置如下:

![image-20220102131927984](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/047904c3f26e4ccd83fdfe414d9bcab2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

上图代码如下：

xml

复制代码

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE  SYSTEM "http://java.sun.com/dtd/.dtd">

<>
<comment>IK Analyzer 扩展配置</comment>

 <!--用户可以在这里配置自己的扩展字典 -->

<entry key="ext_dict">itmentu_ext.dic</entry>

 <!--用户可以在这里配置自己的扩展停止词字典-->

<entry key="ext_stopwords"></entry>

 <!--用户可以在这里配置远程扩展字典 -->
 <!-- <entry key="remote_ext_dict">words_location</entry> -->
 <!--用户可以在这里配置远程扩展停止词字典-->
 <!-- <entry key="remote_ext_stopwords">words_location</entry> -->

</>
```

我们将改好的文件重新上传到`elasticsearch`容器的`/usr/share/elasticsearch/plugins`目录下，重启`elasticsearch`容器即可。

操作如下:

复制代码

`#将上传的 config 拷贝到服务器

# 将 ik 文件夹拷贝到 elasticsearch 容器中

docker cp ik elasticsearch:/usr/share/elasticsearch/plugins

# 重启容器

docker restart elasticsearch`

在使用`Kibana`测试 `http://192.168.211.130:5601/app/dev_tools#/console` 效果如下：

![image-20220102133431459](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9d7b0f09a1348b8bc487b9351d0b517~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 4.4.2 自定义停用词汇

自定义停用词典和自定义词典一样，需要先创建自己的词典，再引用自己的词典即可。

复制代码

`1:创建词典
2:引用词典`

1)创建词典

在`config`中创建自己的停用词典，例如叫`itmentu_stop.dic`，在文件中添加自定义的停用词语，操作如下：

![image-20220102133606560](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9bc84adf9c84a8aab3301070f4328cb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

2)引用词典

修改`config/IKAnalyzer.cfg.xml`引用自己创建的`itmentu_stop.dic`停用词典，配置如下:

![image-20220102133807712](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d66ae26953e34c19ab228ad3bab10307~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

上图代码如下:

xml

复制代码

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE  SYSTEM "http://java.sun.com/dtd/.dtd">

<>
<comment>IK Analyzer 扩展配置</comment>

 <!--用户可以在这里配置自己的扩展字典 -->

<entry key="ext_dict">itmentu_ext.dic</entry>

 <!--用户可以在这里配置自己的扩展停止词字典-->

<entry key="ext_stopwords">itmentu_stop.dic</entry>

 <!--用户可以在这里配置远程扩展字典 -->
 <!-- <entry key="remote_ext_dict">words_location</entry> -->
 <!--用户可以在这里配置远程扩展停止词字典-->
 <!-- <entry key="remote_ext_stopwords">words_location</entry> -->

</>
```

我们将改好的文件重新上传到`elasticsearch`容器的`/usr/share/elasticsearch/plugins`目录下，重启`elasticsearch`容器即可。

操作如下:

复制代码

`#将上传的 config 拷贝到服务器

# 将 ik 文件夹拷贝到 elasticsearch 容器中

docker cp ik elasticsearch:/usr/share/elasticsearch/plugins

# 重启容器

docker restart elasticsearch`

在使用`Kibana`测试 `http://192.168.211.130:5601/app/dev_tools#/console` 效果如下：

![image-20220102134039836](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f134ad6b05464045956bb870300a672e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们可以发现，不再有`带你`的分词了，说明停用分词也生效了。

## 5 ElasticSearch 集群安装

部署 es 集群可以直接使用`docker-compose`来完成，不过要求你的 Linux 虚拟机至少有**4G**的内存空间。

首先我们需要创建 docker-compose 脚本，再运行脚本实现安装即可，集群配置脚本如下`elasticsearch.yml`：

yaml

复制代码

```yaml
version: '3.8'
services:
es01:
image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
container_name: elasticsearch01
environment:

- node.name=elasticsearch01
- cluster.name=es-docker-cluster
- discovery.seed_hosts=elasticsearch01,elasticsearch01
- cluster.initial_master_nodes=elasticsearch01,elasticsearch01,elasticsearch01
- bootstrap.memory_lock=true
- "ES_JAVA_OPTS=-Xms512m -Xmx512m"
  ulimits:
  memlock:
  soft: -1
  hard: -1
  volumes:
- data01:/usr/share/elasticsearch/data
  ports:
- 9200:9200
  networks:
- elastic
  es02:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
  container_name: elasticsearch02
  environment:
- node.name=elasticsearch02
- cluster.name=es-docker-cluster
- discovery.seed_hosts=elasticsearch01,elasticsearch03
- cluster.initial_master_nodes=elasticsearch01,elasticsearch02,elasticsearch03
- bootstrap.memory_lock=true
- "ES_JAVA_OPTS=-Xms512m -Xmx512m"
  ulimits:
  memlock:
  soft: -1
  hard: -1
  volumes:
- data02:/usr/share/elasticsearch/data
  networks:
- elastic
  es03:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
  container_name: elasticsearch03
  environment:
- node.name=elasticsearch03
- cluster.name=es-docker-cluster
- discovery.seed_hosts=elasticsearch01,elasticsearch02
- cluster.initial_master_nodes=elasticsearch01,elasticsearch02,elasticsearch03
- bootstrap.memory_lock=true
- "ES_JAVA_OPTS=-Xms512m -Xmx512m"
  ulimits:
  memlock:
  soft: -1
  hard: -1
  volumes:
- data03:/usr/share/elasticsearch/data
  networks:
- elastic
  volumes:
  data01:
  driver: local
  data02:
  driver: local
  data03:
  driver: local
  networks:
  elastic:
  driver: bridge
```

作者：bysecby
链接：<https://juejin.cn/post/7074115690340286472>
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
