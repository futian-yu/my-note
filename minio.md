**minio**

​	minio是一个存储文件的工具。它兼容亚马逊S3云存储服务接口.可以存储图片、视频、文件、容器等。对象带下可以从0到5T不等.

​	桶可以理解为根目录，根目录下的一直链接到文件的所有url可以认为是文件名(objectName),所以分目录存储只是形式上的. 下载文件都是先找到桶，然后根据文件名来下载对应文件.；

```java
String bucketName = "log";//桶
String objectName = "loginLog/2019-8-26-xxxxx.txt";//文件名
InputStream stream = minioClient.getObject(bucketName,objectName);
```

