---
title: mysql进阶sql用法（二）
date: 2019-10-10 17:47:47
tags: mysql
---
&#160;&#160;&#160;&#160;&#160;&#160;

### JSON
#### 创建JSON项:
```
create table emp_details( emp_no int primary key, details json );
```

#### 插入JSON
```
insert into emp_details(emp_no,details) values(  '4', '{"location":"IN","phone":"+11800000000","email":"abc@example.com",                                "address":{"line1":"abc","line2":"xyz street","city":"Banalore",                "pin":"560103"}}' );
```

#### 检索JSON
```
select emp_no,details->'$.address.pin' pin from emp_details;
```
#### pretty打印
```
select emp_no ,json_pretty(details) from emp_details;
```
<!--more-->
#### JSON查找

```
1.使用 where

select emp_no from emp_details where details->>'$.address.pin'="560103";

2.使用JSON_CONTAINS

select json_contains(details->> '$.address.pin', "560103") from emp_details;

```
#### 检查JSON字段名是否存在
```
select json_contains_path(details,'one',"$.address.line1") from emp_details;

one表示至少存在一个键

select json_contains_path(details,'all',"$.address.line1","$.address.line5") from emp_details;

all表示全部存在

```

#### 修改JSON内容
```
JSON_SET(): 替换现在又的值，如果key不存在则新增key并添加值

update emp_details set details = json_set(details, "$.address.pin", "56100", "$.nickname", "kai") where emp_no = 4;

JSON_INSERT():插入新的值，但不替换现有值

update emp_details set details=json_insert(details, "$.address.pin", "560132", "$.address.lin4", "A Wing") where emp_no = 4;

JSON_REPLACE(): 仅替换现有值

update emp_details set details=json_replace(details, "$.address.pin", "560132", "$.address.lin5", "Landmark")  where emp_no =4;


```


#### 其他：
```
获取JSON 所有的key:

select json_keys(details) from emp_details where emp_no=4;

给出JSON 文档中所有的键:

select json_length(details) from emp_details where emp_no =4;
```

### CTE(common table expression) -- 公共表表达
#### CTE非递归查询
```

将同一张表需要查询两次变成查询一次即可

with cte as (select year(from_date) as year, sum(salary) as sum from salaries group by year) select q1.year, q2.year as next_year, q1.sum as sum, q2.sum as next_sum, 100*(q2.sum - q1.sum)/q1.sum as pct from cte as q1, cte as q2 where q1.year = q2.year-1;
```
#### CTE递归查询:
```
递归查询分为两步，首先找根节点(seed),通过关键字UNION [ALL] 或者 UNION DISTINCT 进行分离


with recursive employee_paths (id,name,path) as(select id, name, cast(id as char(200) )from employees_mgr where manager_id is null union all select e.id, e.name, concat(ep.path, ',', e.id) from employee_paths as ep join employees_mgr as e on ep.id = e.manager_id ) select * from employee_paths order by path;
```

#### 生成列
```
创建生成列：

create table employeeFN (emp_no int(11) not null, birth_date date not null, first_name varchar(14) not null, last_name varchar(16) not null, gender enum('M','F') not null, hire_date date not null, full_name varchar(30) as (concat(first_name, ' ', last_name)), primary key (emp_no) ,key name (first_name,last_name) )engine=InnoDB DEFAULT CHARSET=utf8mb4;

插入数据：

insert into employeeFN (emp_no, birth_date, first_name, last_name, gender, hire_date) values (123456, '1987-10-02', 'ABC', 'XYZ', 'F', '2008-07-28');
```

#### 窗口函数
```
alter table employees add hire_date_year year as (year(hire_date)) virtual;

无子区分的情况:

select concat (first_name, " ", last_name) as full_name, salary, row_number() over(order by salary desc) as 'Rank' from employees join salaries on salaries.emp_no=employees.emp_no limit 10;

有子区分的情况:

select hire_date_year, salary, row_number() over(partition by hire_date_year order by salary desc) as 'Rank' from employees join salaries on salaries.emp_no=employees.emp_no order by salary desc limit 10;

```

#### 命名窗口
```
将窗口函数命名后，可以重复调用：

select hire_date_year, salary, rank() over w as 'Rank', first_value(salary) over w as 'first', nth_value(salary,3) over w as 'third', last_value(salary) over w as 'last' from employees join salaries on salaries.emp_no = employees.emp_no window w as (partition by hire_date_year order by salary desc )order by salary desc limit 10;

```
 
