https://blog.csdn.net/chiceT/article/details/96474540

一番查阅之后，明白了，点击效果是水波纹的，这玩意需要在最近的一个 Material 类控件里绘制，并不是在使用的控件的本体使用，所以我的问题就明确了，点击效果被绘制到外面了。

解决之道
咱给 IconButton 加一层 Material 嵌套

```
Material(
                          type: MaterialType.circle,
                          color: Colors.transparent,
                          clipBehavior: Clip.antiAlias,
                          child: IconButton(
                            icon: Icon(Icons.mic_outlined),
                            onPressed: () {},
                            padding: EdgeInsets.all(20.0),
                            iconSize: 50,
                          ),
                        )

```
