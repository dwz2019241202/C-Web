## MySQL常用指令

1. **cmd登录： mysql -uroot -p201911808359X**

2. **查看mysql中所有数据库：show databases;**

3. **选择使用test数据库：use test;**

4. **创建bjpowernode数据库：create database bjpowernode;**

5. 数据库当中最基本的元素是表：table

6. 任何一个表都有行列：行（被称为数据/记录） 列（被称为字段）

7. 每一个字段都有：字段名、数据类型、约束等属性

8. **查看某个数据库下有哪些表：show tables;**

9. 注意：以上命令不区分大小写

10. 关于SQL语句的分类：

    ​	DQL: 数据查询语言（凡是带有select关键字的都是查询语句）

    ​			select ......

    ​	DML:数据操作语言（凡是对表中的数据进行增删改的都是DML）

    ​			insert（增） delete（删） updata（改）

    ​	DDL:数据定义语言（凡是带有create增、drop删、alter改的都是DDL）

    ​			**DDL主要操作的是表的结构。不是表中的数据**

    ​	TCL:事务控制语言，包括：事务提交commit    事务回滚rollback

    ​	DCL:数据控制语言，例如：授权grant、 撤销授权revoke

11. **注意：mysql不见' ;  '不执行，分号是结束符，\c 用来终止输入**

12. **查询表中数据:   select * from 表名（表所有字段数据）这种方式效率低，可读性差，在实际开发中不建议**

    ​							**select a,b,c,d（所有字段名） from 表名（查询所有字段数据，实际开发中使用）**

    ​						   **select 字段名 from 表名;（查询一个字段）**

    ​							**select 字段名1, 字段名2 from 表名 （查询多个字段）**

13. **给查询的列起别名： select 字段名 as 别名 from 表名** 

    ​									as 可以省略，select deptno, dname deptname from dept;（dname 重命名为deptname）

    ​									**假设起别名时，别名里面有空格， select deptno， dname dept name from dept; 这样的语句不符合语法规范，编译报错，解决方法是加单引号， select deptno, dname 'dept name' from dept;** 

    ​									注意：在所有数据库中，字符串统一使用单引号括起来，单引号是标准，很通用

14. 查询时可以对输出进行数据处理：

    ​		**select ename, sal*12 as yearsal from emp;**         //sal is month salary, the type of sal is 		int which can be calculated  

    ​		可以对输出的数据进行加减乘除运算，但是不改变原表的数据值

15. 条件查询：将表中符合条件的查询出来

    ​		   	**select**

    ​						**字段1，字段2，字段3,......**

    ​				**from**

    ​						**表名**

    ​				**where**

    ​						**条件；**

    ​		条件：	

    ​						等于=：select ename, sal from emp where sal = 800;

    ​							  select ename, sal from emp where ename = ‘SMITH’;      // 查询名字为SMITH

    ​					   不等号! =  or   <>：select ename, sal from emp where sal != 800;

    ​														select ename, sal from emp where sal <> 800;

    ​						**between ... and ... 两个值之间（必须遵循左小右大的原则）**，等同于 >=  and <=

    ​						is null 为空， is not null 不为空，**注意：在数据库当中null不能使用等号进行衡量。需要使用 is null**

    ​						and 并且   or或者，注意：and 比 or 优先级高， and先执行，or后执行，and 和 or同时出现最好用括号进行约束

    ​						**in 包含，相当多个or； not in 表示不在这个范围中**： select ename, job from emp where job in('Alibaba', 'Tecent')    //        						查询工作在阿里巴巴或者腾讯的人

    ​																													 select ename, sal from emp where sal in(800, 1000, 2000)    //        						查询工资为800或1000或2000的人

    ​						like：模糊查询   select ename from emp where ename like '%o%'   //查询名字中有o的

    

16. 查看mysql数据库版本号： select version

17. 查询表的结构：desc 表名  （desc = describe描述）

