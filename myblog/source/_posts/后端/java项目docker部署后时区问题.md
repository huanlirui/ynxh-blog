
# java项目使用docker部署后时区问题

今天发现个诡异的问题。服务器时区是一样的，mysql时区设置是system

但是时间在入库前的sql语句中会被偏移8小时。

一看就是时区问题。

原来是正式环境使用docker部署。dockerFile中需要指定时区。

```dockerFile
#时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

然而这个方法并没有用。

```java
 TimeZone timeZone = TimeZone.getDefault();
        System.out.println("当前时区：" + timeZone.getID());
```

依然打印的是UTC时区。

docker 容器已经是 中国时区了。奇怪

查资料查到这个写法，试了下成功了

``` dockerFile
#时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

在docker run 的时候，增加参数

```
-v /etc/localtime:/etc/localtime:ro \\
```
