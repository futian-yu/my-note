#### mybatis中foreach标签详解

foreach的主要用在构建in条件中，它可以在SQL语句中进行迭代一个集合。

foreach元素的属性主要有 item，index，collection，open，separator，close。

item表示集合中每一个元素进行迭代时的别名，
index指 定一个名字，用于表示在迭代过程中，每次迭代到的位置，
open表示该语句以什么开始，
separator表示在每次进行迭代之间以什么符号作为分隔 符，
close表示以什么结束。


在使用foreach的时候最关键的也是最容易出错的就是collection属性，该属性是必须指定的，但是在不同情况 下，该属性的值是不一样的，主要有一下3种情况：

1. 如果传入的是单参数且参数类型是一个List的时候，collection属性值为list

2. 如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array

3. 如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，当然单参数也可

4. 如果不想用上面三种默认的话，可以在Mapper接口参数前加上@Parma()注解 改变属性值；例如

   ```java
   public List<String> queryExistCodeInCode(@Param("codes") List<String> codes);
   //由此衍伸出，如果一个dto对象里有List属性，传入dto后，也可以直接以dto.list进行循环，无需重新开辟传入一个list；
   ```

   

以封装成map，实际上如果你在传入参数的时候，在breast里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key 下面分别来看看上述三种情况的示例代码：



1.单参数List 类型

<select id="dynamicForeachTest" resultType="Blog">
           select * from t_blog where id in
        <foreach collection="list" index="index" item="item" open="(" separator="," close=")">
                #{item}       
        </foreach>    
</select>

上述collection的值为list，对应的Mapper是这样的
public List dynamicForeachTest(List ids);
测试代码：

```java
@Test
     public void dynamicForeachTest() {
         SqlSession session = Util.getSqlSessionFactory().openSession();      
         BlogMapper blogMapper = session.getMapper(BlogMapper.class);
          List ids = new ArrayList();
          ids.add(1);
          ids.add(3);
           ids.add(6);
         List blogs = blogMapper.dynamicForeachTest(ids);
          for (Blog blog : blogs)
              System.out.println(blog);
          session.close();
      }
```


2.单参数array数组的类型：
<select id="dynamicForeach2Test" resultType="Blog">
     select * from t_blog where id in
     <foreach collection="array" index="index" item="item" open="(" separator="," close=")">
          #{item}
     </foreach>
</select>   

上述collection为array，对应的Mapper代码：
public List dynamicForeach2Test(int[] ids);
对应的测试代码：

```java
@Test
 public void dynamicForeach2Test() {
         SqlSession session = Util.getSqlSessionFactory().openSession();
         BlogMapper blogMapper = session.getMapper(BlogMapper.class);
         int[] ids = new int[] {1,3,6,9};
         List blogs = blogMapper.dynamicForeach2Test(ids);
         for (Blog blog : blogs)
         System.out.println(blog);    
         session.close();
 }
```


3.自己把参数封装成Map的类型
<select id="dynamicForeach3Test" resultType="Blog">
         select * from t_blog where title like "%"#{title}"%" and id in
          <foreach collection="ids" index="index" item="item" open="(" separator="," close=")">
               #{item}
          </foreach>
 </select>
上述collection的值为ids，是传入的参数Map的key，对应的Mapper代码：
public List dynamicForeach3Test(Map params);
对应测试代码：

```java
@Test
    public void dynamicForeach3Test() {
        SqlSession session = Util.getSqlSessionFactory().openSession();
         BlogMapper blogMapper = session.getMapper(BlogMapper.class);
          final List ids = new ArrayList();
          ids.add(1);
          ids.add(2);
          ids.add(3);
          ids.add(6);
         ids.add(7);
         ids.add(9);
        Map params = new HashMap();
         params.put("ids", ids);
         params.put("title", "中国");
        List blogs = blogMapper.dynamicForeach3Test(params);
         for (Blog blog : blogs)
             System.out.println(blog);
         session.close();
     }
```

————————————————



**2.Mybatis的mapper xml文件中的常用标签。**

一、SQL语句标签：

