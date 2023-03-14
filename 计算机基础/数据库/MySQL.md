# 登录数据库

```mys
mysql -u root -p
```



# 查询所有的数据库

```mysql
show datdabases;
```



# 切换到某个数据库

```mysql
use mydb;
```



# 查看该数据库的所有表

```mysql
show tables;
```



# 查看该表有哪些列

例如使用 user 表

```mysql
show columns from user;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302163259.png)

# SELECT



## 检索数据

```mysql
select username from user;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302163330.png)

## 检索多个列

```mysql
select username, passwd from user;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302163506.png)

## 检索所有的列

```mysql
select * from user;
```

## 检索不同的行

distinct 关键字，只返回不同的行

```mysql
select distinct username from user;
```

## 限制结果

使用 `limit` 字句，限制输出的行数

```mysql
# 只显示 5 行
select username from user limit 5;

# 从第 5 行之后的 5 行
select username from user limit 5,5;
```



# ORDER（排序）

升序（从 A -> Z）

```mysql
SELECT username, passwd
FROM user
ORDER BY username;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302174519.png)



降序（从 Z -> A）

```mysql
SELECT username, passwd
FROM user
ORDER BY username DESC;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302174549.png)



## 多个列排序

```mysql
SELECT username, passwd
FROM user
ORDER BY username DESC, passwd;
```



# WHERE

```mysql
SELECT username, passwd
FROM user
WHERE username = 'abc';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303194729.png)



| 操作符  | 说明               |
| ------- | ------------------ |
| BETWEEN | 在指定的两个值之间 |



```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303200646.png)



## and

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE vend_id = 1004 AND prod_price <= 10;
```

![image-20230303200859644](/home/vaesong/.config/Typora/typora-user-images/image-20230303200859644.png)



## or

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE vend_id = 1004 OR vend_id = 1003;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303201053.png)

## IN

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE vend_id IN (1001, 1006)
ORDER BY prod_name;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303201431.png)



## NOT

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE vend_id NOT IN (1001, 1006)
ORDER BY prod_name;
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303201536.png)



# LIKE 通配符

## %

表示任何字符出现的任何次数，和别的语法里常用的 * 是一样的吧

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE 'jet%';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303202015.png)



## _

下划线  _  只能匹配单个字符

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name LIKE '_ ton%';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303202208.png)



# REGEXP 正则表达式

正则表达式中， . 代表着匹配任意字符

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '.anvil';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303202716.png)

## |

这里一定要注意，写成 '1000 | 2000' 只有 2000 的结果，而不会有 1000 的结果，因为把 1000 后面的空格算上了

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '1000|2000';
```

![image-20230303203228885](/home/vaesong/.config/Typora/typora-user-images/image-20230303203228885.png)



## 匹配多个字符

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '[123] Ton';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303203320.png)

## 匹配范围

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '[1-6] Ton';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303203636.png)



## 匹配特殊字符

使用 `\\` 做引导

```mysql
SELECT vend_id, prod_name, prod_price
FROM products
WHERE prod_name REGEXP '\\.';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230303203759.png)























# INSERT

## 数据插入

```mysql
INSERT INTO 表名(key, key)
VALUES('value', 'value'), ('value', 'value');
```



```mysql
insert into user values('hrj','123');

insert into user(username, passwd) values('tianyu', '1q2w3e');

insert into user(username, passwd) values('abc', 'abc'), ('qwe', '678');
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302165620.png)

## 插入检索出的数据

```mysql 
insert into client(username, passwd) select username, passwd from user;
```



# UPDATE

```mysql
UPDATE 表名
SET KEY = ‘VALUE’
WHERE KEY = ‘VALUE’;
```



```mysql
update user
set passwd = 'hrj', 
where username = 'hrj';
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230302165633.png)

# DELETE

```mysql
DELETE FROM 表名
WHERE key = 'value';
```



```mysql
DELETE FROM user
WHERE username = 'hrj';
```



# TABLE

## 创建表

```mysql
create table client(username char(50) not null, passwd char(50) not null,primary key (username)) ENGINE=InnoDB;
```



## 更新表

添加一列

```mysql
ALTER TABLE 表名
ADD phone_num CHAR(12);
```



删除一列

```mysql
ALTER TABLE 表名
DROP COLUMN phone_num;
```



## 删除表

```mysql
DROP TABLE 表名;
```



## 重命名表

```mysql
RENAME TABLE client TO cl;
```



