# MySQL
## 逻辑框架

![image-20200212163519505](images/image-20200212163519505.png)

![image-20200212162808559](images/image-20200212162808559.png)

![image-20200212162903337](images/image-20200212162903337.png)

###  Connectors

指的是不同语言中与SQL的交互
### Management Serveices & Utilities： 
系统管理和控制工具
### Connection Pool: 连接池
管理缓冲用户连接，线程处理等需要缓存的需求。
负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，
接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。

### SQL Interface: SQL接口。

接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
### Parser: 解析器。
SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。
在 MySQL中我们习惯将所有 Client 端发送给 Server 端的命令都称为 query ，在 MySQL Server 里面，连接线程接收到客户端的一个 Query 后，会直接将该 query 传递给专门负责将各种 Query 进行分类然后转发给各个对应的处理模块。
主要功能：
a . 将SQL语句进行语义和语法的分析，分解成数据结构，然后按照不同的操作类型进行分类，然后做出针对性的转发到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。
b.  如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的
### Optimizer: 查询优化器。
SQL语句在查询之前会使用查询优化器对查询进行优化。就是优化客户端请求的 query（sql语句） ，根据客户端请求的 query 语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个 query 语句的结果
他使用的是“选取-投影-联接”策略进行查询。
       用一个例子就可以理解： select uid,name from user where gender = 1;
       这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤
       这个select查询先根据uid和name进行属性投影，而不是将属性全部取出以后再进行过滤
       将这两个查询条件联接起来生成最终查询结果

### Cache和Buffer： 查询缓存。
他的主要功能是将客户端提交 给MySQL 的 Select 类 query 请求的返回结果集 cache 到内存中，与该 query 的一个 hash 值 做一个对应。该 Query 所取数据的基表发生任何数据的变化之后， MySQL 会自动使该 query 的Cache 失效。在读写比例非常高的应用系统中， Query Cache 对性能的提高是非常显著的。当然它对内存的消耗也是非常大的。
如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等
### 存储引擎接口

存储引擎接口模块可以说是 MySQL 数据库中最有特色的一点了。目前各种数据库产品中，基本上只有 MySQL 可以实现其底层数据存储引擎的插件式管理。这个模块实际上只是 一个抽象类，但正是因为它成功地将各种数据处理高度抽象化，才成就了今天 MySQL 可插拔存储引擎的特色。
     从图2还可以看出，MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者都可以按照自己的意愿来进行开发。
    注意：存储引擎是基于表的，而不是数据库。

#### 查询命令

![image-20200212163704336](images/image-20200212163704336.png)

#### 区别

![image-20200212163732423](images/image-20200212163732423.png)

## SQL执行顺序

### 手写

![image-20200212165340598](images/image-20200212165340598.png)

### 机读

![image-20200212165414016](images/image-20200212165414016.png)

![image-20200212165302041](images/image-20200212165302041.png)

## Join

![image-20200212165455947](images/image-20200212165455947.png)

```mysql
SELECT * from employees e INNER JOIN departments d on e.department_id = d.department_id;

SELECT * from employees e LEFT JOIN departments d on e.department_id = d.department_id;

SELECT * from employees e RIGHT JOIN departments d on e.department_id = d.department_id;

SELECT * from employees e LEFT JOIN departments d on e.department_id = d.department_id WHERE d.department_id is NULL;

SELECT * from employees e RIGHT JOIN departments d on e.department_id = d.department_id where e.department_id IS NULL

SELECT * from employees e LEFT JOIN departments d on e.department_id = d.department_id where d.department_id is NULL
	UNION SELECT * from employees e RIGHT JOIN departments d on e.department_id = d.department_id where e.department_id is NULL;
```

## 索引

MySQL官方对索引的定义为：索引(Index)是帮助MySQL高校获取数据的数据结构。可以得到索引的本质：索引是数据结构

![image-20200212172631886](images/image-20200212172631886.png)

数据本身之外,数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，
这样就可以在这些数据结构的基础上实现高级查找算法,这种数据结构就是索引。

![image-20200212173649619](images/image-20200212173649619.png)

我们平时所说的索引，如果没有特别指明，都是指B树(多路搜索树，并不一定是二叉树)结构组织的索引。其中聚集索引，次要索引，覆盖索引，
复合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。当然,除了B+树这种类型的索引之外，还有哈希索引(hash index)等。

