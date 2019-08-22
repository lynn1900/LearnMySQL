# MySQL学习笔记

## 准备工作

1. 环境变量配置(为path路径附加mysql的bin目录)

- 一次性方案：`PATH="$PATH":/usr/local/mysql/bin`
- [永久性方案](https://www.cnblogs.com/wangrui-techbolg/archive/2012/12/22/2829614.html)

2. 修改**密码**：`ALTER USER 'root'@'localhost' IDENTIFIED BY ‘密码’;`



## 入门语句

1. 数据库的连接：`mysql -uroot -p`
2. 库级操作

- **显示**数据库：`show databases;`
- **选择**数据库：`use dbName;`
- **创建**数据库：`create database dbName charset utf8;`
- **删除**数据库：`drop database dbname;`

3. 表级操作

- **显示**表：`show tables;`

- **创建**表：

  ```
  create table tableName (
  列名称1　列类型　[列参数]　[not null default ],
  ....列2...
  ....
  列名称N　列类型　[列参数]　[not null default ]
  )engine myisam/innodb charset utf8/gbk
  ```

  - 例

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

- **删除**表内容：`truncate tableName;`

- **删除**表：`drop table tableName;`

- **查看**表结构：`desc tableName;`



## 增删改查（表数据）

1. **增加**表数据：`insert`

例1：插入1行所有列

```
insert into class
(Id, Sname, Gender, Company, Salary, Fanbu)
values
(1, '张三', '男', '百度', 8888.67, 234);
```

例2：插入1行部分列Id

```
insert into class
(Sname, Gender, Salary)
Values
('李四','男',8750.43)
```

注：由于id是自增型，所以会自动填充；Fanbu默认填充0，因为default 0

例3：插入1行，不声明插入的列，理解为依次插入所有列

```
insert into class
values
(3,'王五','女','新浪',5678.99,125);		%此时，虽然Id是自增型的，但是仍需填写id（列与值必须按顺序一一对应）
```

例4：插入多行

```
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
where id = 1;				% where=1或者不加where语句，会修改所有行
```

注：where后可以是其他表达式，如Sname = '张三'  and Salary > 5000.00。



3. **删除**表数据：`delete`（删除只能删除一整行，不能删除一行里的某几列）

```
delete from class where Salary > 8000;
```

4. **查看**表数据：`select`

例1：`select * from class;`（所有行所有列）

例2：`select * from class where Id > 3;`（指定行所有列）

例3：`select Sname, Company, Salary from class;`（所有行指定列）

例4：`select Sname, Company, Salary from class where id = 3;`（指定行指定列）



## 三大列类型

### 数值型

- **整型：Tinyint/ smallint/ mediumint/int/ bigint**

  - **整型所占字节与存储范围：**占字节越多,存储范围越大。

  |   类型    | 字节 | 位数 | 存储范围(无符号) |  存储范围（带符号）  |
  | :-------: | :--: | :--: | :--------------: | :------------------: |
  |  tinyint  |  1   |  8   |      0—255       |       -128—127       |
  | smallint  |  2   |  16  |     0—65535      |     -32768—32767     |
  | mediumint |  3   |  24  |       ...        |         ...          |
  |    Int    |  4   |  32  |       ...        |         ...          |
  |  bigint   |  8   |  64  |                  |                      |
  |  一般地   |  n   |  8n  |     0—2^8n-1     | -2^(8n-1)—2^(8n-1)-1 |

  涉及**二进制补码**问题：最高位的0/1表示符号，0表示正，1表示负。

  以tinyint为例，正数 = 绝对值位，负数= 绝对值位 - 128。如正数：0000 0000 ---> 0，0111 1111 --->127，负数：1000 0000 ---> -128，1111 1111 --->-1

  - **整型系统的可选参数 :** XXint(M) unsigned zerofill

  例：`age tinyint(4) unsigned`、`stunum smallint(6) zerofill`

  `Unsigned`：代表此列为无符号类型， 范围从0开始；不加unsinged， 则该列默认是有符号类型,范围从负数开始。

  `Zerofill`：代表0填充，即：如果该数字不足参数M位，则自动补0，补够M位。如果没有zerofill属性，单独的参数M，没有任何意义。如果设置某列为zerofill，则该列已经默认为 unsigned，无符号类型。

- **小数型：浮点型：float(M,D)、定点型：decimal(M,D)**

  M是"精度"，代表总位数；D是"标度"，代表小数位。如float(6,2)表示-9999.99—9999.99，如果加了unsigned，则是0—9999.99。

  - **float和decimal空间上的区别：**float能存10^38，10^-38。如果 M<=24，占4个字节，24 <M <=53，8个字节；decimal () ，变长字节。decimal比float精度更高，适合存储货币等要求精确的数字。

### 字符串型

- 定长字符串：Char(M)

  M 代表宽度， 0<=M<=255之间。如果实际存储内容不足M个，则后面加空格补齐。取出来的时候，再把后面的空格去掉。char(5)下 'aa ' ---> 'aa   ' ---> 'aa'

  例：char(5)，输入 'aa ' ---> 存储 'aa   ' ---> 取出 'aa'。

- 变长字符串：Varchar(M)

  M代表宽度，0<=M<=65535（以ascii字符为例，utf822000左右）。不用空格补齐，但列内容前有1～2个字节来标志该列的内容长度。

  **char和varchar比较：**好比于坐车，一种是统一票价，提高了效率，省了售票员；另一种是根据站数区分票价，需要一个售票员，专门记录客户的站数。

  |  类型   | 宽度 | 可存 | 实存 |     实占空间     |     利用率     |
  | :-----: | :--: | :--: | :--: | :--------------: | :------------: |
  |  char   |  M   |  M   |  i   |        M         |   i/M<=100%    |
  | varchar |  M   |  M   |  i   | i 字符+(1~2)字节 | i/(i+1~2)<100% |

  注意: char(M)，varchar(M)限制的是字符，不是字节。即 char(2) charset utf8，能存2个utf8字符，比如'中国'。

  **char与varchar型的选择原则：**

  1、考虑空间利用效率

  四字成语表，char(4), 

  个人简介，微博140字，varchar(140)

  2、考虑速度

  用户名: char 

- Text 文本类型：可以存比较大的文本段，搜索速度稍慢。

  不用加默认值(加了也没用)

- Blob二进制类型：用来存储图像，音频等二进制信息。

  注意：当输入文本类型时，不会报错，意义在于防止因为字符集的问题，导致信息丢失。比如:一张图片中有0xFF字节，这个在ascii字符集认为非法，在入库的时候，被过滤了。

### 日期时间类型