18. 排序

    查询所有员工薪资，排序

    select

    ​	ename, sal

    from 

    ​	emp

    order by

    ​	sal   //按薪资排序，默认指定升序(**asc**可以不写)

    

    select

    ​	ename, sal

    from 

    ​	emp

    order by

    ​	sal  **desc**   //按薪资排序,指定降序

    

    查询员工名字和薪资，要求按薪资升序，如果薪资一样再按名字升序

    select ename, sal from emp order by sal asc, ename asc;   //sal在前，先按sal排序，然后再按ename排序

    

    根据字段位置也可以排序

    select eanme sal from emp order by 2;   //2 是指第二列，第二列是sal

    了解一下，不建议在开发中使用，因为不建壮，列的顺序很容易发生改变，列顺序改变之后，2就无意义了

    

    **找出工资在1200 到 2000 之间，按降序排列**

    **select ename, sal from emp where sal between 1200 and 2000 order by  sal desc**    **//注意不能改变关键字位置 order by应该在where 之后**

    **以上语句执行顺序必须掌握：**

    ​			**第一步：from**

    ​			**第二步：where**

    ​			**第三步：select**

    ​			**第四步：order by**

19. 数据处理函数（又称单行处理函数，与之相对的多行处理函数）

    单行处理函数：一个输入对应一个输出，对每一行的数据进行处理

    多行处理函数：多行输入对应一个输出，例如求所有人的工资总和

20. | **lower**                                        | **转换小写：select Name from country where lower(Name) = 'china'** |
    | ------------------------------------------------ | ------------------------------------------------------------ |
    | upper                                            | 转换大写：select lower(Name) as Name from country //输出每行Name的大写 |
    | substr                                           | 取子串（**substr(被截取的字符串，起始下标，截取的长度)**），**注意：起始下标从1开始，没有0** |
    | length                                           | 取长度:length(String)                                        |
    | trim                                             | 去空格                                                       |
    | concat                                           | 字符串拼接：concat(String1, String2)                         |
    | str_to_data                                      | 将字符串转换为日期                                           |
    | data_format                                      | 格式化日期                                                   |
    | format                                           | 设置千分位                                                   |
    | round                                            | 四舍五入: **round(数值类数据，保留多少位小数可以为负数)**   //负数即为保留到小数点之前多少位 |
    | rand                                             | 生成随机数                                                   |
    | ifnull                                           | 可以将null转化成一个具体的值，函数用法：ifnull（数据，被当作哪个值）      注意：null只要参与运算，最终结果一定是null |
    | case ... when ... then ... when ... then ... end | 例子：当员工为MANAGER时，工资上涨10%；当员工为SALESMAN时，工资上涨50%；其他正常  select ename,job,sal as oldsal, (case job when 'MANAGER' then sal*1.1 when 'SALESMAM' then sal*1.5 else sal end) as newsal |
    |                                                  |                                                              |

21. 分组函数（多行处理函数）

    多行处理函数：输入多行，最终输出一行

    | max   | 最大值   |
    | ----- | -------- |
    | min   | 最小值   |
    | sum   | 求和     |
    | avg   | 平均值   |
    | count | 计算数量 |

22. 分组函数自动忽略null，不需要对null进行提前处理

23. count（具体字段）：表示统计该字段下所有不为null的元素个数

    count（*）：表示统计表当中的总行数（只要有一行数据，count++）

    因为每一行数据不可能都是null，一行数据中有一列不为null，那么这行数据就是有效的

24. 分组函数不能直接使用在where子句中

    select ename, sal from emp where sal > min(sal)

    这样写会报错。？？？？？？？？？？？？？？？？

    ### 分组查询（*********）

    **在实际应用中，可能要先进行分组，然后对每一组数组进行操作，这个时候就要用到分组查询**

    **select ...... from ...... group by ......** 

    **select ......  from ...... where ...... group by ...... order by ......**

    **执行顺序：1.from     2.where    3.group by   4.select    5.order by**

    

    **select ename,sal from emp where sal > min(sal);   //报错**

    **因为分组函数只能在分组之后才能使用，而 where 在 group by 之前**

    **select ename, min(sal) from emp   //可以使用**

    **因为select在 group by之后，虽然并没有分组，于是默认整张表为一组**

    

    

    **找出每个工作岗位的工资和**

    **实现思路：先将表中记录按照工作岗位分组，然后求工资和**

    **select job, sum(sal) from emp group by job;    //成功**

    **select  ename,job,sum(sal) from emp group by job   //以上语句在MySQL中可以执行，但是没有意义；在oracle中执行会报错**