1、<!--查询语句-->  

<select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.String" >
	select   
</select> 

2、<!--插入语句-->  



```xml
<insert id="insert" parameterType="pojo.OrderTable" >  
 insert into ordertable (order_id, cid, address,   
create_date, orderitem_id)  

values (#{orderId,jdbcType=VARCHAR},#{cid,jdbcType=VARCHAR}, #{address,jdbcType=VARCHAR},   

 #{createDate,jdbcType=TIMESTAMP}, #{orderitemId,jdbcType=VARCHAR})
 </insert> 
```



3、<!--删除语句-->  

```xml
<delete id="deleteByPrimaryKey" parameterType="java.lang.String" >
delete from ordertable  
where order_id = #{orderId,jdbcType=VARCHAR}  
</delete>  
```



4、<!--修改语句-->  



```xml
 <update id="updateByPrimaryKey" parameterType="pojo.OrderTable" >  
  update ordertable  
set cid = #{cid,jdbcType=VARCHAR},  

  address = #{address,jdbcType=VARCHAR},  

  create_date = #{createDate,jdbcType=TIMESTAMP},  

  orderitem_id = #{orderitemId,jdbcType=VARCHAR}  

where order_id = #{orderId,jdbcType=VARCHAR} 
  </update> 
```



1》需要配置的属性：

★id="xxxx" ——表示此段sql执行语句的唯一标识，也是接口的方法名称【必须一致才能找到】

★parameterType="xxxx" ——表示该sql语句中需要传入的参数， 类型要与对应的接口方法的类型一致【可选】

★resultMap="xxx"—— 定义出参，调用已定义的<resultMap>映射管理器的id值

★resultType="xxxx"——定义出参，匹配普通Java类型或自定义的pojo【出参类型若不指定，将为语句类型默认类型，如<insert>语句返回值为int】

2》注意： 至于为何<insert><delete><update> 语句的返回值类型为什么是int，有过JDBC操作经验的朋友可能会有印象，增删改操作实际上返回的是操作的条数。而Mybatis框架本身是基于JDBC的，所以此处也沿袭这种返回值类型。

3》传参和取值：mapper.xml 的灵活性还体现在SQL执行语句可以传参，参数类型通过parameterType= "" 定义

★取值方式1：#{value jdbcType = valuetype}：jdbcType 表示该属性的数据类型在数据库中对应的类型，如 #{user jdbcType=varchar} 等价于 String username；

★取值方式2：${value } : 这种方式不建议大量使用，可能会发送sql注入而导致安全性问题。一般该取值方式可用在非经常变化的值上，如orderby ${columnName}；

二、sql片段标签<sql>：通过该标签可定义能复用的sql语句片段，在执行sql语句标签中直接引用即可。这样既可以提高编码效率，还能有效简化代码，提高可读性

```
1》 需要配置的属性：id="" ———表示需要改sql语句片段的唯一标识

 2》引用：通过<include refid="" />标签引用，refid="" 中的值指向需要引用的<sql>中的id=""属性
```

1、<!--定义sql片段-->  

```xml
<sql id="orderAndItem"> 
o.order_id,o.cid,o.address,o.create_date,o.orderitem_id,i.orderitem_id,i.product_id,i.count  
  </sql>  
```

2、<!--引用sql片段-->  

```xml
select
<include refid="orderAndItem" />  

from ordertable o  

join orderitem i on o.orderitem_id = i.orderitem_id  

where o.order_id = #{orderId}  
```



 

三、映射管理器resultMap：映射管理器，是Mybatis中最强大的工具，使用其可以进行实体类之间的关系，并管理结果和实体类间的映射关系

1》需要配置的属性：<resultMap id="" type=""></resutlMap>

  id=""——表示这个映射管理器的唯一标识，外部通过该值引用；

type ="" ——表示需要映射的实体类；

2》 需要配置的参数：<id column = "" property= "" />

  <id>标签指的是：结果集中结果唯一的列【column】 和 实体属性【property】的映射关系，

3》注意：<id>标签管理的列未必是主键列，需要根据具体需求指定；

