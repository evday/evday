# MySQL数据库 :four_leaf_clover:

## :one: 库操作:deciduous_tree:

```python
show databases (查看所有的数据库) 
use 数据库名 
DROP DATABASES 数据库名
ALERT DATABASE 数据库名 charset utf8  (修改数据库字符编码)
```

## :two: 存储引擎:deciduous_tree: 

1. ### 什么是存储引擎

   现实生活中我们使用文件存储不同类型的文件，每种文件类型对应各自不同的处理机制，如比如处理文本用txt类型，处理表格用excel，处理图片用png等。同样数据库中的表也有不同的的类型，表的类型不同，会对应不同的mysql处理机制，这些表类型就称为存储引擎。存储引擎说白了就是如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法。

2. ### MySQL 中都有哪些存储引擎，它们有什么区别

   在Windows中可以使用命令 `show engines`  查看所有的存储引擎  在Linux 中使用 `show engines\G` 查看所有存储引擎

   `InnoDB` 是`MySQL` 默认的存储引擎 ，其特点是支持事物、支持行锁、支持外键，并且默认读取的时候不会产生锁。

   `MyISAM` 不支持事物，支持表锁、支持全文索引。在 `MySQL 5.5.8` 版本之前是默认的存储引擎

   `NDB`  是 一个集群存储引擎，其特点是将数据全部放在内存中，因此按主键查找的速度相当快。(OLTP)

   `Memory` 数据存放在内存中，数据库重启或发生崩溃，表中的数据都会消失。`Memory` 默认使用哈希索引。

   ​	      (OLTP数据库应用中的临时表)和(OLAP数据库应用中的维度表)

   `Inforbright` 特点是存储是按照行而非列的，适合 (OLAP) 的数据库应用。

3. ###  `OLTP` 和`OLAP`的区别

   数据处理大致可以分成两大类：联机事务处理OLTP（on-line transaction processing）、联机分析处理OLAP（On-Line Analytical Processing）。OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。

   `OLTP` 联机事物处理系统，表示事务性非常高的系统，一般是高可用的在线系统，在这样的系统中，以小的事务以及小的查询为主，单个数据库中每秒处理的交易成百上千个，`Select` 语句执行量每秒高达成千上万个。常见的OLTP系统有，电子商务，银行，证券等。

   `OLAP` 联机分析处理系统，也就是数据仓库。所谓数据仓库是对于大量已经由OLTP形成的数据的一种分析型的数据库，用于处理商业智能、决策支持等重要的决策信息；数据仓库是在数据库应用到一定程序之后而对历史数据的加工与分析，读取较多，更新较少，(大数据)

## :three: 表操作:deciduous_tree:

1. ### 表的基本操作

   ```python 
   create databases db1 charset utf8; # 创建数据库
   use db1; # 使用数据库
   create table t1(id int,name varchar(30),sex enum("male","female"),age int(3)); # 创建表t1
   show tables; # 显示当前库里的所有表
   desc t1; # 查看表结构
   show create table t1\G; #查看详细表结构，存储引擎可以从这里查看
   select id,name,sex,age from t1; # 从t1表中查找id,name,sex,age字段对应的数据
   insert into t1 values (1,"bai","male",18),(2,"xiang","female",19); # 往t1表中插入数据
   ```

2. ### 数据库中的数据类型

   1. **数值类型**

      ```python
      1）整数类型 存储年龄、等级、id、各种号码等 分为 smallint int bigint 默认长度为11，不用刻意指定
      2）浮点型 存储身高、体重、薪资 float(单精度，非准确)、double(双精度，非准确)、decimal(准确小数值)
      ```

   2. **日期类型**

      存储用户注册时间、文章发布时间、员工入职时间、出生时间等  date(日期)、time(时分秒)、datetime(日期+时分秒)、year(年)、timestamp(日期+时分秒) ；datetime(8字节存储空间) 对比 timestamp(4字节存储空间) 浪费空间

   3. **字符串类型** 

      char (定长，简单粗暴、浪费空间、存取速度快)

      varchar(不定长，精准、节省空间、存取速度慢)

   4. **枚举、集合类型**

      enum  单选 只能在给定的值当中选择一个

      set 多选  在给定的值当中选一个或者多个