**重要结论：在一条select语句中，如果有group by 语句的话，select后面只能跟：参加分组的字段，以及分组函数。其他的一律不能跟**    

**找出不同部门、不同岗位的最高薪？**
联合分组：**select deptno,job,max(sal) from emp group by deptno,job;**



**Question：找出每个部门最高的薪资，要求显示大于3000的最高薪资**
方法一：having（注意：having不能单独使用，having不能代替where，having必须和group by联和使用）  select deptno,max(sal) from emp group by deptno having max(sal) > 3000;

方法二：先用where过滤所有大于3000的薪资，然后再分组

select deptno, max(sal) from emp where sal > 3000 group by deptno;

**方法一的效率比较低，where 和 having 优先选择where， where实在完成不了，再选择having**



**Question：找出每个部门的平均薪资，要求显示平均薪资高于2500（where无法替代having）**

select deptno, avg(sal) from emp group by deptno having avg(sal) >2500;

**总结：**

​	select ...... from ...... where ......group by ...... having ...... order by......

执行顺序： from  ->  where   ->   group by   ->   having   ->  select   ->    order 

by   (**注意：以上关键字只能按照这个顺序来**)



25. distinct去除重复记录（**原表数据不会被修改，只是查询结果去重**）

select distinct job from emp;  //查询job字段去重

select ename, distinct job from emp;   //这种写法是错误的，distinct只能放在所有字段的最前方

select dinstinct ename, job from emp;  //这种写法是正确的，取ename 与 job 联合的不同



26. **连接查询**

    select ename, dname **from emp, dept**;  //ename为员工名字段，dname为部门名字段，emp为员工表，dept为部门表，并没有加限制

    当两张表进行连接查询，**没有任何条件限制时**， 最终查询结果条数，是两张表条数的乘积，这种现象被称为笛卡尔积现象。

    

    select ename, dname **from emp, dept** where **emp.deptno = dept.deptno**;   //加以限制部门编号相同，避免出现笛卡尔积现象

    **加限制和不加限制相比匹配次数没有减少，只是选择性输出而已**

    

    ```
    **//表起别名，很重要，效率问题**
    **select e.name, d.name from emp e, dept d where e.deptno = d.deptno;    //emp表起别名e， dept表起别名d，**
    ```

    **注意：通过笛卡尔积现象得出表的连接次数越多，效率越低，尽力避免表的连接次数**

    

    

    **内连接（inner join）之等值连接**

    ```
    //sql92语法
    select e.name, d.name from emp e, dept d where e.deptno = d.deptno;
    
    //sql99语法
    select e.name, d.name from emp e inner join dept d on e.name = d.name;
    ```

     **sql92结构不够清晰，表的连接条件和后期进一步筛选，都放在了where里**

    **sql99表连接的条件是独立的，连接之后，如果还要进一步筛选，在往后继续添加where条件**

    ```
    //SQL99语法
    select ...... from a join b on 表a和表b的条件 where 进一步筛选的条件
    ```

    

    

    **内连接之非等值连接**

    **Question：找出每个员工的工资等级，要求显示员工的名字、薪资和薪资等级**

    ```
    select e.name, e.sal, s.grade from emp e inner join salgrade s on e.sal between s.losal and s.hisal;
    ```

    

    **内连接之自连接（将一张表看出那个两张表）**

     **Question：查询员工的领导，并输出员工名和领导名**

    技巧：先在emp表中获取员工的领导编号，再根据领导编号在emp表中查询领导名字

    ```
    select a.ename, b.ename from emp a inner join emp b on a.mgr = b.empno;  //将emp表看作a, b 两张表； a.mgr = b.empno 员工的领导编号就等于领导的员工编号
    ```

    

    **外连接（左右连接）**

    **在外链接中，两张表发生主次关系**

    **相同条件下外连接的查询结果数量>=内连接的查询结果数量**

    ```
    //右链接
    select e.ename d.dname from emp e right outer join dept d on e.deptno = d.deptno;  //right表示join右边的那张表为主表，将符合条件的两表中的内容输出，并且输出不符合条件的主表中的内容
    
    //左连接
    select e.ename d.dname from dept d left outer join emp e on e.deptno = d.deptno;
    ```

     

    **多表连接**

    多表连接中内连接和外连接可以混合

    ```
    select ... from a join b on a和b的连接条件 join c on a和c的连接条件 join d on a和d的连接条件    
    ```

