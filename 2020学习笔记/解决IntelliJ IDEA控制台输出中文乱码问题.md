# 解决IntelliJ IDEA控制台输出中文乱码问题

IntelliJ IDEA 真的是一款很方便的Java开发工具，但是关于中文乱码这个问题我不得不吐槽，这个编码也弄得这么麻烦干嘛，真想找idea开发者干架，我敢打包票我能在一分钟之内一拳飞过去让他跪下掐指住我的人中求我不要死 ~我有一块托大的腹肌，害羞~ 咳咳，扯远了，下面就讲一下怎么解决常见的中文乱码问题。

# 1、

找到idea的安装目录——> bin——>找到下图文件并分别在这两个文件内容的末尾添加如下代码

```
-Dfile.encoding=UTF-8
复制代码
```



![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c377ad786ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

什么？小白不会操作？？~还是演示一个例子吧，我太难了...~

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c3776e50911?imageslim)

两个文件都是如上操作！！！



# 2、

配置项目编码及IDE编码，File——>Setting 进入

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c37755fcdb3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



# 3、

配置项目启动服务器参数，如果是普通java项目则如下操作

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c3776583f04?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

若是Web项目则如下

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c377d1206b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



# 4、

到这里，如果以上招数都使遍了，还是不行？你要知道真相只有一个~

可能是你无意中点了右下角或者那里的编码，改了一下，或者就被idea记录到 `encodings.xml` 中，当你再次访问的时候，它就会用那种编码 ~这种情况我想可能只有idea开发者和idea开发者的妈妈知道了~，解决方法就是打开项目目录下的.idea的文件夹，里面有个`encodings.xml` 的文件，打开`encodings.xml` 然后除了`UTF-8` 的都删了就ojbk了，具体操作如下：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c377d73723c?imageslim)



# 5、

最重要的一步就是重启idea，是的关掉所以idea窗口，`重启`！！！

最后，博主我就是使用到最后一步才成功的，当然这个是个别情况，我就是不小心点导入项目编码不一致从而点到编码跟换之后就乱码的，花了一上午解决，最终解决效果如下

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/11/30/16eb9c38214a7c9b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





### **注意：**idea64.exe.vmoptions文件添加utf-8编码，只是在安装路径上添加可能没用；需要在idea中搜索文件名，手动添加上utf-8编码；

![1594630273511](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1594630273511.png)