3. ### 完整性约束

   ```python 
   NOT NULL & default # NULL NULL 不可为空，必须赋值，可指定默认值 default,不指定默认显示NULL
   eg: sex enum("male","female") not null default "male"
     
   UNIQUE #唯一 
   eg: #方法一 create table department(id int,name varchar(10) unique,comment varchar(100))
   	#方法二 create table department(id int,name varchar(10),comment varchar(100) constraint uk_name unique(name))
   NOT NULL & UNIQUE # 不为空且唯一 （主键）
   eg: create table t1(id int not null unique)
   # 联合唯一
   eg: create table service(id int primary key auto_increment,name varchar(10),host varchar(20) not null ,port int not null,unique(host,port)) # 联合唯一
     
   PRIMARY KEY #主键primary key是innodb存储引擎组织数据的依据，innodb 称为索引组织表，一张表只有一个主键
   # 单列主键
   eg：create table department(id int primary key,name varchar(20),comment varchar(20))
   	create table department(id int,name varchar(20),constraint pk_name primary key(id))
   # 双列主键(不常用)
   eg: create table service(ip varchar(20),port char(5),service_name varchar(10) not null primary key(ip,port))
     	
   AUTO_INCREMENT #自增，且被约束字段同时被key约束
   eg: create table student(id int primary key auto_increment,name varchar(20))
     	insert into student values ('bai'),('xiaotu');  # 不指定ID 自增
       insert into student values (3,"xiang"),(4,"laozhongyi"); # 也可指定id
   ps: # 对于自增字段 使用delete删除后，再插入值，该字段任然按照前面的位置继续增长
     	eg: delete from student; # 只删一行数据 指定删除 delete from student where id = 1；
      	# 可以使用truncate清空表，这时候自增字段从1开始
       eg: truncate student;

         
   FOREIGN KEY  # 外键 表类型必须是InnoDB 存储引擎，且被关联字段保证唯一
   eg: create table press(id int primary key auto_increment,name varchar(20));
   	#press_id外键，关联父表（press主键id），同步更新，同步删除
     	create table book(id int primary key auto_increment,name varchar(20),press_id int not null,foreign key(press_id) references press(id)on delete cascadeon update cascade);
   # 多对多
   eg: create table author(id int primary key auto_increment,name varchar(20));
     	# 存放多对多关系(第三张表)
       create table author2book(id int not null unique auto_increment,author_id int not null,
       book_id int not null,constraint fk_author foreign key(author_id) references author(id)
       on delete cascade on update cascade,constraint fk_book foreign key(book_id) references book(id) on delete cascade on update cascade,primary key(author_id,book_id));
     	# 给 author 插入数据
       insert into author(name) values('bai'),('xiang'),('laozhongyi'),('huazai');
   # 一对一 学生一定是一个客户，客户不一定是学生，但有可能成为学习
   eg: create table customer(id int primary key auto_increment,name varchar(20) not null,qq varchar(20) not null,phone char(11) not null);
     	create table student(id int primary key auto_increment,class_name varchar(20) not null,
       customer_id int unique,foreign key(customer_id) references customer(id) on delete cascade
       on update cascade);
   ```

4. ### 修改表 ALTER TABLE

   ```python
   1) #修改存储引擎
   eg: alter table book engine=innodb;
   2) #添加字段
   eg: alter table student add name varchar(20) not null,add age int not null default=22;
   3) #删除字段 
   eg: alter table student drop qq;
   4) #修改字段类型modify
   eg: alter table student modify id int(11) not null primary key auto_increment; #修改为主键
   5）#增加约束
   eg: alter table student modify id int(11) not null auto_increment;
   6) #对已存在的表增加复合主键
   eg: alter table service modify add primary key(host_ip,port);
   7) # 增加主键
   eg: alter table student1 modify name varchar(10) not null primary key;
   8) #增加主键和自增长
   eg: alter table student modify id int not null primary key auto_increment;
   9) #删除主键(先删除自增约束，再删除主键)
   eg: alter table student modify id int not null;
     	alter table student drop primary key;
   ```

5. ### 复制表

   ```python
   # 复制表结构+内容（key不会被复制，主键、外键和索引）
   eg: create table new_book select * from book;
   # 只复制表结构
   eg: create table new_book select * from book where 1=2;
   # 删除表
   eg: drop table new_book;
   ```

## :four:  数据操作:deciduous_tree: 