27. **子查询**

    select 语句中嵌套select语句， 被嵌套的select语句被称为子查询

    ```
    select ..(select). from ..(select). where ..(select).
    ```

​		

**where子句中的子查询**

**Question：找出比最低工资高的员工姓名工资**

```
select ename, sal from emp where sal > (select min(sal) from emp);    //where子句中的子查询，牢记where后不能直接使用分组函数
```



**from子句中的子查询（注意：from后面的子查询，可以将子查询的查询结果当作一张临时表）**

**Question：找出每个岗位的平均工资的薪资等级**

```
select a.job, a.average, b.grade from (select job, avg(sal) as average from emp) a left join salgrade b on a.average between b.losal and b.hisal; 　
```



**where子句中的子查询(这个内容不需要掌握，了解即可)**



**union合并查询结果**

对于表连接来说，每连接一次表，则匹配次数成倍增长，但是union可以减少匹配次数，还可以拼接查询结果（生成临时表，以空间换时间）

union使用时，要求两个结果集的列数相同，且字段相同（在MySQL中列相同字段不同也能运行，但在Oracle中会报错）

**Question：查询工作岗位是MANAGER 和SALESMAN的员工；**

```
select　ename, job from emp where job = 'MANAGER' or job = 'SALESMAN'
select　ename, job from emp where job in ('MANAGER', 'SALESMAN')

//union实现效率更高，
select ename, job from emp where job = 'MANAGER' 
union
select ename, job from emp where job = 'SALESMAN' 
```





**limit：将查询结果的一部分取出来（例如取前五条记录）**

完整语法： **limit startIndex, length **

缺省语法：**limit 5 //取前五**

注意：**limit 在order by之后执行**

**Question：按照薪资降序，取排名前五的员工**

```
select ename, sal from emp order by sal desc limit 5;  //取排名前五的员工
```



28. **表的创建（建表）**

    建表的语法格式（建表属于DDL语句）

    表名：建议以t_ 或者 tbl_ 开始，可读性强。

    字段名：见名知意

    ```
    create table 表名(
    	字段名1 数据类型,
    	字段名2 数据类型,
    	字段名3 数据类型,
    )
    ```

29. **表的删除**

    ```
    drop table 表名   //当该表不存在时会报错哦
    drop table if exists 表名  //如果这张表存在，则删除
    ```

    

30. **数据类型**

    | varchar      | 可变长度的字符串（会根据实际的数据长度动态分配空间，但是速度慢） |
    | ------------ | ------------------------------------------------------------ |
    | **char**     | **定长字符串（分配固定长度的空间，使用不恰当可能导致空间的浪费）** |
    | **int**      | **数字中的整数型**                                           |
    | **bigint**   | **数字中的长整型，等同long**                                 |
    | **float**    | **单精度浮点数**                                             |
    | **double**   | **双精度浮点数**                                             |
    | **data**     | **短日期**                                                   |
    | **datatime** | **长日期**                                                   |
    | **clob**     | **字符大对象，最多可以存储4G的字符串**                       |
    | **blob**     | **二进制大对象，专门用来存储图片，声音，视频等流媒体文件**   |

    