### 索引分类

#### 单值索引

即一个索引只包含单个列，一个表可以有多个单列索引

#### 唯一索引

索引列的值必须唯一，但允许有空值

#### 复合索引

即一个索引包含多个列

### 基本语法

##### 创建

```mysql
CREATE [UNIQUE] INDEX  indexName ON mytable(columnname(length));
```

如果是CHAR,VARCHAR类型，length可以小于字段实际长度；
如果是BLOB和TEXT类型，必须指定length。

```mysql
ALTER mytable ADD [UNIQUE]  INDEX [indexName] ON(columnname(length));
```

##### 删除

```mysql
DROP INDEX [indexName] ON mytable;
```

##### 查看

```mysql
SHOW INDEX FROM table_name\G
```

![image-20200212191854985](images/image-20200212191854985.png)

### 索引结构

##### BTree索引

![image-20200212192541313](images/image-20200212192541313.png)

![image-20200212192631045](images/image-20200212192631045.png)

##### Hash索引

##### full-text索引

##### R-Tree索引

### 是否需要建立索引

 ![image-20200212192858667](images/image-20200212192858667.png)

### 性能分析

#### MySQL Query Optimizer

![image-20200212193304941](images/image-20200212193304941.png)

#### MySQL常见瓶颈

+ CPU:CPU在饱和的时候一般发生在数据装入在内存或从磁盘上读取数据时候
+ IO:磁盘I/O瓶颈发生在装入数据远大于内存容量时
+ 服务器硬件的性能瓶颈：top,free,iostat和vmstat来查看系统的性能状态

#### Explain

使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是结构的性能瓶颈

```mysql
EXPLAIN select * from employees;
```

![image-20200212193901659](images/image-20200212193901659.png)

+ id

  select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

  + 三种情况

    + id相同，执行顺序由上至下

      ![image-20200212194332532](images/image-20200212194332532.png)

    + id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

      ![image-20200212194353136](images/image-20200212194353136.png)

    + id相同不同，同时存在

      ![image-20200212194416441](images/image-20200212194416441.png)

+ select_type

  ![image-20200212194834172](images/image-20200212194834172.png)



![image-20200212194853575](images/image-20200212194853575.png)

+ table

  显示这一行的数据是关于哪张表的

+ type

  ![image-20200212195243264](images/image-20200212195243264.png)

显示查询使用了何种类型从最好到最差依次是：system>const>eq_ref>ref>range>index>ALL

![image-20200212195544767](images/image-20200212195544767.png)



+ possible_keys

  显示可能应用在这张表中的索引,一个或多个。查询涉及的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

+ key

  实际使用的索引。如果为null则没有使用索引

  查询中若使用了覆盖索引，则索引和查询的select字段重叠

+ key_len

  表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好

  key_len显示的值为索引最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

  计算公式=> https://www.cnblogs.com/gomysql/p/4004244.html

+ ref

  显示索引那一列被使用了，如果可能的话，是一个常数。那些列或常量被用于查找索引列上的值

+ rows

  根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

+ Extra

  包含不适合在其他列中显示但十分重要的额外信息

  ![image-20200212203633868](images/image-20200212203633868.png)

#### Case

![image-20200213192659775](images/image-20200213192659775.png)

### 索引优化

### 索引失效

![image-20200213200556089](images/image-20200213200556089.png)

#### 最佳左前列

```sql
create index idx_employee_nameAgePos on employee(name,age,pos);

# 但是使用的时候必须name必须索引 并且 name age pos 不能跳过age
```



![image-20200213201057471](images/image-20200213201057471.png)

#### 存储引擎不能使用索引中范围条件右边的列

> age > 11依然会使用索引 但是 后面pos=‘manager 不会使用索引’

![image-20200213202559608](images/image-20200213202559608.png)

![image-20200213205007403](images/image-20200213205007403.png)

like KK%相当于=常量     %KK和%KK% 相当于范围

![image-20200213205035622](images/image-20200213205035622.png)



![image-20200213211301463](images/image-20200213211301463.png)

## 查询截取分析

+ 观察至少跑一天
+ 开启慢查询日志，设置阈值，如果超过5秒的就是慢SQL，并将它抓取出来
+ explain+慢SQL分析
+ show profile
+ 运维经理 or DBA，进行SQL数据库路服务器参数调优