1. ### 单表查询

   关键字执行优先级:ballot_box_with_check:

   ```python
   # from-->where-->group by-->having-->select-->distinct-->order by-->limit
   1) 找到表from #从那张表查
   2）拿着where 指定的条件从表中取出一条条记录
   3）将取出的一条条记录进行group by分组，没有group by 默认为一组
   4）将分组结果having 过滤
   5）执行select #选择
   6）执行distinct #去重
   7）将得到的结果进行排序 order by
   8）限制显示结果的条数
   ```

   WHERE 约束:ballot_box_with_check:

   ```python
   1) 比较运算符：> < >= <= <> !=
   2) between 80 and 100 值在10到20之间
   3) in(80,90,100) 值是10或20或30
   4) like 'egon%' pattern可以是%或_，%表示任意多字符_表示一个字符 
   5) 逻辑运算符：在多个条件直接可以使用逻辑运算符 and or not
   ```

   GROUP BY 分组:ballot_box_with_check:

   ```python 
   # GROUP BY 关键字 和 GROUP_CONCAT() 函数一起使用
   eg: select post GROOUP_CONCAT(name,salary) from employee GROUP BY post;
   # GROUP BY 关键字 和 聚合函数 一起使用
   eg: select post avg(salary) from employee GROUP BY post;
   ```

   HAVING 与 WHERE 不同之处:couplekiss_woman_woman:

   ```python
   # 执行优先级 WHERE > GROUP BY > HAVING
   # WHERE 发生在分组group by 之前，因而where 中可以有任何字段，但是不能有聚合函数
   # HAVING 发生在分组 group by之后 因而having可以使用分组字段，不能直接使用其他字段，可以使用聚合函数
   ```

   ORDER BY 对查询结果排序

   ```python
   # 降序desc 按照年龄排序，年龄相同按照薪资排序
   eg：select * from employee order by age, salary desc
   ```

   LIMIT  限制查询结果的记录数

   ```python
   # #从第5开始，即先查询出第6条，然后包含这一条在内往后查5条
   eg：select * from employee order by salary desc limit 5,5;
   ```

   REGEXP 使用正则表达式 

   ```python
   # 以 ale 开头
   eg: select * from employee where name REGEXP '^ale';
   # 以 on 结尾
   eg: select * from employee where name REGEXP 'on$';
   # 查看所有员工中名字是jin开头，n或者g结果的员工信息
   eg: select * from employee where name REGEXP '^jin.*[ng]$';
   ```

2. ### 多表查询

   多表链接查询：

   ```python
   # 外链接语法
   select 字段列表 from 表1 inner|left|right|full join 表2 on 表1.字段=表2.字段 #ps:mysql 不支持full 
   1) # 交叉链接，不适用任何匹配，笛卡尔积 eg: select * from employee,department
   2) # 內链接 inner join，只链接匹配的行 找到两张表的共同部分，相当于利用条件从笛卡尔积中筛选出正确的结果
      eg: select employee.id,employee.name,department.name form employeee inner join department on
          employee.dep_id = department.id;
   3) # 左链接 left join，优先显示左表的全部记录 本质是在內链接的基础上增加左边有右边没有的结果
      eg: select employee.id,employee.name,department.name from employee left join department on
          employee.dep_id = department.id; # 无结果适用null填充
   4) # 右链接 right join 优先显示右表的全部记录 本质是在內链接的基础上增加左边没有右边有的结果
      eg: select employee.id,employee.name,department.name from employee right join department on
          employee.dep_id = department.id; # 无结果适用null填充
   5) # 全外链接，显示左右两个表的全部记录 注意 mysql 不支持full join 但是我们可以构建出这种效果
      eg: select * from employee left join department on employee.de_id = department.id union
          select * from employee right join department on employee.de_id = department.id 
   	   #注意关键字 union 会去掉相同的记录。
   ```

   符合条件的链接查询

   ```python 
   # 以内连接的方式查询employee和department表，并且以age字段的升序方式显示
   eg: select employee.name,employee.age,department.name from employee inner join department where 
   	employee.dep_id = department.id and age>25 order by age desc;
   ```

   子查询 

   ```python
   #1：子查询是将一个查询语句嵌套在另一个查询语句中。
   #2：内层查询语句的查询结果，可以为外层查询语句提供查询条件。
   #3：子查询中可以包含：IN、NOT IN、ANY、ALL、EXISTS(返回True 或者 False) 和 NOT EXISTS等关键字
   #4：还可以包含比较运算符：= 、 !=、> 、<等
   1) 带in关键字的子查询 
      # 查询平均年龄在25岁以上的部门名
      eg: select id,name from department where id in (select dep_id from employee group by	dep_id
          having avg(age) > 25);
   2) 带比较运算符的子查询
      # 查询大于部门内平均年龄的员工名、年龄
      eg: select t1.name,t1.age from employee t1 inner join (select dep_id,vag(age) avg_age from 
          employee group by dep_id) t2 on t1.dep_id = t2.dep_id where t1.age > t2.avg.age;
   3) 带EXISTS关键字的子查询 内存查询不返回结果而是返回一个Bolle值，True 或者 False
      # department表中存在id=200，Ture
      eg: select * from employee where exists (select id from department where id = 200);
   单表链接查询
   # 查询每个部门最新入职的那位员工
   eg: select * from employee as t1 inner join (select post,max(hire_date) max_date from employee
       group by post) as t2 on t1.post = t2.post where t1.hire_date = t2.max_date;
   ```