31. **插入数据insert (DML)**

    语法格式：

    **注意：字段名要和值要一一对应（数据类型要相互对应）**

    ```
    insert into 表名(字段1,字段2,字段3,...) value(值1, 值2,值3,...)
    ```

    ```
    //一次插入多条记录
    insert into 表名(字段1,字段2,字段3,...) value(值1, 值2,值3,...), (...), (...)
    ```

    **注意：insert语句但凡执行成功那么会多出一条记录，没有给其他字段指定值的话，默认值是null，也可以设置默认值**

    ```
    create table if exists t_student(
    	no int,
    	name varchar(32),
    	sex char(1) default 'm'  //设置默认值为m
    );
    ```

    insert语句中的字段名可以缺省，但是要将所有字段的value填写完全

    

32. **插入日期**

    | **str_to_data** | **将字符串varchar类型转换成data类型，语法格式：str_to_data('字符串日期', ' 日期格式')** |
    | --------------- | ------------------------------------------------------------ |
    | **data_format** | **将data类型转换成具有一定格式的varchar类型，语法格式：data_format(data类型数据, '日期格式')** |

    mysql日期格式： %Y 年；    %m  月；   %d   日；   %h   时；  %i   分；  %s    秒； 

    **注意：插入日期时，要用str_to_data要将日期字符串转为data类型**

    ```
    insert into t_user(id, name, birth) values(1, 'zhangsan', str_to_data('01-10-1990', '%m-%d-%Y'));
    ```

    **如果你写的字符串是%Y-%m-%d的格式，就不需要进行转化**

33. **data 和 datatime 两个类型的区别**

    data 是短日期，只包含年月日信息

    datatime是长日期，包含年月日时分秒信息

    mysql短日期默认格式：%Y-%m-%d

    mysql长日期默认格式：%Y-%m-%d %h:%i:%s

    ```
    insert into t_user(id, name, birth, create_time) value(1, 'zhangsan', '1990-10-01', '2002-05-18 15:49:38');
    ```

     **now()函数获取当前系统时间，类型为datatime**

34. **修改updata（DML）**

**语法格式：updata 表名 set 字段名1=值1,字段名2=值2,字段名3=值		3,... where 条件；       注意：没有where条件限制会导致所有数据全部		更新**

```
updata t_user set name = 'jack', birth = '2001-10-07' where id = 2;
```



35. **delete删除数据**

    **语法格式：delete from 表名 where 条件**

    **注意：如果不加where条件，整张表的会被删除**

36. **快速建表**

    ```
    create table emp2 as select * from emp 
    ```

    **原理：将一个查询结果当作一张表新建**

37. **将查询结果插入一张表中**

    ```
    insert into 表名 select ...   //注意查询结果与表的结构是否匹配
    ```

38. **快速删除表中数据（truncate table 表名）**

    delete语句删除数据的原理?**(delete属于DML语句!!! )**
    **表中的数据被删除了，但是这个数据在硬盘上的真实存储空间不会被释放!!!这种删除缺点是:删除效率比较低.**
    **这种删除优点是:支持回滚，后悔了可以再恢复数据!! !**

    truncate语句删除数据的原理?
    **这种删除效率比较高,表被一次截断，物理删除。这种删除缺点:不支持回滚。**
    **这种删除优点:快速。**
    **用法: truncate table dept_bak;(这种操作属于DDL操作。)**

    大表非常大，上亿条记录?? ??
    删除的时候，使用delete，也许需要执行1个小时才能删除完!效率较低。
    可以选择使用truncate删除表中的数据。只需要不到1秒钟的时间就删除结束。效率较高。但是使用truncate之前，必须仔细询问客户是否真的要删除，并警告删除之后不可恢复!

    **truncate 是删除数据，表的结构还在**

    **drop table 表名；   //这不是删除表中数据，这是把表删除**