```
<result column= " " property=" " />  <result>标签指的是：结果集中普通列【column】 和 实体属性【property】的映射关系；
```

 4》 需要维护的关系：所谓关系维护是值在主表查询时将其关联子表的结果也查询出来

1）一对一关系

```java
<assocation property = " " javaType=" ">  

property = “ ”→ 被维护实体在宿主实体中的属性名，

javaType = " "→ 被维护实体的类型

Orderitem.java

package pojo;   

public class Orderitem {  
private String orderitemId;  

private String productId;  

private Integer count;    

private Product product;     
```

从上方代码段可以看出：Product 对象在 Orderitem 实体中以 product 属性存在 

Orderitemmapper.xml

```xml
<resultMap id="BaseResultMap" type="pojo.Orderitem" >  

   <id column="orderitem_id" property="orderitemId" jdbcType="VARCHAR" />  

   <result column="product_id" property="productId" jdbcType="VARCHAR" />  

   <result column="count" property="count" jdbcType="INTEGER" />  
```

  ★ <!-- 通过association 维护 一对一关系 -->  

  

```xml
 <association property="product" javaType="pojo.Product">  
<id column="product_id" property="productId"/>  

<result column="product_factroy" property="productFactroy"/>  

<result column="product_store" property="productStore"/>  

<result column="product_descript" property="productDescript"/>  
   </association>  

 </resultMap>  
```



通过xml的配置可以看出，在resultMap映射管理器中，通过<association> 进行了维护，也就是在查询Orderitem对象时，可以把关联的Product对象的信息也查询出来

2）一对多关系的维护

<collection property=" " ofType=" "> 

property = “”→ 被维护实体在宿主实体中的属性名

ofType=“ ”→是被维护方在宿主类中集合泛型限定类型

【由于在一对多关系中，多的一放是以List形式存在，因此ofType的值取用Lsit<?> 的泛型对象类型】



```java
OrderTable.java

public class OrderTable {  
private String orderId;  

private String cid;  

private String address;  

private Date createDate;  

private String orderitemId;   

private List<Orderitem> orderitemList ; 
}  
```





```xml
OrderTableMapper.xml;

<resultMap id="BaseResultMap" type="pojo.OrderTable" >  

   <!--  
WARNING - @mbggenerated  

 This element is automatically generated by MyBatis Generator, do not modify.  

This element was generated on Fri May 06 15:49:42 CST 2016.
   -->  

   <id column="order_id" property="orderId" jdbcType="VARCHAR" />  

   <result column="cid" property="cid" jdbcType="VARCHAR" />  

   <result column="address" property="address" jdbcType="VARCHAR" />  

   <result column="create_date" property="createDate" jdbcType="TIMESTAMP" />  

   <result column="orderitem_id" property="orderitemId" jdbcType="VARCHAR" />  
```



  ★ <!--维护一对多的关系  -->  

```xml
<collection property="orderitemList" ofType="pojo.Orderitem">  

    <id column="orderitem_id" property="orderitemId"/>  

    <result column="product_id" property="productId"/>  

    <result column="count" property="count"/>  

</collection>   
 </resultMap>  
```



5》在resultMap 中需要注意两点：

关联关系的维护可以根据实体类之间的实际情况进行嵌套维护



```xml
<resultMap id="BaseResultMap" type="pojo.OrderTable" >  
<id column="order_id" property="orderId" jdbcType="VARCHAR" />  

<result column="cid" property="cid" jdbcType="VARCHAR" />  

<result column="address" property="address" jdbcType="VARCHAR" />  

<result column="create_date" property="createDate" jdbcType="TIMESTAMP" />  

<result column="orderitem_id" property="orderitemId" jdbcType="VARCHAR" />  
```

   ★<!--维护一对多的关系  -->  

```xml
    <collection property="orderitemList" ofType="pojo.Orderitem">  

        <id column="orderitem_id" property="orderitemId"/>  

        <result column="product_id" property="productId"/>  

        <result column="count" property="count"/>  
```

<span style="white-space:pre">        </span><!--嵌套一对一关系-->  

