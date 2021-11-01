**github检索技巧**

```java
//1.in关键词限制搜索范围
	xxx关键字 in:name或description或readme
	秒杀系统eg:seckill in:name,readme,description
//2.star和fork范围搜索
	springboot stars:>=5000 //查找stars数大于5000的springboot项目
	springcloud forks:>=500 //查找forks数大于500的springcloud项目
	springboot forks:100..200 stars:80..100 //forks在100到200之间,stars在80到100之间
//3.awesome加强搜索【学习重要】
	awesome redis //加强搜索redis,搜索优秀的redis相关的项目，包括框架、教程、书籍等
//4.高亮显示某一行代码【github交互开发方便，直接给链接指出代码行数】
	eg: 		       https://github.com/codingXiaxw/seckill/blob/master/src/main/java/cn/codingxiaxw/dao/SeckillDao.java#L13(链接直接给出第13行,并高亮显示。#L13-L23:高亮显示13-23行)
//5.项目内搜索
	英文字母t（搜索方便，直接整成一个列表）
	其他快捷键：https://help.github.com/en/articles/using-keyboard-shortcuts
```