## :five: 索引与慢查询优化:deciduous_tree:

1. 什么是索引？

   索引在mysql中也叫做“键”，是存储引擎快速找到记录的一种数据结构。索引对于良好的性能非常关键，尤其是当数据量越来越大，缩影对性能的影响愈发重要。索引优化是对查询性能优化最有效的手段。

2. MySQL 常见索引

   ```
   普通索引 index
   唯一索引：
   		主键索引 primary key :不为空且唯一
   		唯一索引 unique 唯一
   联合索引:
   		PRIMERY KEY(ID,NAME): 联合主键索引
   		UNIQUE(ID,NAME): 联合唯一索引
   		INDEX(ID,NAME): 联合普通索引
   ```

3. 索引的两大类型hash 和 btree

   ```python
   # 我们在创建上述索引的时候，可以为其指定类型，分两类
   1) hash 类型索引：查询单条快，范围查询慢
   2) btree 类型索引：层数越多，数据量指数级增长(INNODB 默认支持btree)
   # 不同的存储引擎支持的索引类型
   1) INNODB 支持事物，支持行锁，支持btree,full-text 不支持 hash
   2) MyISAM 不支持事物，支持表锁，支持btree,full-text 不支持 hash
   3) Memory 不支持事物，支持表锁，支持btree，hash 不支持 full-text
   4) NDB 支持事物，支持行锁，支持hash 不支持 btree full-text
   ```

4. 创建、删除索引

   ```python
   # 建表时创建索引
   eg: create table t1(id int,name varchar(20),age int,sex enum("male","female")
       unique key uni_id(id),# 唯一索引 
       index ix_name(name)); # 普通索引
   # CREATE 在已经建立的表上创建索引
   eg: create index ix_age on t1(age)
   # ALERT TABLE 在已存在的表上建立索引
   eg: alter table t1 add index ix_sex(sex);
   # 删除索引
   eg: DROP INDEX 索引名 on 表名；
   ```

5. 建立索引的原则

   ```
   1）最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹    配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的,如果建    立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
   2）尽量选择区分度高的列作为索引 比如不要用性别来做索引，因为性别无外乎male 和 female 区分度几乎为0
      这时候和与全表扫描的磁盘IO没有多大区别，所以速度会很慢。
   3）索引列不能参与计算
   ```
##:six: 数据备份pymysql模块:deciduous_tree:

1. 使用Mysqldump 实现逻辑备份

   ```python
   # 单库备份
   mysqldump -uroot -p123 db1> db1.sql
   mysqldump -uroot -p123 db1 table1 table2 > db1_table1-table2.sql # 单库下面指定表的备份
   # 多库备份
   mysqldump -uroot -p123 --databases db1 db2 mysql db3 > db1_db2_mysql_db3.sql
   # 备份所有库
   mysql -uroot -p123 --all-datebases > all.sql
   ```

2. 恢复逻辑备份

   ```python
   # 进入到备份目录backup下
   [root@egon backup]# mysql -uroot -p123 < /backup/all.sql # < 后面是备份文件的路径
   ```

3. 数据库迁移

   ```python
   # 执行数据库迁移务必保证数据库版本相同
   mysqldump -h 源ip -uroot -p123 --databases db1 | mysql -h 目标ip -uroot -p456
   ```

4. pymysql模块

   ```python
   1) 链接、执行sql、关闭游标
   import pymysql
   conn=pymysql.connect(host='127.0.0.1',user='root',password='',database='learn',charset='utf8')
   cursor = conn.cursor #获取游标
   sql = 'select * from userinfo where name="%s" and password = "%s"%(user,pwd)' # 执行sql
   res = cursor.execute(sql) # 执行sql,返回sql 查询成功的记录数目
   cursor.close()#关闭游标
   conn.close() # 关闭链接
   2) execute() 之sql注入
   # sql 注入的方式
   	1. --之后的sql均会被注释掉,绕过密码登录
     	eg: select * from userinfo where name = 'xiang' --'hahahaha' and password = "123" 
       2. 使用or绕过登录,用户名不存在时，1=1 永远成立
       eg: select * from userinfo where name = 'xiang' or 1=1 --'hahhaha' and password = '123';
   # 解决方式(excute帮我们做字符串拼接)
   eg: sql='select * from userinfo where name = %s and password = %s' # !!! %s无需加括号
     	res = cursor.execute(sql,[user,pwd])
   ```