39. **创建表加入约束（很重要）**

    在创建表时，我们可以给表中的字段加上一些约束，来保证这个表中数据的完整性、有效性

    约束包括： 非空约束：not null   唯一性约束：unique   

    主键约束：primary key(简称PK)   外键约束：foreign key(简称FK)

    检查约束：check(mysql 不支持，Oracle支持)

40. **非空约束（not null 只有列级约束，没有表级约束）**

    ```
    create table t_vip(
    	id int;
    	name varchar(225) not null;   //name字段不能为空
    )
    ```

41. **唯一性约束：unique**

    唯一性约束unique约束的字段不能重复，但是可以都为为NULL；

42. **两个字段联合唯一（表级约束）**

    ```
    create table t_vip(
    	id int,
    	name varchar(255),
    	email varchar(255),
    	unique(name,email)  //约束没有添加在列后面，这种约束被称为表级约束
    )
    ```

    **当需要给多个字段联合起来添加一个约束时，需要用到表级约束**

    

43. **主键约束**

    主键：每一行记录的**唯一标识**。

    主键值是每一行记录的身份证号

    **注意：任何一张表都应该有主键，没有主键，表无效**

```
//添加主键约束
create table t_vip(
	id int primary key,
	name varchar(255)
);

create table t_vip(
	id int,
	name varchar(255),
	primary key(id)  //表级约束
);
```

**表级约束主要是给多个字段联合起来添加约束，primary key()可以联合多个字段做主键：复合主键！！！**

**在实际开发中不建议使用复合主键，建议使用单一主键**

**一张表，主键约束只能添加1个。（主键只能有一个）**

**主键值建议使用：int bigint char等类型，  不建议使用：varchar来做主键**

主键除了:单一主键和复合主键之外，还可以这样进行分类?
自然主键:主键值是一个自然数，和业务没关系。
业务主键:主键值和业务紧密关联，例如拿银行卡账号做主键值。这就是业务主键!在实际开发中使用业务主键多，还是使用自然主键多一些?
自然主键使用比较多，因为主键只要做到不重复就行，不需要有意义。业务主键不好，因为主键一旦和业务挂钩，那么当业务发生变动的时候，可能会影响到主键值，所以业务主键不建议使用。尽量使用自然主键。



**自动维护一个自然主键**

```
create table t_vip(
	id int primary key auto_increment,    //anto_increment表示自增，从1开始
	name varchar(255)
)
```



44. **外键（foreign key()）**

t_student 中的 cno集合从属于 t_class 中的classno集合，插入cno值必须属于classno,否则会报错。这样做是为了：**减少数据冗余造成的空间浪费**

![image-20230101185255171](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230101185255171.png)



**方案二：外键**

<img src="C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230101185105418.png" style="zoom:150%;" />

```
create table t_class(
	classno int primary key,
	classname varchar(255)
);

create table t_student(
	no int primary key auto_increment,
	name char(255),
	cno int,
	foreign key(cno) references t_class(classno)
); //t_class 是父表；  t_student 是子表。
```

**创建表顺序：先创建父表，再创建子表**

**插入顺序：先插入父表，再插入子表**

**删除顺序：先删除子表中内容，再删除父表中内容**

**删除表顺序：先删除子表，在删除父表**



**1. 子表中的外键引用的父表中的某个字段，被引用的这个字段不一定是主键，但至少具有unique唯一性**

**2.外键可以为NULL**



45. **存储引擎（不同存储引擎，表存储数据的方式不同）了解即可**

    ```
    //指定存储引擎和字符编码方式
    create table 表名(...)engine = 引擎名 default charset=编码方式；
    ```

    mysql默认的存储引擎是InnoDB   默认的编码方式是utf-8

    ```
    show create table 表名  //显示创建表时的mysql语句细节
    ```

    ```
    show engines \G;   //查看当前MySQL版本支持哪些存储引擎
    ```

46. **事务（重点：必须理解）**

事务：**事务就是一个完整的业务逻辑，本质就是多条DML语句同时成功，或者同时失败**