### 查询优化

#### 小表驱动大表

![image-20200214135936226](images/image-20200214135936226.png)

![image-20200214140241297](images/image-20200214140241297.png)

![image-20200214140248743](images/image-20200214140248743.png)

```sql
select * from employees e WHERE EXISTS (select 1 from departments d where e.department_id = d.department_id)
```

#### Order by 关键字优化

+ ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序

  + MySQL支持二种方式的排序，FileSort和Index,Index效率高。它指MySQL扫描索引本身完成排序。FileSort方式效率较低。

  + ORDER BY满足两情况，会使用Index方式排序

    + ORDER BY语句使用索引最左前列

    + 使用where子句与OrderBy子句条件列组合满足索引最左前列

      ```sql
      -- salary排在前面
      create index idx_emp_nameSalary on employees(salary,first_name,last_name);
      
      -- using index
      explain select first_name,last_name,salary from employees where salary> 5000 order by salary
      
      
      -- salary排在后面 但是order by salary
      create index idx_emp_nameSalary on employees(first_name,last_name，salary);
      
      -- using filesort
      explain select first_name,last_name,salary from employees where salary> 5000 order by salary
      ```

+ 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀
+ 如果不在索引列上，filesort有两种算法：mysql就要启动双路排序和单路排序
  + 双路排序
    + MySQL4.1之前是使用双路排序，字面意思是两次扫描磁盘，最终得到数据。读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据传输
    + 从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。
  + 取一批数据，要对磁盘进行两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二张改进的算法，就是单路排序。
  + 单路排序
    + 从磁盘读取查询需要的所有列，按照orderby列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。
  + 结论及引申出的问题
    + 由于单路是后出来的，总体而言好过双路
    + 但是用单路有问题

+ 优化策略

  + 增大sort_buffer_size参数的设置
  + 增大max_length_for_sort_data参数的设置

  ![image-20200214142840604](images/image-20200214142840604.png)

+ 总结

  ![image-20200214143035790](images/image-20200214143035790.png)

#### Group by 关键字优化

+ groupby实质是先排序后进行分组，遵照索引建的最佳左前缀
+ 当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置
+ where高于having,能写在where限定的条件就不要去having限定了。

### 慢查询日志

![image-20200214143544556](images/image-20200214143544556.png)

![image-20200214143644859](images/image-20200214143644859.png)

+ 查看是否开启及如何开启

  ```sql
  -- 默认
  SHOW VARIABLES LIKE '%slow_query_log%'
  -- 开启
  set global slow_query_log = 1
  ```

  ![image-20200214143917192](images/image-20200214143917192.png)

  

+ 查看当前多少秒算慢

  > SHOW VARIABLES LIKE 'long_query_time%';

+ 设置慢的阙值时间

  > set global long_query_time=3;

+ 为什么设置后看不出变化？

  > 需要重新连接或者新开一个回话才能看到修改值。
  > SHOW VARIABLES LIKE 'long_query_time%';
  >
  > show global variables like 'long_query_time';

+ 记录慢SQL并后续分析

+ 查询当前系统中有多少条慢查询记录

#### 日志分析工具mysqldumpshow

> mysqldumpshow --help

![image-20200214145912731](images/image-20200214145912731.png)

![image-20200214145957082](images/image-20200214145957082.png)

### 批量插入数据

+ 建表

+ 设置参数log_trust_function_createors

+ 创建函数保证每条数据都不同

  + 随机产生字符串

    ```sql
    delimiter $$
    create FUNCTION rand_string(n INT) RETURNs VARCHAR(255)
    BEGIN
    	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyz';
    	DECLARE return_str VARCHAR(255) DEFAULT '';
    	DECLARE i INT DEFAULT 0;
    	WHILE i < n DO
    	SET return_str = CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1 + RAND() * 52),1));
    	SET i = i + 1;
    	end WHILE;
    	return return_str;
    END $$
    ```

  + 随机产生部门编号

    ```sql
    delimiter $$
    create FUNCTION rand_sum()
    returns INT(5)
    BEGIN
    	DECLARE i INT DEFAULT 0;
    	SET i = FLOOR(100+RAND()*10);
    	return i;
    end $$
    ```