## :seven: 视图、触发器、事务、存储过程、函数:deciduous_tree:

1. **视图**

   ```python
   # 什么是视图
   视图是一个虚拟表(非真实存在)，其本质是【根据SQL语句获取动态的数据集，并为其命名】，用户使用时只需要使用【名称】即可获得结果集，可以将该结果集当做表来用，这样再想操作该数据集的时候无需重复写复杂的sql语句了。
   # 创建视图 语法CREATE VIEW 视图名称 AS  SQL语句
   eg: CREATE VIEW teacher_view as select tid from teacher where tname='李平老师'；#注意写关键字VIEW
   	ps: 数据库中只存放视图的定义，不存在视图对应的数据
   # 使用视图
   1）create view course_view as select * from course;# 创建视图，视图名为course_view
   2）update course_view set name = "xxx" # 更新视图中的数据
   3）insert into course_view values(5,'yyy',2)# 往视图中插入数据
   4）select * from course # 发现原始表的记录也跟着修改了
   # 修改视图 语法 ALTER VIEW 视图名称 AS SQL语句
   eg: alert view teacher_view as select * from teacher where id > 3;
   # 删除视图
   eg: DROP VIEW 视图名
   ```

2. **触发器** 

   ```python
   # 使用触发器可以定制用户对表进行【增、删、改】操作时前后的行为，注意没有查询	
   1) 创建触发器
   eg: delimiter//
     	create trigger tri_after_insert_cmd after insert on cmd for each row
       begin
       if new.success = 'no' then
       insert into errlog(err_cmd,err_time)valuse(new.cmd,new.sub_time); #这里的分号必须加上
       end if
       end//
       delimiter; # 特别注意：new 表示即将插入的数据行，old 表示即将删除的数据行
   2) 使用触发器
   # 触发器无法由用户直接调用，而是由对表的【增、删、改】操作被动 触发的
   eg: drop brigger tri_after_insert_cmd;
   ```

3. **事务** 

   ```python
   # 事务用于将某些操作的多个sql作为原子操作，一旦有某一个出现错误，即可回滚到原来的状态，从而保证数据库数	据的完成性
   eg: start transaction;
     	update user set balance = 100 where name = 'xiang'; # xiang支付了嫖资100块
       update user set balance = 10 where name = 'bai'; # 皮条客bai老板 收了10块中介费
       update user set balance = 90 where name = 'kun'; # kun小姐 收到了嫖资 90 块
      	rollback;
       commit;
   ```

4. **存储过程** 

   ```python 
   1）什么是存储过程
   存储过程包含了一系列可执行的sql语句，存储过程存放在MySQL中，通过调用它的名字就可以执行它内部的一堆sql语句，使用存储过程可以替代程序写的SQL语句，实现程序和SQL解耦，但是给给程序的扩展带来不便。、
   ps: 程序与数据库结合使用的三种方式
    	1. MySQL：存储过程 程序： 调用存储过程
       2. MySQL:         程序： 纯sql语句
       3. MySQL:         程序： 类和对象，即ORM(但本质还是sql语句)
   2）创建存储过程
   eg: delimiter // # 无参
       create procedure p1()
       begin
       	select * from blog;
         	insert into blog(name,sub_time) values ('风清'，now());
       end //
       delimiter;
       call p1() # 在mysql中调用
       cursor.callproc('p1') # 基于pymysql调用
       
       delimiter // # 有参
       create procedure p2(
         	in n1 int,
         	in n2 int
       )
       BEGIN 
       	SELECT * FROM blog where id > n1;
      	END //
       delimiter;
       call p2(3,2) # 在mysql中调用
       cursor.callproc('p2',(3,2)) # 基于pymysql调用
    3）删除存储过程
   	drop procedure proc_name 
   ```

5. **函数** 

   ```python
   # 常用的MySQL内置函数
   聚合函数
       AVG(col)返回指定列的平均值
       COUNT(col)返回指定列中非NULL值的个数
       MIN(col)返回指定列的最小值
       MAX(col)返回指定列的最大值
       SUM(col)返回指定列的所有值之和
       GROUP_CONCAT(col) 返回由属于一组的列值连接组合而成的结果    
   ```
