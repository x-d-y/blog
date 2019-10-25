---
title: mysql常用命令(一)
date: 2019-09-28 11:29:03
tags: mysql
---
&#160;&#160;&#160;&#160;&#160;&#160;

### 数据库操作

#### 创建数据库：
```
CREATE DATABASE <databaseName>
```

#### 查看数据库：
```
SHOW DATABASES;
```
#### 查看数据文件存放目录：
```
SHOW VARIABLES LIKE 'datadir';
```
<!--more-->
### 数据表操作

#### 创建数据表：
```
CREATE TABLE IF NOT EXISTS <tableName> (
  id int AUTO_INCREMENT PRIMARY KEY
) ENGINE=<databaseEngin>;
```

#### 查看数据表：
```
SHOW TABLES;
```

#### 查看表结构：
```
SHOW CREATE TABLE <tableName>\G
或
DESC <tableName>;
```
#### 克隆表结构：
```
CREATE TABLE <new table name> LIKE <old table name>
```
#### 删除表:
```
DROP TABLE <tableName>
```

### 行数据操作

#### 插入行：
```
INSERT IGNORE INTO <tableName> (<column1, column2 ... ...>)
VALUES
(<row1>),(<row2>),(... ...);
```
#### 更新行:
```
UPDATE <tableName> SET <column1> = <value1>, <column2> = <value2> where id = <id_value>;
```
#### 删除行：
```
DELETE FROM <tableName> WHERE id = <id_value> AND <column1> = <value1>;
```

#### 替换行：
```
如果replace的行已经存在，则会先删除该行，然后再删除该行

REPLACE INTO <tableName> VALUES (row);
```

#### 更改行的重复项:
```
如果该行不存在，则创建，如果存在则直接修改 on duplicate key update所在值

INSERT INTO <tablename> VALUES(row) ON DUPLICATE KEY UPDATE <column> = <column> + VALUES(<column>);
```

#### 删除所有行：
```
TRUNCATE TABLE <tableNme>;
```
#### 查询行所有columns:
```
SELECT * FROM <tableName>
```

#### 查询指定的列：
```
SELECT <colum1>,<colum2> FROM <tableName>
```

#### 按条件查询指定列：
```
SELECT <colum> FROM <tableName> WHERE <colum1> = <value1> AND <colum2>= <value2>;
```

#### 按条件筛选之一：
```
SELECT * FROM <tableName> WHERE <column> IN (<value1>,<value2>,... ...);

例:
select * from employees where last_name in('Christ', 'Lamba', 'Baba'); 
```
#### 按条件筛选范围
```
SELECT * FROM <tableName> WHERE <column> BETWEEN <value1> AND <value2>;

例:
select count(*) from employees where hire_date between '1986-12-01' and '1986-12-31';
```

#### 否定条件筛选
```
SELECT * FROM <tableName> WHERE <column> NOT ... ...;

例:
select count(*) from employees where hire_date not between '1986-12-01' and '1986-12-31';
```

#### 匹配查询
```
%为多个通配符，表示任意多个字符串
_为精确数量个通配符,表示一个任意字符串

SELECT * FROM <tableName> WHERE <column> LIKE <_value%>

例:
select count(*) from employees where first_name like 'christ%';

select count(*) from employees where first_name like '__ka%';

```

#### 限制查询数量:
```
SELECT * FROM <tableName> WHERE <column> = <value> LIMIT <number>;

例:
select * from employees where hire_date< '1986-01-01' limit 10;
```

#### 查询条数:
```
SELECT COUNT(*) FROM <tableName>;
```

### 查询结果操作

#### 排序查询:

```
SELECT * FROM <tableName> ORDER BY <column> <DESC/ASC>;

例:
select * from salaries order by salary desc;

select * from salaries order by salary asc;

```
#### 分组操作-COUNT:
```
SELECT * FROM <tableName> GROUP BY <column>;

例:
select gender,count(*) as count from employees group by gender;

select first_name, count(*) as count from employees group by first_name order by count desc limit 10;

```
#### 分组操作-AVG：
```
SELECT SUM(<column>) FROM <tableName> GROUP BY <column>;

例:
select year(from_date), sum(salary) as sum from salaries group by year(from_date) order by sum desc;

```
### 过滤操作

#### DISTINCT:
```
SELECT DISTINCT <column> FROM <tableName>;

select distinct title from titles;
```

#### HAVING
```
SELECE <column> from <tableName> HAVING <key> > <value>;

select emp_no, avg(salary) as avg from salaries group by emp_no having avg >140000 order by avg desc;

```

### 多表联合查询

#### JOIN AND

```
SELECT <column> FROM <tableName1> JOIN <tableName2> ON <condition1> JOIN <tableName3> ON <conditon2> AND <conditon3>

select emp.emp_no, emp.first_name, emp.last_name, dept.dept_name from employees as emp join dept_manager as dept_mgr on emp.emp_no=dept_mgr.emp_no and emp.emp_no=110022 join departments as dept on dept_mgr.dept_no=dept.dept_no;

select dept_name, avg(salary) as avg_salary from salaries join dept_emp on salaries.emp_no=dept_emp.emp_no join departments on dept_emp.dept_no=departments.dept_no group by departments.dept_no order by avg_salary desc;

select emp1.* from employees emp1 join employees emp2 on emp1.first_name=emp2.first_name and emp1.last_name=emp2.last_name and emp1.gender = emp2.gender and emp1.hire_date= emp2.hire_date and emp1.emp_no!=emp2.emp_no order by first_name,last_name;


select tt.emp_no, emp.first_name, emp.last_name from titles tt join employees emp on emp.emp_no=tt.emp_no where tt.title="Senior Engineer" and from_date="1986-06-26";
```

#### 使用子语句查询

```
SELECT * FROM <tableName> WHERE <conditon=(SQLCODE)>

select first_name, last_name from employees where emp_no in (select emp_no from titles where title="Senior Engineer" and from_date="1986-06-26");

select emp_no from salaries where salary=(select max(salary) from salaries);

```