+ 创建存储过程

   ```sql
  delimiter $$
  create PROCEDURE insert_emp(in start int(10),in max_num int(10))
  BEGIN
  DECLARE i int DEFAULT 0
  	set autocommit = 0;
  	REPEAT
  	SET i = i + 1
  	insert into employees(first_name,last_name) values(rand_string(6),rand_string(10),RAND() * 10000);
  	UNTIL i = max_num
  	end REPEAT;
  	COMMIT;
  end $$
   ```

+ 调用存储过程

  > call insert_emp(100,500000)

### show Profile

是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优测量

官网：http://dev.mysql.com/doc/refman/5.5/en/show-profile.html

默认情况下，参数处于关闭状态，并保存最近15次的运行结果

+ 是否支持

  ![image-20200214170853007](images/image-20200214170853007.png)+ 

+ 开启功能，默认是关闭，使用前需要开启

+ 运行SQL

  ```sql
  select * from emp group by id%10 limit 150000
  select * from emp group by id%20 order by 5
  ```

+ 查看结果

  show profiles;

+ 诊断SQL，show profile cpu,block io for query 上一步前面的问题SQL 数字号码；

  ![image-20200214171132376](images\image-20200214171132376.png)

  ![image-20200214171145693](images\image-20200214171145693.png)

+ 日常开发需要注意的结论

  + converting HEAP to MyISAM 查询结果太大，内存都不够用了往磁盘上搬了。

  + Creating tmp table 创建临时表

    ![image-20200214172423604](images/image-20200214172423604.png)

    + 拷贝数据到临时表
    + 用完再删除

  + Copying to tmp table on disk 把内存中临时表复制到磁盘，危险！！！

  + locked

### 全局查询日志

+ 配置启用

  ![image-20200214172853345](images/image-20200214172853345.png)

+ 编码启用

  ![image-20200214172909877](images/image-20200214172909877.png)

+ 永远不要在生产环境开启这个功能

## MySQL锁机制

![image-20200214173304255](images/image-20200214173304255.png)

### 锁的分类

#### 从数据操作的类型（读、写）分

##### 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响

![image-20200214194257504](images/image-20200214194257504.png)

![image-20200214194341253](images/image-20200214194341253.png)

![image-20200214194326464](images/image-20200214194326464.png)

##### 写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。

![image-20200214194413898](images/image-20200214194413898.png)

![image-20200214194426962](images/image-20200214194426962.png)

+ 结论

![image-20200214194733702](images/image-20200214194733702.png)

#### 从对数据操作的颗粒度

##### 表锁（偏读）

偏向MyISAM存储引擎，开销小，加锁快，无死锁，锁定粒度大，发生锁冲突的概率最高，并发最低

> show open tables;

![image-20200214195003019](images/image-20200214195003019.png)

##### 行锁

偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）;二是采用了行级锁

+ 事务ACID

  ![image-20200214195429344](images/image-20200214195429344.png)

+ 并发事务带来的问题

  + 更新丢失

    ![image-20200214195719463](images/image-20200214195719463.png)

  + 脏读

    ![image-20200214195737025](images/image-20200214195737025.png)

  + 不可重复读

    ![image-20200214195801302](images/image-20200214195801302.png)

  + 幻读

    ![image-20200214195824621](images/image-20200214195824621.png)

+ 事务隔离级别

  ![image-20200214200402720](images/image-20200214200402720.png)

+ case

  ![image-20200214201548328](images/image-20200214201548328.png)

  + 无索引行锁升级为表锁

    varchar  不用 ' '  导致系统自动转换类型, 行锁变表锁

  + 间隙锁

    ![image-20200214202709367](images/image-20200214202709367.png)

+ 如何锁定一行

  ![image-20200214203103046](images/image-20200214203103046.png)

+ 总结

  ![image-20200214203942215](images/image-20200214203942215.png)

  ![image-20200214204015451](images/image-20200214204015451.png)

+ 优化建议
  + 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
  + 合理设计索引，尽量缩小锁的范围
  + 尽可能较少检索条件，避免间隙锁
  + 尽量控制事务大小，减少锁定资源量和时间长度
  + 尽可能低级别事务隔离

#### 页锁（了解）

开销和加锁时间界于表锁和行锁之间：会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

## 主从复制

### 基本原理

### 基本原则

### 最大问题

### 一主一从常见配置

