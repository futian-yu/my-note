# Mybatis值trim标签

   Mybatis具有实现动态SQL的能力，使用这个特性免不了会用到trim这个标签，trim标签的功能简单来说就是自定义格式拼凑SQL语句。
        trim有4个属性：

> prefix：表示在trim包裹的SQL前添加指定内容
> suffix：表示在trim包裹的SQL末尾添加指定内容
> prefixOverrides：表示去掉（覆盖）trim包裹的SQL的指定首部内容
> suffixOverrides：表示去掉（覆盖）trim包裹的SQL的指定尾部内容

来看例子：
查找某个用户，通过名字和年龄

```java
public User getUser(@Param("LastName") String lastName,@Param("age") Integer age,@Param("phone") String phone);
1
```

对应xml，先看首先想到的：

```xml
<select id="getUser" result="user">
	select * from user_tab where last_name=#{lastName} and age=#{age} and phone=#{phone}
</select>
123
```

执行时输出的SQL语句
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181211220455409.png)

1. **prefix**

```xml
<select id="getUser" resultType="user">
	select * from user_tab 
	<trim prefix="where">
		last_name=#{lastName} and age=#{age} and phone=#{phone}
	</trim>
</select>
123456
```

输出同样SQL，这个就是prefix的效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181211220502544.png)

1. **prefixOverrides**
   当然实际使用时查找条件都需要做一些判断，最基本的是否为null。

```xml
<select id="getUser" resultType="user">
	select * from user_tab 
	<trim prefix="where">
		<if test="lastName != null">
			last_name=#{lastName}
		</if>
		<if test="age != null">
			and age=#{age}
		</if>
		<if test="phone != null">
			and phone=#{phone}
		</if>
	</trim>
</select>
1234567891011121314
```

执行语句与上面一样，但是假如现在lastName参数为null，我们再看：

```java
// 查询
User user = userMapper.getUser(null, 12,"120");
12
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/201812112215159.png)
因为lastName为null，所以第一个if不成立里面的SQL语句不拼接，第二个if里面的and边紧跟在where后面了，语法错误，这是只要加上prefixOverride即可,把首个“and”去掉

```xml
<select id="getUser" resultType="user">
	select * from user_tab 
	<trim prefix="where" prefixOverrides="and">
		<if test="lastName != null">
			last_name=#{lastName}
		</if>
		<if test="age != null">
			and age=#{age}
		</if>
		<if test="phone != null">
			and phone=#{phone}
		</if>
	</trim>
</select>
1234567891011121314
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018121122185231.png)
如果是先要去掉and获取or，则必须这样写**prefixOverrides=“and |or”**,注意中间的空格。

1. **suffix、suffixOverrides**
   上面查找，这里来看更新

```java
// 根据id来更新
public void updateUser(@Param("user") User user);
12
```

对应的xml

```xml
<!--普通版-->
<update id="updateUser">
	update user_tab
	set last_name=#{lastName},age=#{age},phone=#{phone}
	where id=#{id}
</update>
123456
```

使用suffix，并加上条件判断

```xml
<update id="updateUser">
	<trim suffix="where id=#{id}">
		update user_tab
		set
		<if test="lastName != null">
			last_name=#{lastName},
		</if>
		<if test="age != null">
			age=#{age},
		</if>
		<if test="phone != null">
			phone=#{phone}
		</if> 
	</trim>
</update>
123456789101112131415
```

执行输出跟上面的普通版语句一样没问题，可以看到suffix的效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181211223143532.png)
同样的假如phone为null了，

```java
// 更新
userMapper.updateUser(new User(2, "jack", 20, null));
12
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181211223736266.png)
因为最会一个if条件不成立，所以倒数第二个if里面的逗号在where的左侧语法错误，此时使用suffixOverrides即可

```xml
<update id="updateUser">
	<trim suffix="where id=#{id}" suffixOverrides=",">
		update user_tab
		set
		<if test="lastName != null">
			last_name=#{lastName},
		</if>
		<if test="age != null">
			age=#{age},
		</if>
		<if test="phone != null">
			phone=#{phone}
		</if> 
	</trim>
</update>
123456789101112131415
```

再次执行输出，成功执行，这个就是suffixOverrides的作用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181211224104682.png)

好了，这个就是trim标签即属性的基本介绍，具体怎么使用看实际情况，上面只是作为演示。



**mybatis也可输入拼接好的sql，再执行：**

```java
**mybatis可以灵活动态拼接sql**，变量值可以灵活运用；

​	如：  select a from b where a.id in <foreach>...</foreach>

可以改写成字符串拼接的方式： 

​			拼接id：3','4','5...     此处拼接要把字符串最前和最后的一个单引号去除，因为，mybatis解析#{}的时候，会自动给它在外围拼接上单引号；

​			select a from b wher a.id in (#{id})



也可以先在service层拼接好sql，然后传给xml执行；

​	如：string sql = "select ...";    xml查询标签中直接写：${sql}
```

