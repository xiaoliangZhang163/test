# SQL初级

数据库：DataBase（DB）是指长期保存在计算机上，按照一定规则存储的数据集合，这个数据集合可以让用户或者应用共享其中的数据。

对数据库（本文以MySQL为例）的操作：

1、通过系统cmd ---> 进入数据库 mysql -u root -p -->输入密码 出现welcome...证明登录成功；

2、通过数据库管理软件NavicatPremium操作。

MySQL中的常用数据类型：

int 整型

float、double 浮点型

float(5,2)表示最多可5位，其中必须有2位是小数如999.99

decimal 用来存储工资从MySQL 5.1之后才有

char 固定长度的字符串类型 char(10) 'aaa       '占10位

varchar 可变长度字符串类型 varchar(10) 'aaa' 占3位

text 字符串类型超长

blob 字节类型

date 日期类型格式为 yyyy-MM-dd

time 时间类型格式为 HH:mm:ss

timestamp 时间戳类型 yyyy-MM-dd HH:mm:ss 会自动赋值 会自动赋值当前时间

datetime 时期类型yyyy-MM-dd HH:mm:ss

SQL的概述：结构化查询语言Structure Query Language，也是关系型数据库语言的国际标准。

SQL语句的分类：
一、DDL (Data Definition Language)数据库定义语言，用来定义数据库对象：库、表、列等。
对数据库的操作：

1、查询：show databases;查看当前数据库服务器中有几个库；show create database数据库名;查看前面创建的数据库定义信息。

2、创建：create database 数据库名 character set utf8; 创建数据库并设置字符集。

3、修改：alter database 数据库名 character set gbk;修改数据库编码集。

4、删除：drop database 数据库名;

5、其他：select database();查看当前正在使用的数据库；use 数据库名;切换数据库；show tables;查看当前库下所有的表。

对表的操作：

1、创建：create table 表名(字段名1(表头) 字段类型(本列可存储的数据类型) 约束条件(可不写),...,字段名N 字段类型 约束条件)；

2、修改： alter table 表名 character set 字符集; 修改表的字符集；alter table 表名 change 旧字段名 新的字段名 数据类型字段重命名（不能修改字段的类型和约束）；alter table 表名 modify 字段名 字段类型；修改列属性（只能修改类型和约束条件）；alter table 原表名 rename 新表名; rename table 原表名 to 新表名修改表名。

3、删除：drop table 表名。

4、查看：desc表名;查看数据库中的表结构；show create table表名;查看创建表的语句；show tables;查看数据库中所有的表。

对字段的操作:

1、新增：alter table 表名 add 新增的字段名 字段类型；

2、删除：alter table 表名 drop 字段名。

二、DCL (Data Contorl Language)数据控制语言，用来定义访问权限和安全级别，需要使用root登录才可以。
1、创建用户：create user 用户名@xxx identified by '密码';

xxx可以是localhost (表示只可在本地登录)、% (代表所有ip地址)、或其他指定IP地址。

2、分配权限：grant 权限1,...权限n on 数据库.* to 用户名@xxx;表示给用户名@xxx对数据库中所有表权限1。（*.*表示所有数据库中所有表，超级用户类似root）。All表示所有权限，create drop alter select，增删改查四权限。

3、撤销授权：revoke 权限1,...权限n on 数据库名.* from 用户名@xxx;

4、查看权限：show grants for 用户名@xxx;

5、删除用户：drop user 用户名@xxx;

三、DML(Data ManipulationLanguage)数据操作语言，用来定义数据库记录(数据)：增、删、改。
1、insert插入：insert into 表名(列名1,..列名n) values(值1,...值n)

注意：1. 列名可省略但是在插入数据的时候和表结构完全一致

   2. 列名和值的类型/个数/顺序 必须一致

   3. 插入空值，使用 null

   4. 日期类型可以使用- , . 分割 或者不分割 "2018-3-12"   

   5. 同时插入多条数据(值1,值2...值n),(值1,值2...值n),(值1,值2...值n)...

2、update修改：update 表名 set 列名1=值1,列名2=值2.... where 列名=值

注意：where子句可省略，表示修改表中所有记录.。

3、delete删除：delete from 表名 where 列名=值

    内容延伸：TRUNCATE TABLE：删除内容、释放空间但不删除定义。删除更彻底，所有的日志记录都会没有，并且是一把全清，不跟where条件。自增长的ID列也会归零，以后插入记录ID从1开始。
        DELETE TABLE:删除内容不删除定义，不释放空间。再插入记录，ID会从上次最大的数字开始，可有where条件。
        DROP TABLE：删除内容和定义，释放空间。

四、DQL(Data Query Language)数据查询语言，用来查询记录(数据)。
查询语句，不改变数据，只是返回一个数据集（虚拟表）。

查询语句书写顺序：

    select（字段控制）列名（AS 别名）from 表名 where - group by - having-- order by - limit

查询语句执行顺序：

    from - where - group by - having - select - order by - limit

列名：最终返回的结果集;

表名：得出结果的表（可以多张联合）;

行条件：条件可使用的符号有：>，<，！= ，<>，=，> = ，<=，之间...和以及in在XXX范围内，is null是空，and和，or或者，not非，like模糊查询，接字符串％表示任意0-n个字符，_任意一个字符。

字段控制：distinct去重，IFNULL（列名，值）用指定值替换null，AS别名（长列名改为新名，查看方便，as可省略）。

聚合函数：sum avg max min count，对结果进行处理

group by对结果分组;

拥有分组后的行条件;

order by列名aec / des对结果升序/降序;

limit x，n从x开始取n个。限定查询结果的起始行以及总行数。用于分页查询。

1.数据总数count（）; 2.每页显示数pageSize（）; 3.页数= count％pageSize == 0？count / pageSize：count / pageSize + 1; 4.第一页起始行：0一页10条：select * from xx limit 0,10;每页的起始行=（页码-1）* pageSize