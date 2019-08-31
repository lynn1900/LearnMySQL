# MySQL学习笔记

## 准备工作

1. 环境变量配置(为path路径附加mysql的bin目录)

- 一次性方案：`PATH="$PATH":/usr/local/mysql/bin`
- [永久性方案](https://www.cnblogs.com/wangrui-techbolg/archive/2012/12/22/2829614.html)

2. 修改**密码**：`ALTER USER 'root'@'localhost' IDENTIFIED BY ‘密码’;`

3. 数据库的启动：`net start mysql`（也可以通过系统偏好设置-mysql进行启动）

4. 数据库的连接：`mysql -h localhost -u root -p`
5. 清屏：control + L
6. 取消执行当前语句：\c



## 库级操作

- **显示**数据库：`show databases;`
- **选择**数据库：`use dbName;`
- **创建**数据库：`create database dbName charset utf8;`
- **删除**数据库：`drop database dbname;`



## 表级操作

- **显示**表：`show tables;`

- **创建**表：

  ```
  create table 表名 (
  列名称1　列类型　[列参数]　[not null default ],
  ....列2...
  ....
  列名称N　列类型　[列参数]　[not null default ]
  )engine myisam/innodb charset utf8/gbk
  ```

  例：班级成员信息

  ```
  create table class(
  Id int primary key auto_increment,
  Sname varchar(10) not null default '',
  Gender char(1) not null default '',
  Company varchar(20) not null default '',
  Salary decimal(6,2) not null default 0.00,
  Fanbu smallint not null default 0
  )engine myisam charset utf8;
  ```

- **删除**表：`drop table 表名;`

- **查看**建表过程：`show create table 表名;`

- **改表名**：`rename 表名;`



## 列操作

- **增加**列：

  1. 默认加在最后一列：`alter table tableName add 列名称 列类型 列参数;`

  2. 加在某列后：`alter table 表名 add 列名称 列类型 列参数 after 某列名称;`

  3. 加在第一列：`alter table 表名 add 列名称 列类型 列参数 first;`

- **删除**列：`alter table 表名  drop 列名;`

- **修改**列类型：`alter table 表名 modify 列名 新类型 新参数;`

- **修改**列名：`alter table 表名 change 旧列名 新列名 新类型 新参数;`

- **清空**表数据：`truncate 表名;`

- **查看**列：`desc 表名;`



## 三大列类型

### 数值型

Note：除了整数型，其他都要加引号

- **整型：Tinyint/ smallint/ mediumint/int/ bigint**

  【整型所占字节与存储范围】占字节越多,存储范围越大。

  |   类型    | 字节 | 位数 | 存储范围(无符号) |  存储范围（带符号）  |
  | :-------: | :--: | :--: | :--------------: | :------------------: |
  |  tinyint  |  1   |  8   |      0—255       |       -128—127       |
  | smallint  |  2   |  16  |     0—65535      |     -32768—32767     |
  | mediumint |  3   |  24  |       ...        |         ...          |
  |    Int    |  4   |  32  |       ...        |         ...          |
  |  bigint   |  8   |  64  |                  |                      |
  |  一般地   |  n   |  8n  |     0—2^8n-1     | -2^(8n-1)—2^(8n-1)-1 |

  涉及**二进制补码**问题：最高位的0/1表示符号，0表示正，1表示负。以tinyint为例，正数 = 绝对值位，负数= 绝对值位 - 128，如正数：0000 0000 ---> 0，0111 1111 --->127，负数：1000 0000 ---> -128，1111 1111 --->-1

  **【整型系统的可选参数】XXint(M) unsigned zerofill**

  例：`age tinyint(4) unsigned`、`stunum smallint(6) zerofill`

  `Unsigned`：代表此列为无符号类型， 范围从0开始；不加unsinged， 则该列默认是有符号类型,范围从负数开始。

  `Zerofill`：代表0填充，即：如果该数字不足参数M位，则自动补0，补够M位。如果没有zerofill属性，单独的参数M，没有任何意义。如果设置某列为zerofill，则该列已经默认为 unsigned，无符号类型。

- **小数型：浮点型：float(M,D)、定点型：decimal(M,D)**

  M是"精度"，代表总位数；D是"标度"，代表小数位。如float(6,2)表示-9999.99—9999.99，如果加了unsigned，则是0—9999.99。

  【float和decimal空间上的区别】float能存10^38，10^-38。如果 M<=24，占4个字节，24 <M <=53，8个字节；decimal () ，变长字节。decimal比float精度更高，适合存储货币等要求精确的数字。

### 字符串型

- **定长字符串：Char(M)**

  M 代表宽度， 0<=M<=255之间。如果实际存储内容不足M个，则后面加空格补齐。取出来的时候，再把后面的空格去掉。

  例：char(5)，输入 'aa ' ---> 存储 'aa   ' ---> 取出 'aa'。

- **变长字符串：Varchar(M)**

  M代表宽度，0<=M<=65535（以ascii字符为例，utf822000左右）。不用空格补齐，但列内容前有1～2个字节来标志该列的内容长度。

  【char和varchar比较】好比于坐车，一种是统一票价，提高了效率，省了售票员；另一种是根据站数区分票价，需要一个售票员，专门记录客户的站数。

  |  类型   | 宽度 | 可存 | 实存 |     实占空间     |     利用率     |
  | :-----: | :--: | :--: | :--: | :--------------: | :------------: |
  |  char   |  M   |  M   |  i   |        M         |   i/M<=100%    |
  | varchar |  M   |  M   |  i   | i 字符+(1~2)字节 | i/(i+1~2)<100% |

  注意: char(M)，varchar(M)限制的是字符，不是字节。即 char(2) charset utf8，能存2个utf8字符，比如'中国'。

  【char与varchar型的选择原则】

  1、考虑空间利用效率

  四字成语表，char(4), 

  个人简介，微博140字，varchar(140)

  2、考虑速度

  用户名: char 

- **Text 文本类型：** 可以存比较大的文本段，搜索速度稍慢。

  不用加默认值(加了也没用)

- **Blob二进制类型：** 用来存储图像，音频等二进制信息。

  注意：当输入文本类型时，不会报错，意义在于防止因为字符集的问题，导致信息丢失。比如:一张图片中有0xFF字节，这个在ascii字符集认为非法，在入库的时候，被过滤了。

### 日期时间类型

|       类型        |               标准               |                    范围                    | 存储需求 |
| :---------------: | :------------------------------: | :----------------------------------------: | :------: |
|     日期 date     |            YYYY-MM-DD            |           1000-01-01～9999-12-31           |  3字节   |
|     时间 time     |             HH:MM:SS             |           -838:59:59 ~ 838:59:59           |  3字节   |
| 日期时间 datetime |       YYYY-MM-DD HH:MM:SS        | 1000/01//01 00:00:00 ~ 9999:12:31 23:59:59 |  8字节   |
| 时间戳 timestamp  |            同datetime            |                 同datetime                 |  4字节   |
|     年份 year     | YYYY(8.0版本不再支持year(2)数据) |             1901～2155 和 0000             |  1字节   |

注：在部分"数据库指导"文档中，会推荐使用timestamp类型代替datetime字段，其理由是timestamp类型使用4字节，而datetime字段使用8字节，但随着磁盘性能提升和内存成本降低，在实际生产环境中，使用timestamp类型并不会带来太多性能提升，反而可能因timestamp类型的定义和取值范围限制和影响业务使用。

例：用时间戳 timestamp类型记录当前时间

```
create table login(
id int,
logintime timestamp default CURRENT_TIMESTAMP  %默认值为当前时间
)engine myisam charset utf8;
```

`insert into login (id) values (1),(2),(3);`

Table login：

|  id  |      logintime      |
| :--: | :-----------------: |
|  1   | 2019-08-29 18:04:50 |
|  2   | 2019-08-29 18:04:50 |
|  3   | 2019-08-29 18:04:50 |

时间戳列可以有四种组合定义，其含义分别为：

  1、当字段定义为timestamp，表示该字段在插入和更新时都不会自动设置为当前时间。

  2、当字段定义为timestamp DEFAULT CURRENT_TIMESTAMP，表示该字段仅在插入且未指定值时被赋予当前时间，再更新时且未指定值时不做修改。

  3、当字段定义为timestamp ON UPDATE CURRENT_TIMESTAMP，表示该字段在插入且未指定值时被赋值为"0000-00-00 00:00:00",在更新且未指定值时更新为当前时间。

  4、当字段定义为timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP，表示该字段在插入或更新时未指定值，则被赋值为当前时间。

### 三大列类型练习：某社交网站会员信息

| 列名称    | 列类型                        | 默认值 | 是否主键 |
| --------- | ----------------------------- | ------ | -------- |
| Id        | int unsigned                  |        | PRI      |
| Username  | varchar(20) ---> char(20)     | ''     |          |
| Gender    | char(1)，Tinyint              |        |          |
| Weight    | tinyint unsigned              |        |          |
| Birth     | date                          |        |          |
| Salary    | decimal(8,2)                  |        |          |
| Lastlogin | int unsigned                  |        |          |
| Intro     | varchar(1000) ---> char(1000) |        |          |

优化1：这张表除了username/intro列之外，每一列都是定长的。我们不妨让其所有列，都定长，但是可以极大提高查询速度，用空间换时间。

优化2：username char(20)是会造成空间的浪费，但是提高的速度，值！

Intro char(1500) 却浪费的太多了，另一方面，人的简介，一旦注册完，改的频率也并不高。我们可以把 intro列单独拿出来，另放一张表里。

在开发中，会员的信息优化往往是把频繁用到的信息优先考虑效率，存储到一张表中，

不常用的信息和比较占据空间的信息，优先考虑空间占用，存储到辅表中。建表语法

所谓建表就是一个声明列的过程。

| 列名称   | 列类型        | 默认值 | 是否主键 |
| -------- | ------------- | ------ | -------- |
| Id       | int unsigned  |        | PRI      |
| Username | char(20)      | ''     |          |
| Intro    | varchar(1000) |        |          |



## 增删改查（表数据）

##### 增[insert]删[delete]改[update]查[select]

---

1. **增加**表数据：`insert`

   ```
   例1：插入1行所有列
   
   insert into class
   (Id, Sname, Gender, Company, Salary, Fanbu)
   values
   (1, '张三', '男', '百度', 8888.67, 234);
   ```

   ```
   例2：插入1行部分列Id
   
   insert into class
   (Sname, Gender, Salary)
   Values
   ('李四','男',8750.43)
   
   ⚠️ 由于id是自增型，所以会自动填充；Fanbu默认填充0，因为default 0
   ```

   ```
   例3：插入1行，不声明插入的列，理解为依次插入所有列
   
   insert into class
   values
   (3,'王五','女','新浪',5678.99,125);	
   
   ⚠️ 此时，虽然Id是自增型的，但是仍需填写id（列与值必须按顺序一一对应）
   ```

   ```
   例4：插入多行
   
   insert into class
   values
   (4,'tom','男','腾讯',9999.99,500),
   (5,'jerry','男','华为',9998.33,600);
   ```

2. **修改**表数据：`update `

   ```
   update class
   set
   Gender = '女',
   Company = '千度'
   where id = 1;
   
   ⚠️ where=1或者不加where语句，会修改所有行,也可以接条件表达式
   ```

3. **删除**表数据：`delete`

   ```
   delete from class where Salary > 8000;
   
   ⚠️ 删除只能删除一整行，不能删除一行里的某几列
   ```

4. **查看**表数据：`select`

   ```
   例1：（一般）select 列名（或*） from class where id = 3;
   ```

   ```
   例2：（广义投影）将列作为变量运算，运算结果还能起个别名
   
   select goods_id, (market_price - shop_price) as discount from goods where goods_id < 3;
   ```



## select 5种子句

##### 5种子句写的时候要有严格的顺序：where | group by | having | order by | limit

---

1. **where 条件查询**

   列就是***变量***，在每一行上，列的值都在变化。Where条件是表达式，在哪一行上表达式为真，哪一行就取出来。where子句针对的对象是磁盘上的***数据表***文件去select的，而select出来后的数据是存放在内存中的一个***临时***"结果集"。

   ```
   例1：🆙【模糊查询 like】——“%”：通配任意字符；“_”：通配单一字符
   
   select * from goods where goods_name like '诺基亚%';
   ```

   ```
   例2：查找goods表中本店比市场价格省200以上的商品，差价命名为discount
   
   🙆 select goods_id, goods_name, (market_price - shop_price) as discount from goods where (market_price - shop_price) > 200;` 
   
   🙅 select goods_id, goods_name, (market_price - shop_price) as discount from goods where discount > 200;
   
   ⚠️ where是对表中的数据发挥作用，此时还没有产生discount。
   ```

   ```
   例3：【floor(X)函数：不大于X的最大整数值】有一张表mian和数组(自建)，把num值处于[20,29]之间的改为20，num值处于[30,39]之间的改为30。
   
   update mian set num = floor(num/10)*10 where num >= 20 and num <= 39;
   ```

   ```
   例4：【substring(str,n)：子字符串】查找诺基亚手机，并去前缀’诺基亚‘
   
   select goods_id, goods_name, substring(goods_name,4) from goods where goods_name like '诺基亚%';
   ```

   ```
   例5：【concat：结合两字符串】 查找诺基亚手机，并把’诺基亚‘前缀换成‘HTC‘
   
   select goods_id, goods_name, concat('HTC', substring(goods_name,4)) from goods where goods_name like '诺基亚%';
   
   update goods set goods_name = concat('HTC', substring(goods_name,4)) where goods_name like '诺基亚%';
   ```

   ---

2. **group 分组查询与统计函数**

   ```
   例6：【max，min，sum，avg】
   select max(goods_number) from goods;
   ```

   ```
   例7：【count】
   统计goods表中总商品数(绝对行数)：select count(*)(或count(1)) from goods;
   统计goods_name非null的商品数：select count(goods_name) from goods;
   
   ⚠️ count(*)和count(1)：对于myisam引擎的表，没有区别的。这种引擎内部有一计数器在维护着行数。Innodb的表，用count(*)直接读行数，效率很低，因为innodb真的要去数一遍。
   ```

   ```
   例8：【group 配合统计函数】计算goods表中每个栏目下的库存量之和：
   select cat_id, sum(goods_number) from goods group by cat_id;
   
   ⚠️ group by a, b ,c 为例，则select的列只能在a, b, c 里选择语义上才没有矛盾。
   ```

   ---

3. **having 筛选查询**

   having针对的对象是内存表结构中的***"结果集"***。如果同时写了where和having子句，where子句肯定要写在having子句前面，因为having子句是针对where子句查询出来的结果集来操作的。

   ```
   例9：查询本店价比市场价省的钱，且筛选出省钱200以上的商品（例2的having解法）
   select goods_id, goods_name,(market_price-shop_price) as save from goods having save > 200;
   ```

   ```
   例10：查询每个栏目积压的货款，且筛选出积压货款大于20000的栏目
   select cat_id, (goods_number * shop_price) as remain from goods group by cat_id having remain > 20000;
   ```

   ``` 
   🌟例11：查询挂科数大于2门的学生的平均成绩
   解法一：
   🙆 select name,sum(score<60) as fail,avg(score) from result group by name having fail>=2;
   🙅 select name,count(score<60) as fail,avg(score) from result group by name having fail>=2;
   ⚠️ 无论是count(1)还是count(0)都会被统计
   
   解法二【子查询】：
   思路：Step1. 找出挂科数>=2的人及其挂科数；Step2. 计算他们的平均分
   🙆 select name, avg(score) from result where name in (select name from (select name, count(1) as gks from result where score<60 group by name having gks>=2) as tmp) group by name;
   ```

   ---

4. **order by 排序查询**

   对结果集进行排序 

   `...order by 列1 asc/desc 列2 asc/desc 列3 asc/desc...	%不写默认asc`

   ```
   例12：按价格由高到低排序
   select goods_id,goods_name,shop_price from goods order by shop_price desc;
   ```

   ```
   例13：先按栏目由低到高排序,栏目内部再按价格由高到低排序【有冲突时，顺序决定优先】
   select goods_id,cat_id,goods_name,shop_price from goods order by cat_id asc,shop_price desc;
   ```

   ---

5. **limit 限制条目**

   `...limit [offset] N	%offset：偏移量；N：取出条目	`

   ```
   例14：查询本店价格最高的前3名
   select goods_id,goods_name,shop_price from goods order by shop_price desc limit 3;
   ```

   ```
   例15：取出点击量第3名到前5名的商品
   select goods_id,goods_name,click_count from goods order by click_count desc limit 2,3;
   ```

---

参考文章：[最全的MySQL基础](https://www.cnblogs.com/lms520/p/5427685.html)、[燕十八MySQL-秘籍](https://blog.csdn.net/qixibalei/article/details/54340872)、[燕十八mysql复习](https://blog.csdn.net/z10160/article/details/10610637)