## 数据库分类
- 关系型数据库
- 非关系型数据库

## 数据库类型
- 关系型数据库
  - mysql
  - sql sever
  - Oracle
- 非关系型数据库
  - mysql（网站数据库首选）

### 数据库语言
- DDl（Definition）：数据库定义语言，用来定义数据库对象：库、表、列
  - 建库语句：`create database day0313`
  - 使用库：`use day0313`
  - 建表语句：   
  ```database
  create table student(
      s_numbel  varchar(6),
      s_name    varchar(20),
      s_age     int,
      s_gender  varchar(10)
  )
  ```
  - 修改表名：`alert table 旧表名 rename 【to】 新表名`
  - 修改字段数据类型：`alert table 表名 modify 字段名 数据类型`
  - 修改字段名称：`alert table 表名 change 旧字段名 新字段名 新数据类型`
  - 增加字段名：`alert table 表名 add 新字段名 数据类型 【约束条件】 【first|after 已存在字段名】`

  - 查看表：`desc 表名`
  - 删除表：`drop table 表名`
  - 删除字段名：`alert table 表名 drop 字段名`

  - 字段名的约束
  > 主键约束：`primary key`   
  > 自增约束:`primary key auto_increment`   
  > 外键表:`foreign key（字段名） references 主表名（主键字段） `   
  > 非空约束：`not null`
  > 唯一性约束：`unique`
  > 默认约束：`default 值`

- DML(Mainpulation):数据库操作语言，定义数据库记录
- DCL（Control）:数据库控制语言，设置安全级别
- DQL（Query）:查询语言，用来查询记录
  - 查询所有字段：`select * from 表名`
  - 查询输出指定字段：`select 字段名 from 表名`
  - 条件查询：`select  * from 表名 where s_age < 15`
`select  * from 表名 where s_age between 15 and 45`
  - 多条件查询：`select  * from 表名 where s_name=‘张三’ and s_age<15`
`select  * from 字段名 where age<15 or `