什么是一个完整的业务逻辑?
假设转账，从A账户向B账户中转账10000。将A账户的钱减去10000 (update语句)
将B账户的钱加上10000 ( update语句)这就是一个完整的业务逻辑。
以上的操作是一个最小的工作单元，要么同时成功，要么同时失败，不可再分。这两个update语句要求必须同时成功或者同时失败，这样才能保证钱是正确的。



**只有DML语句才会有事务这一说，其他语句和事务无关**

insert delete updata 只有以上三个语句和事务有关系，其他没有关系

只要你的操作一旦涉及到数据的增、删、改，那么就一定要考虑安全问题。



**InnoDB存储引擎：提供一组用来记录事务性活动的日志文件**



提交事务：commit; 语句

回滚事务：rollback; 语句（**回滚永远都是只能回滚到上一次的提交点**）

事务对应单词：transaction

测试一下，在mysql当中默认的事务行为是怎徉的?
mysql默认情况下是支持自动提交事务的。(自动提交)什么是自动提交?
每执行一条DML语句，则提交一次!

**将mysql的自动提交机制关闭,，开启事务：**

```
//先执行这个指令：
start transaction;
insert
insert
rollback; //这里能回滚到start transaction处
```



**事务隔离级别**

查看当前系统隔离级别：

```
select @@global.transaction_isolation;
```

设置当前系统隔离级别：

```
set global transaction isolation level 事务隔离级别;
```

四种隔离级别（**隔离级别是表名事务A和事务B之间的关系**）：

​	**读未提交 read uncommitted**

读未提交:read uncommitted(最低的隔离级别)
什么是读未提交?
事务A可以读取到事务B未提交的数据。这种隔离级别存在的问题就是:
脏读现象!(Dirty Read)
我们称读到了脏数据。
这种隔离级别一般都是理论上的，大多数的数据库隔离级别都是二档起步!

​	**读已提交  read committed**

读已提交: read committed
什么是读已提交?
事务A只能读取到事务B提交之后的数据。这种隔离级别解决了什么问题?
解决了脏读的现象。.这种隔离级别存在什么问题?
不可重复读取薮据。
什么是不可重复读取数据呢?
在事务开启之后，第一次读到的数据是3条，当前事务还没有结束，可能第二次再读取的时候，读到的数据是4条，3不等于4称为不可重复读取。

​	**可重复读 repeatable read**

可重复读: repeatable read
什么是可重复读取?
事务A开启之后，不管是多久，每一次在事务A中读取到的数据都是一致的。即使事务B将数据已经修改，并且提交了，事务A读取到的数据还是没有发生改变，这就是可重复读。

​	**序列化/串行化 serialization**

事务排队串行完成；



47. **索引**

    索引是在数据库表的字段上添加的，是为了提高查询效率存在的一种机制。一张表的一个字段可以添加一个索引，当然，多个字段联合起来也可以添加索引。索引相当于一本书的目录，是为了缩小扫描范围而存在的一种机制。

    

    提醒1:在任何数据库当中主键上都会自动添加索引对象，id字段上自动有索引，因为id是PK。另外在mysql当中，一个字段上如果有unique约束的话，也会自动创建索引对象。

    

    提醒2:在在何数据库当中，任何一张表的任何一条记录在硬盘存储上都有一个硬盘的物理存储编号。

    

    提醒3:在mysql当中，索引是一个单独的对象，不同的存储引擎以不同的形式存在，在MyISAM存储引擎中，索引存储在一个.MYI文件中。在InnoDB存储引擎中索引存储在一个逻辑名称叫做tablespace的当中。在MEMORY存储引擎当中索引被存储在内存当中。不管索引存储在哪里，索引在mysql当中都是一个树的形式存在。(自平衡二叉树:B-Tree)


    **什么条件下我们会主动给字段添加索引**

条件1:数据量庞大（到底有多么庞大算庞大，这个需要测试，因为每一个硬件环境不同)

