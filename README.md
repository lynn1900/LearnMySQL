# MySQL学习笔记

## 准备工作

1. 环境变量配置(为path路径附加mysql的bin目录)

- 一次性方案：`PATH="$PATH":/usr/local/mysql/bin`
- [永久性方案](https://www.cnblogs.com/wangrui-techbolg/archive/2012/12/22/2829614.html)

2. 修改**密码**：`ALTER USER 'root'@'localhost' IDENTIFIED BY ‘密码’;`
3. 数据库的启动：`net start mysql`（也可以通过系统偏好设置-mysql进行启动）
4. 数据库的连接：`mysql -h localhost -u root -p`
5. 查看用户名和主机：`select user();`
6. 查看版本：`select version();`
7. 清屏：`control + L`
8. 取消执行当前语句：`\c`
9. null：比较null只能用 is null / is not null，不能用 =null / !=null



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

- **改表名**：`rename table 表名 to 新表名;`



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
   例1. 🆙【模糊查询 like】——“%”：通配任意字符；“_”：通配单一字符
   
   select * from goods where goods_name like '诺基亚%';
   ```

   ```
   例2. 查找goods表中本店比市场价格省200以上的商品，差价命名为discount
   
   🙆 select goods_id, goods_name, (market_price - shop_price) as discount from goods where (market_price - shop_price) > 200;` 
   
   🙅 select goods_id, goods_name, (market_price - shop_price) as discount from goods where discount > 200;
   
   ⚠️ where是对表中的数据发挥作用，此时还没有产生discount。
   ```

   ```
   例3.【floor(X)函数：不大于X的最大整数值】有一张表mian和数组(自建)，把num值处于[20,29]之间的改为20，num值处于[30,39]之间的改为30。
   
   update mian set num = floor(num/10)*10 where num >= 20 and num <= 39;
   ```

   ```
   例4.【substring(str,n)：子字符串】查找诺基亚手机，并去前缀’诺基亚‘
   
   select goods_id, goods_name, substring(goods_name,4) from goods where goods_name like '诺基亚%';
   ```

   ```
   例5.【concat：结合两字符串】 查找诺基亚手机，并把’诺基亚‘前缀换成‘HTC‘
   
   select goods_id, goods_name, concat('HTC', substring(goods_name,4)) from goods where goods_name like '诺基亚%';
   
   update goods set goods_name = concat('HTC', substring(goods_name,4)) where goods_name like '诺基亚%';
   ```

   ---

2. **group by 分组查询与统计函数**

   ```
   例6.【max，min，sum，avg】
   select max(goods_number) from goods;
   ```

   ```
   例7.【count】
   统计goods表中总商品数(绝对行数)：select count(*)(或count(1)) from goods;
   统计goods_name非null的商品数：select count(goods_name) from goods;
   
   ⚠️ count(*)和count(1)：对于myisam引擎的表，没有区别的。这种引擎内部有一计数器在维护着行数。Innodb的表，用count(*)直接读行数，效率很低，因为innodb真的要去数一遍。
   ```

   ```
   例8.【group 配合统计函数】计算goods表中每个栏目下的库存量之和：
   select cat_id, sum(goods_number) from goods group by cat_id;
   
   ⚠️ group by a, b ,c 为例，则select的列只能在a, b, c 里选择语义上才没有矛盾。
   ```

   ---

3. **having 筛选查询**

   having针对的对象是内存表结构中的***"结果集"***。如果同时写了where和having子句，where子句肯定要写在having子句前面，因为having子句是针对where子句查询出来的结果集来操作的。

   ```
   例9. 查询本店价比市场价省的钱，且筛选出省钱200以上的商品（例2的having解法）
   select goods_id, goods_name,(market_price-shop_price) as save from goods having save > 200;
   ```

   ```
   例10. 查询每个栏目积压的货款，且筛选出积压货款大于20000的栏目
   select cat_id, (goods_number * shop_price) as remain from goods group by cat_id having remain > 20000;
   ```

   ``` 
   🌟例11. 查询挂科数大于2门的学生的平均成绩
   
   🙆 select name,sum(score<60) as fail,avg(score) from result group by name having fail>=2;
   🙅 select name,count(score<60) as fail,avg(score) from result group by name having fail>=2;
   
   ⚠️ 无论是count(1)还是count(0)都会被统计
   ```
   
   ---
   
4. **order by 排序查询**

   对结果集进行排序 

   `...order by 列1 asc/desc 列2 asc/desc 列3 asc/desc...	%不写默认asc`

   ```
   例12. 按价格由高到低排序
   select goods_id,goods_name,shop_price from goods order by shop_price desc;
   ```

   ```
   例13. 先按栏目由低到高排序,栏目内部再按价格由高到低排序【有冲突时，顺序决定优先】
   select goods_id,cat_id,goods_name,shop_price from goods order by cat_id asc,shop_price desc;
   ```

   ---

5. **limit 限制条目**

   `...limit [偏移量] N		` or  `...limit N offset 偏移量`

   ```
   例14. 查询本店价格最高的前3名
   select goods_id,goods_name,shop_price from goods order by shop_price desc limit 3;
   ```

   ```
   例15. 取出点击量第3名到前5名的商品
   select goods_id,goods_name,click_count from goods order by click_count desc limit 2,3;
   ```



## 子查询

1. **where型子查询**

   ```
   例1. 查询最新(goods_id最大)的商品
   
   select goods_id,goods_name from goods where goods_id = (select max(goods_id) from goods);
   ```

   ```
   例2. 查询每个栏目下goods_id最大的商品
   
   select cat_id,goods_id,goods_name from goods where goods_id in (select max(goods_id) from goods group by cat_id);
   ```

2. **from型子查询**

   内层sql的查询结果，当成一张临时表，供外层sql再次查询。

   ```
   🌟例3. 查询挂科数大于2门的学生的平均成绩（上一节例13的子查询方法）
   【where、from型子查询】：
   思路：Step1. 找出挂科数>=2的人及其挂科数；Step2. 计算他们的平均分
   
   🙆 select name, avg(score) from result where name in (select name from (select name, count(1) as gks from result where score<60 group by name having gks>=2) as tmp) group by name;
   ```

3. **exists型子查询**

   把外层sql的结果，拿到内层sql去测试，如果内层sql成立，则该行取出。
   
   ```
    例4. 查出有商品的栏目
     
   select * from category where exists (select * from goods where goods.cat_id=category.cat_id);
   ```



## 数据联查

1. **全相乘**

   ```
   例1. 查询商品及其栏目名
   
   select goods_id, goods_name, goods.cat_id, category.cat_id, cat_name from goods, category where goods.cat_id = category.cat_id;
   
   ⚠️当某一个列名在两个表中都存在时，要注明是哪一个表中的
   ```

2. **左连接**

   假设A表在左，不动，B表在A表的右边滑动，A表与B表通过一个关系关系来筛选B表的行。

   语法： `A left join B on 条件`

   这一块，形成的也是一个结果集，可以看成一张表，设为C，既如此，可以对C表做查询。

   问：C表可以查询的列有哪些列？	答：A、B的列都可以查

   ```
   例2. 查询商品及其栏目名(例1的左连接解法)
   
   思路：C=[goods left join category on goods.cat_id = category.cat_id]
   select goods_id,goods_name,cat_name from C；
   
   🙆 select goods_id,goods_name,cat_name from goods left join category on goods.cat_id=category.cat_id;
   ```

   ```
   例3. 取出第4个栏目下的商品，以其栏目名
   
   select goods_id,goods_name,cat_name from goods left join category on goods.cat_id = category.cat_id where goods.cat_id = 4;
   
   ⚠️ 此例说明左连接之后还能用where等字句
   ```

   ```
   例4.（配对问题）
   
   1、所有男生带上配偶上舞台，没配偶的拿牌子写null
   select boy.*, girl.* from boy left join girl on boy.link = girl.link;
   
   2、所有女生带上配偶上舞台，没配偶的拿牌子写null
   select boy.*, girl.* from girl left join boy on boy.link = girl.link;
   ```

   ```
   🌟例5. 已知m表给出了主客场战队id和比赛结果，t表给出了战队id对应的战队名，根据t表补充战队名（两次左连接）
   
   select hid,t1.tname,mres,gid,t2.tname,matime from (m left join t as t1 on m.hid = t1.tid) left join t as t2 on m.gid = t2.tid;
   ```

3. **右连接**

   和左连接一回事，`A left join B on 条件` 等价于 `B right join A on 条件`，一般用左连接。

4. **内连接**

   内连接是左右连接的交集，语法：`A inner join B on 条件`

   ```
   例6.（配对问题）所有配偶在场的男生女生带上配偶上舞台
   
   select boy.*, girl.* from girl inner join boy on boy.link = girl.link;
   ```

5. **外连接**

   外连接是左右连接的并集，但是mysql中不支持外连接语法(outer join)，可以用其他方式**(union)**替代解决。

   union语法：`A union B`

   问：能否从2张表查询union？——可以，union合并的是“结果集”，不区分来自哪一张表。
   
   进一步：取自2张表，可以通过别名让2个结果集的列一致，那么如果取出的结果集列名不一样，还能否union？——可以，以**第一个结果集**中的列名为准。

   再进一步：union什么情况下可以用？——只要结果集的**列数一样**即可（即使列类型不一样）。
   
   ```
   例7. 查出价格低于100元和价格高于4000元的商品（不能用or）
   
   select goods_id,goods_name,shop_price from goods where shop_price < 30 union select goods_id,goods_name,shop_price from goods where shop_price > 4000;
   ```
   
   ```
   例8. 查出第三个栏目下价格前3高和第四个栏目下价格前2高的商品
   
   (select goods_id,goods_name,cat_id,shop_price from goods where cat_id = 3 order by shop_price desc limit 3) union (select goods_id,goods_name,cat_id,shop_price from goods where cat_id = 4 order by shop_price desc limit 2);
   
   ⚠️ 如果没有limit，内层order by无意义，会被优化掉；有limit时，order by影响结果集，不会被优化掉。
   ```
   
   **union对重复行的处理：**默认回去重。如果不想去重，用**union all**代替union。
   



## 数学函数

1. abs

2. bin：二进制；hex：十六进制

3. floor：抹掉零头取整

4. rand：随机生成[0,1]之间的数

   ```
   例1.随机生成[5,15]之间的数
   
   select rand()*10+5;
   ```

5. group_concat

   ```
   例2.把cat_id为4的商品id拼接起来
   
   select group_concat(goods_id) from goods where cat_id=4;
   ```



## 字符串函数

1. ascii

2. length：字节长度；char_length：字符数

3. reverse：反转字符串

4. position(s1 in s)：从字符串 s 中获取 s1 的开始位置	

5. right(s,n)：返回字符串 s 的后 n 个字符

   ```
   例1. 查询邮箱后缀
   
   select *,right(email,length(email)-position('@' in email)) from information;
   ```

   

##  日期和时间函数

1. now：当前日期时间

2. curdate/current_date：当前日期

3. curtime/current_time：当前时间

4. dayofweek(注意周日是第一天)、dayofmonth、dayofyear

5. week(curdate())：计算今天是今年第几周

   ```
   例1. 按周计算加班时间
   
   Select week(dt) as week_number, sum(overtime) from jiaban group by week_number;
   ```



## 控制流函数

```
例1. 判断性别

select *, case gender when 1 then '男' when 2 then '女' else '妖' end from gender;

⚠️ case 变量 when 某种情况 then 返回值 when 另一种情况 then 返回值 else 默认值 end；
```

```
例2. 判断性别，让女士优先

select name, if (gender=2,'优先','等待') as vip from gender;

⚠️ if(exp1,exp2,exp3): 若exp1成立则返回exp2，否则exp3。
⚠️ ifnull(exp1，exp2): 若exp1非null则返回exp1，否则exp2。
```



## 视图

1. 什么是视图？

   view 又称虚拟表，view其实就一条查询SQL语句的结果集==>将常用的SQL查询结果集虚拟为一张表存放在内存中

2. 视图有什么用？

- **权限控制：**某几个列允许用户查询，而其他列不允许，可以通过视图开放其中的一部分列，达到权限的控制
- **简化查询**

3. 视图语法

- 创建视图

  ```
  CREATE VIEW view_name AS
  SELECT column_name(s)
  FROM table_name
  WHERE condition;
  
  ⚠️ 视图总是显示最新的数据！每当用户查询视图时，数据库引擎通过使用视图的SQL语句重建数据。
  ```

  ```
  例1. 小说站1000万篇小说分成article1、...、article5 5张表，每张表存储200万篇小说，创建视图合并所有小说。
  
  create view articles as
  select title from article1 union select title from article2 ... select title from article5; 
  ```

- 更新视图

  ```
  CREATE OR REPLACE VIEW view_name AS
  SELECT column_name(s)
  FROM table_name
  WHERE condition;
  ```

- 修改视图数据

  视图【虚拟表】是物理表的一个"投影"，两者是相互影响的。更改物理表，虚拟表也会更改；同理，更改虚拟表，物理表也会更改！

  视图某种情况下是可以修改的：**物理表和虚拟表的列能一一对应**。

  ⚠️ 视图增删改也会影响表

- 撤销视图

  ```
  DROP VIEW view_name;
  ```




## 字符集与乱码问题

1. 字符集

   **GB2312编码**：1981年5月1日发布的简体中文汉字编码国家标准。GB2312对汉字采用**双字节**编码，收录7445个图形字符，其中包括6763个汉字。

   **BIG5编码**：台湾地区繁体中文标准字符集，采用双字节编码，共收录13053个中文字，1984年实施。

   **GBK编码**：1995年12月发布的汉字编码国家标准，是对GB2312编码的扩充，对汉字采用**双字节**编码。GBK字符集共收录21003个汉字，包含国家标准GB13000-1中的全部中日韩汉字，和BIG5编码中的所有汉字。

   **Unicode编码**：国际标准字符集，Unicode包含了全世界所有的字符。Unicode最多可以保存4个字节容量的字符。也就是说，要区分每个字符，每个字符的地址需要4个字节。这是十分浪费存储空间的，于是，程序员就设计了几种字符编码方式，比如：**UTF-8,UTF-16,UTF-32**。

2. 为什么会乱码？

- 原因一：解码时与实际编码不一致（可修复）
- 原因二：传输过程中，解码不一致，导致字节丢失（不可修复）

3. 怎么能不乱码？

   客户端[GBK]--不转-->连接器[GBK]--转UTF8-->服务器[UTF8]

   客户端[GBK]--转UTF8-->连接器[UTF8]--不转-->服务器[UTF8]

   服务器[UTF8存放数据]-->连接器处理[转换为客户端字符集]-->客户端[GBK显示数据]

   ⚠️声明字符集：正确指定客户端的编码-合理选择连接器的编码-正确指定返回内容的编码

   客户端：`set character_set_client = gbk/utf8;`【谁连接服务器谁就是客户端，客户端字符集是多变的】

   连接器：`set character_set_connection = gbk/utf8;`

   服务器：`set character_set_results= gbk/utf8;`

   三者一致时（三合一）：`set names gbk/utf8;` 

   

参考文章：[最全的MySQL基础](https://www.cnblogs.com/lms520/p/5427685.html)、[燕十八MySQL-秘籍](https://blog.csdn.net/qixibalei/article/details/54340872)、[燕十八mysql复习](https://blog.csdn.net/z10160/article/details/10610637)