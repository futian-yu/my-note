# **mysql语法及函数**

**1.ifnull（）函数**

```sql
SELECT
        IFNULL(t.REFUND_ID, 0)
        FROM
        klk_ar_settle_headers t
        WHERE
        t.REFUND_ID IN ('22222')
//此处返回为null
```

```sql
select  ifnull((SELECT
        t.REFUND_ID
        FROM
        klk_ar_settle_headers t
        WHERE
        t.REFUND_ID IN ('22222')),0) from dual
//此处返回为0
```



**2.mysql批量插入（单双重循环）**

​		DELIMITER 其实就是告诉[MySQL](http://lib.csdn.net/base/mysql)解释器，该段命令是否已经结束了，mysql是否可以执行了。默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，mysql将会执行该命令。

```sql
DROP PROCEDURE IF EXISTS proc_initData;--如果存在此存储过程则删掉
DELIMITER $
CREATE PROCEDURE proc_initData()
BEGIN
  DECLARE DTime DATETIME DEFAULT '2020-06-29';
  DECLARE d INT DEFAULT 1;
  DECLARE i INT DEFAULT 1;
  WHILE d<=2 DO
    WHILE i<=1000 DO
      INSERT INTO act_evt_log(type_,lock_time_) VALUES("test12",DTime);
      SET i = i+1;
    END WHILE;
    SET DTime = DATE_ADD(DTime,INTERVAL 1 DAY);
    SET d = d+1;
    SET i = 1;
  END WHILE; 
END $
CALL proc_initData();
```

**3.mysql常用语法**

```sql
--	3.0 =========数据库操作=========
	-- 查看数据库
    SHOW DATABASES;
    -- 创建数据库
    CREATE DATABASE db_name;
    -- 使用数据库
    USE db_name;
    -- 删除数据库
	DROP DATABASE db_name;

--	3.1 =========表复制=========   
   -- 复制表结构及数据到新表 
    create table 新表
    select * from 旧表 

    -- 只复制表结构到新表（此种方式主键、索引等未复制）
    create table 新表
    select * from 旧表 where 1=2
    
    -- 只复制表结构到新表（主键、索引等也一起复制）
    create table 新表 like 旧表

    -- 复制旧表的数据到新表（假设两个表结构一样）
    insert into 新表
    select * from 旧表 

    -- 复制旧表的数据到新表（假设两个表结构不一样）
    insert into 新表(字段1,字段2,…….)
    select 字段1,字段2,…… from 旧表
    
    -- 删除表数据（表不删）
    delete from table_name [where...]
    -- 删除整张表(不可恢复)
    drop table table_name;
--	3.2 =========其他========= 
	-- 表重命名
	RENAME TABLE name_old TO name_new;	
	-- 增加字段
	ALTER TABLE tb_name ADD COLUMN address varchar(80) NOT NULL;
	-- 插入数据
    INSERT INTO tb_name(id,name,score)VALUES(NULL,'张三',140),(NULL,'张四',178),(NULL,'张五',134);
    -- 插入检索出来的数据
	INSERT INTO tb_name(name,score) SELECT name,score FROM tb_name2;
	-- 删除数据
	DELETE FROM tb_name WHERE id=3;



```

**4.SQL之case when then用法**

（1）case具有两种格式。简单case函数和case搜索函数。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```sql
--简单case函数
case sex
  when '1' then '男'
  when '2' then '女’
  else '其他' end
--case搜索函数
case when sex = '1' then '男'
     when sex = '2' then '女'
     else '其他' end  
-- 注意：可以不写else但是一定要有end ，没有else如果不满足条件会以null填充
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这两种方式，可以实现相同的功能。简单case函数的写法相对比较简洁，但是和case搜索函数相比，功能方面会有些限制，比如写判定式。

还有一个需要注重的问题，case函数只返回第一个符合条件的值，剩下的case部分将会被自动忽略。

```sql
--比如说，下面这段sql，你永远无法得到“第二类”这个结果
case when col_1 in ('a','b') then '第一类'
     when col_1 in ('a') then '第二类'
     else '其他' end  
```

（2）将sum与case结合使用，可以实现分段统计。

```sql
SQL> select
  2    sum(case u.sex when 1 then 1 else 0 end)男性,
  3    sum(case u.sex when 2 then 1 else 0 end)女性,
  4    sum(case when u.sex <>1 and u.sex<>2 then 1 else 0 end)性别为空
  5  from users u;
 
        男性         女性       性别为空
---------- ---------- ----------
         3          2          0
```

**5.SQL中的GROUP BY用法示例**

###### 聚合函数和Group by

​		脑子分析的时候，把group by(a)的字段a放在第一列。

```
聚合函数： sql语言中一种特殊的函数:聚合函数，SUM, COUNT, MAX, MIN, AVG等。这些函数和其它函数的根本区别就是它们一般作用在多条记录上。SELECT SUM(population) FROM COUNTRY 这里的SUM作用在所有返回记录的population字段上，结果就是该查询只返回一个结果，即所有国家的总人口数。通过使用GROUP BY 子句，可以让SUM 和 COUNT 这些函数对属于一组的数据起作用。当你指定 GROUP BY region 时， 属于同一个region(地区)的一组数据将只能返回一行值，也就是说，表中所有除region(地区)外的字段【被Group by指定的列可以返回多个值外】，只能通过 SUM, COUNT等聚合函数运算后返回一个值。HAVING子句可以筛选成组后的各组数据，WHERE子句在聚合前先筛选记录.也就是说作用在GROUP BY 子句和HAVING子句前。而 HAVING子句在聚合后对组记录进行筛选。二、例子：一）显示每个地区的总人口数和总面积: SELECT region, SUM(population), SUM(area)FROM COUNTRYGROUP BY region 
```

| region | Population | are     |
| ------ | ---------- | ------- |
| 上海   | 1080       | 12000km |
| 广州   | 2123       | 18000km |

先以region把返回记录分成多个组，这就是GROUP BY的字面含义。分完组后，然后用聚合函数对每组中的不同字段(一或多条记录)作运算。
[总人口数](https://www.baidu.com/s?wd=总人口数&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3nWn4nWDsrjuhrHFWm1--0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm3rHbkPWDd)

- **group by 用法记录**

  ```sql
  -- 脚本
  CREATE TABLE `employee` (
    `id` int NOT NULL,
    `name` varchar(20) DEFAULT NULL,
    `salary` int DEFAULT NULL,
    `departmentid` int DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
  
  INSERT INTO `employee`(`id`, `name`, `salary`, `departmentid`) VALUES (1, 'Joe', 70000, 1);
  INSERT INTO `employee`(`id`, `name`, `salary`, `departmentid`) VALUES (2, 'Henry', 80000, 2);
  INSERT INTO `employee`(`id`, `name`, `salary`, `departmentid`) VALUES (3, 'Sam', 60000, 2);
  INSERT INTO `employee`(`id`, `name`, `salary`, `departmentid`) VALUES (4, 'Max', 90000, 1);
  INSERT INTO `employee`(`id`, `name`, `salary`, `departmentid`) VALUES (5, 'JIM', 90000, 1);
  select * from EMPLOYEE;
  
  -- 起因：想查询各个部门最大工资以及最高工资所对应人的名字
  
  -- 经过：所写的sql如下，最大值是统计出来了，但是name是错的。(其实查询非group by中的字段，mysql版本低一点还会报错)
  
  SELECT name,departmentid,max(salary) 
        FROM EMPLOYEE
        GROUP BY departmentid;
  			
  -- 总结及解决：group只是用来做分组用的，select查询的字段只能是group by 的字段以及聚合函数，否则编译器无法确定非分组字段选择哪个。
  -- 查询所有部门中最大工资可以用其他方式查询.
  select * from EMPLOYEE where salary = (select max(salary) from employee);
  
  -- 查询每个部门中最大工资以及拿最高工资的人的姓名
  select e.name,p.departmentid,p.salary from (select departmentid,max(salary) salary from employee group by departmentid) p left join employee e
   on e.salary = p.salary
  ```

  

三） 查询每个部门的每种职位的雇员数。select deptno,job,count(*) from emp group by deptno,job。







[sql语句](https://www.baidu.com/s?wd=sql语句&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3nWn4nWDsrjuhrHFWm1--0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm3rHbkPWDd)

```




```

表的设计：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061418535166.png)

表里面的内容：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061418543074.png)

 

一：在不使用聚合函数的时候，group by 子句中必须包含所有的列，否则会报错，如下

```
select name,MON from [测试、] group by name
```

会报错：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061418564626.png)

在子句中加上所有的列的时候

```
select name,MON from [测试、] group by name,mon
```

这时候不报错，执行结果

![img](http://pic002.cnblogs.com/images/2012/290564/2012061418584565.png)

此时虽然成功执行了，但是可以看出来group by在这里并没有发挥任何的作用，我们完全可以直接select而不用group by，所以，group by子句要配合聚合函数使用，并且，在配合聚合函数使用的时候，在group by子句中不要加上聚合函数处的列名（加入as了的话）

配合聚合函数使用了的情况：

```
select SUM(mon),name from [测试、] group by name
```

结果：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061419023427.png)

是我们想要的。

再如：

```
select MAX(ID),name from [测试、] group by name
```

结果：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061419034810.png)

也是我们想要的。

 

但是说如果这样的话：

```
select MAX(ID) as id,name from [测试、] group by name,id
```

得到的结果是：

![img](http://pic002.cnblogs.com/images/2012/290564/2012061419052658.png)

很明显这不是我们想要的。

所以这时候在使用聚合函数的地方若是使用了as另外命名，请不要在group by子句后再加上那个令命名的名字，否则就和文章刚开始出现的情况一样.

**6.mysql exists 用法；**

select * from A where A.id in(select B.id from B where A.id = B.id);=>用exists代替

select * from A where exists (select * from B where A.id = B.id);

​	exists返回的是逻辑上的true与false；

所以，select * from A exists (select 1);返回的是A表所有值；

**7.查询场景**；只查询出，头金额减去对应所有行金额汇总 != 0 的所有行；

​		这里用了一个临时表来处理不等于0；

```sql
SELECT
        PAYMENT_HEADER_ID,
        REFUND_ID,
        ORDER_GUID,
        CUSTOMER_CODE,
        PAYMENT_COMPANY,
        CURRENCY_CODE,
        PAYMENT_AMOUNT,
        CONVERSION_AMOUNT,
        CURRENCY_ID
        FROM
        (
        SELECT
        haph.PAYMENT_HEADER_ID,
        haph.REFUND_FLOW_ID REFUND_ID,
        haph.RELATED_SO ORDER_GUID,
        haph.CUSTOMER_CODE,
        fcb.COMPANY_FULL_NAME PAYMENT_COMPANY,
        haph.CURRENCY_CODE,
        haph.PAYMENT_AMOUNT,
        haph.PAYMENT_AMOUNT CONVERSION_AMOUNT,
        haph.CURRENCY_ID,
        SUM( hacpi.PAYMENT_AMOUNT ) SUM_PAYMENT_AMOUNT,
        ( haph.PAYMENT_AMOUNT - SUM( hacpi.PAYMENT_AMOUNT ) ) CONVERSION_PAYMENT_AMOUNT
        FROM
        hscs_ar_payment_headers haph
        LEFT JOIN hscs_ar_act_payment_inf hacpi ON haph.PAYMENT_HEADER_ID = hacpi.PAYMENT_HEADER_ID
        LEFT JOIN fnd_company_b fcb ON fcb.company_code = haph.payment_company
        <where>
            <if test="refundFlowId != null and refundFlowId != ''">
                REFUND_FLOW_ID = #{refundFlowId}
            </if>
            <if test="relatedSo != null and relatedSo != ''">
                and RELATED_SO = #{relatedSo}
            </if>
            <if test="paymentCompany != null and paymentCompany != ''">
                and PAYMENT_COMPANY = #{paymentCompany}
            </if>
            <if test="customerCode != null and customerCode != ''">
                and CUSTOMER_CODE = #{customerCode}
            </if>
            <if test="accountingDate != null">
                and #{accountingDate} >= haph.ACCOUNTING_DATE
            </if>
        </where>
        GROUP BY
        haph.PAYMENT_HEADER_ID
        ) table_a
        WHERE
        CONVERSION_PAYMENT_AMOUNT != 0
```

8.**使用case when批量更新**

```sql
**
-- 更新多个字段(记得加上条件，否则会更新所有)
UPDATE hscs_itf_imp_interfaces 
SET VALIDATE_STATUS = 'S',
STATUS_CODE = 'N',
PROCESS_MESSAGE = '',
VALUE4 = ( CASE VALUE4 WHEN NULL THEN 'DEFAULT' END ),
value5 = ( CASE value5 WHEN NULL THEN 'DEFAULT' END ),
value14 = ( CASE value14 WHEN NULL THEN 'DEFAULT' END ),
VALUE15 = ( CASE VALUE15 WHEN NULL THEN 'DEFAULT' END ),
value17 = ( CASE value17 WHEN NULL THEN 'DEFAULT' END ),
value18 = ( CASE value18 WHEN NULL THEN 'DEFAULT' END ) 
WHERE
 INTERFACE_NAME = 'KLK_ITEM_IF' 
 AND STATUS_CODE IN ( 'E', 'U' )

-- 更新一个字段(记得加上条件，否则会更新所有)
UPDATE hscs_ar_payment_headers 
SET ACCOUNTING_DATE = '2020-09-09',
EXRATE_DATE = '2020-09-09',
EXCHANGE_RATE =
CASE
	payment_header_id 
when -1252	then 1.0000000000
when -1249	then 1.0000000000
end
where payment_header_id in(-1252,-1249)


```

**mysql数据导出**

```sql
导出mysql表(大数据量时处理速度快)自动分批命令：

​	mysqldump -uroot -proot -h 127.0.0.1  hap_db0 hscs_pub_exrate_values>db_date1.sql
```