条件2:该字段经常出现在where的后面，以条件的形式存在，也就是说这个字段总是被扫描。

条件3:该字段很少的DML(insert delete update)操作。(因为DM之后，索引需要重新排序。

**注意：唯一性比较弱的字段上添加索引用处不大。**

建议不要随意添加索引，因为索引也是需要维护的，太多的话反而会降低系统的性能。
建议通过主键查询，建议通过unique约束的字段进行查询，效率是比较高的。



**创建索引**

```
create index 索引名 on 表名(字段名);
//索引名起名规范：emp_ename_index 表名_字段名_index
```

**删除索引**

```
drop index 索引名 on emp;
```

**查询select语句使用什么索引（explain）**

```
explain select * from emp where ename = 'King';
```



**索引失效**

第一种：

```
select * from emp where ename like '%T';
```

这时候索引会失效，所以避免模糊查询的时候，以 “%” 开始。

第二种：

使用or的时候会失效，如果使用or那么要求or两边的条件字段都要有索引，才会走索引，如果其中一边有一个字段没有索引，那么另一个字段上的索引不会实现。所以这就是为什么不建议使用or的原因。

第三种：

使用复合索引的时候，没有使用左侧的列查找，索引失效

```
create index emp_job_sal_index on emp(job,sal);   //创建一个job 和 sal 字段的联合索引

explain select * from emp where sal = 800;  //索引失效
explain select * from emp where job = 'Worker';  //索引成功
```



第四种：

在where当中索引列使用了函数 

```
explain select * from emp where lower(ename) = 'smith';  //索引失效
```



48. **视图：**

    创建视图：

    ```
    create view emp_view as select * from emp;  
    ```

    删除视图：

    ```
    drop view emp_view;
    ```

    **注意：只有DQL语句（查询语句）才能以view的形式创建。**

    create view view_name as 这里的语句必须是查询语句；

49. **视图的作用**

    **我们可以面向视图对象进行增删改查，对视图对象的增删改查，会导致原表被操作!（视图的特点:通过对视图的操作，会影响到原表数据。)**

    ```
    create view emp_view as select e.name,e.sal,d.name from emp e join dept d on e.deptno = d.deptno;
    //select语句过长，经常调用很麻烦，把select的结果创建一个视图，方便对原表数据的操作；
    ```

    假设有一条非常复杂的sgr语句，而这条sQL语句需要在不同的位置上反复使用。每一次使用这个sql语句的时候都需要重新编写，很长，很麻烦，怎么办?
    可以把这条复杂的sQL语句以视图对象的形式新建。
    在需要编写这条sQz语句的位置直接使用视图对象，可以大大简化开发。
    并且利于后期的维护，因为修改的时候也只需要修改一个位置就行，只需要修改视图对象所映时的sQL语句。
    我们以后面向视图开发的时候，使用视图的时候可以像使用table一样。可以对视图进行增删改查等操作。视图不是在内存当中，视图对象也是存储在硬盘上的,不会消失。

50. **数据库设计三范式**

    第一范式:要求任何一张表必须有主键，每一个字段原子性不可再分。
    第二范式:建立在第一范式的基础之上，要求所有非主键字段完全依赖主键，不要产生部分依赖。
    第三范式:建立在第二范式的基础之上，要求所有非主键字段直接依赖主键，不要产生传递依赖。

    **口诀：**

    **多对多，三张表，关系表两个外键**

    **一对多，两张表，多的表加外键**

    **一对一，外键唯一**

    

    数据库设计三范式是理论上的。实践和理论有的时候有偏差。
    最终的目的都是为了满足客户的需求，有的时候会拿冗余换执行速度。因为在sql当中，表和表之间连接次数越多，效率越低。(笛卡尔积>
    有的时候可能会存在冗余，但是为了减少表的连接次数，这样做也是合理的，并且对于开发人员来说，sql语句的编写难度也会降低。
    面试的时候把这句话说上:他就不会认为你是初级程序员了!

    