```xml
        <association property="customer" javaType="pojo.Customer">  

            <id column="cid" property="cid"/>  

            <result column="cname" property="cname"/>  

        </association>  

    </collection>   
  </resultMap>  
```



关于出现重复列名的处理：在实际操作过程中，查询到的结果可能会出现相同的列名，这样会对映射到实体属性带来影响甚至出现报错，那么对待这个问题可以通过对列取别名的方式处理

 

四：常用的动态语句标签：通过动态sql标签可以进行条件判断，条件遍历等操作从而满足结果的需要

 1》<where> ： 使用其可以代替sql语句中的where关键字，一般防止在条件查询的最外层

 2》<if >：条件判断标签，配置属性test=" 条件字符串 "，判断是否满足条件，满足则执行，不满足则跳过

<select id="findOrderItemDetail" parameterType="pojo.Orderitem" resultMap="BaseResultMap">  

```xml
    select orderitem.orderitem_id,product.*   

    from orderitem,product  

    <where>  

        <if test="orderitemId!=null and orderitemId!=''">  

            and orderitem.orderitem_id = #{orderitemId}  

        </if>  

        <if test="productId!=null and productId!=''">  

            and orderitem.product_id = #{productId}  

        </if>  

        <if test="count!=null">  

            and orderitem.count = #{count}  

        </if>  

    </where>  
  </select>  
```



3》<set>：常用于<update>更新语句中，替代 sql中的“set”关键字，特别是在联合<if>进行判断是，可以有效方式当某个参数为空或者不合法是错误的更新到数据库中

```xml
<update id="updateByPrimaryKeySelective" parameterType="pojo.Orderitem" >  

   update orderitem  

   <set >  
<if test="productId != null" >  

   product_id = #{productId,jdbcType=VARCHAR},  

 </if>  

 <if test="count != null" >  

   count = #{count,jdbcType=INTEGER},  

 </if>  
    </set>  

   where orderitem_id = #{orderitemId,jdbcType=VARCHAR}  

 </update>  
```



4》<choose><when></when><otherwise></otherwise>

</choose> 标签组：也是一个用于条件判断的标签组，和<if>的不同之处在于条件从<choose>进入，去匹配<when>中的添加，一旦匹配马上结束；若到找不到匹配项，将执行<other>中的语句；可以理解为<if>是 && 关系 <choose>是 || 关系

<!-- 查询学生list，like姓名、或=性别、或=生日、或=班级，使用choose -->       

<select id="getStudentListChooseEntity" parameterType="StudentEntity" resultMap="studentResultMap">       

```xml
SELECT * from STUDENT_TBL ST        

<where>       

    <choose>       

        <when test="studentName!=null and studentName!='' ">       

                ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')        

        </when>       

        <when test="studentSex!= null and studentSex!= '' ">       

                AND ST.STUDENT_SEX = #{studentSex}        

        </when>       

        <when test="studentBirthday!=null">       

            AND ST.STUDENT_BIRTHDAY = #{studentBirthday}      

        </when>       

        <when test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' ">       

            AND ST.CLASS_ID = #{classEntity.classID}        

       </when>       

       <otherwise>                      

       </otherwise>       

    </choose>       

</where>  
</select>     
```

5》<foreach>标签：该标签的作用是遍历集合类型的条件 

  foreach元素的属性主要有 item，index，collection，open，separator，close。

```
    item表示集合中每一个元素进行迭代时的别名.

    index指 定一个名字，用于表示在迭代过程中，每次迭代到的位置.

    open表示该语句以什么开始，separator表示在每次进行迭代之间以什么符号作为分隔 符.

    close表示以什么结束
```

用来循环 collection : 用来指定循环的数据的类型 可以填的值有：array,list,map item   



```xml
 <delete id="deleteByPriKeys" parameterType="java.lang.String">  
 delete from product where product_Id in  

 <foreach collection="list" item="productId" open="(" separator="," close=")">  

     #{productId,jdbcType = VARCHAR}  

 </foreach>  
  </delete>  
```

————————————————

