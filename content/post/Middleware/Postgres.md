---
title: Postgres 高阶用法
cover: https://pic4.zhimg.com/80/v2-af80b26c6bd5a71eba219eb1e10b579b_720w.jpg
---

# Postgres 高阶用法

## 数据篇

生成100条数据

```postgresql

insert into scores(chinese, math, english)
select random() * 100, random() * 100, random() * 100
from generate_series(1, 100);
```

## JSON篇

查询出json形式的若干条数据

```postgresql
select row_to_json(out1) as response
from (
  select (
    select jsonb_agg(s)
    from (select * from scores,
                        students as s1
    where s1.class_id = s2.class_id) as s) as items,
         avg(val),
         class_id
      from scores,
           students as s2
  where student_id = s2.id
  group by class_id) as out1;

```

同上，但是把多条数据组装成一条返回了

```postgresql

select jsonb_agg(r.response) from (
  select row_to_json(out1) as response
  from (
    select (
      select jsonb_agg(s)
      from (
        select * from scores,
                      students as s1
        where s1.class_id = s2.class_id) as s) as items,
           avg(val),
           class_id
    from scores,
         students as s2
    where student_id = s2.id
    group by class_id) as out1) r;


-- 方式二 稍微简化了
select array_to_json(array_agg(r))
from (
  select (
    select array_agg(s)
    from (select s1.*
    from scores,
         students as s1
    where s1.class_id = s2.class_id) as s) as items,
         avg(val),
         class_id
  from scores,
       students as s2
  where student_id = s2.id
group by class_id) r;
```

查询每个班级的课程平均分，考试人数， 以及所有考生的数据

```postgresql
select (
  select row_to_json(row)
  from (
    select count(row), jsonb_agg(row) as items
    from (
      select row_to_json(row)
      from (
        select * from students,
                      scores
        where students.class_id = cls.id
          and students.id = scores.student_id)
        as row)
      as row)
    as row)
  as data
from clazz as cls;
```

利用临时表查询

```postgresql

select * from (
  select (
    select row_to_json(r)
    from (
      with temp_data as (
        select count(rows),
               avg(rows.val),
               sum(rows.val),
               jsonb_agg(rows) as items from (
          select student_id, scores.id, val, student_id
          from students,
               scores 
          where students.class_id = clazz.id
            and students.id = scores.student_id) as rows)
      select *, clazz.id as class_id
      from temp_data) r ) 
    as row
  from clazz) r;
```
```postgresql

select (
  with recursive student_list as (
--   let student_list = [...]  // 因为需要用来count， 所以先用一个临时表存数据，防止多次查询导致缓慢
    select * from students where cls.id = students.class_id),
    response as (
      select (json_agg(student_list))            as items,
             (select count(*) from student_list) as cnt 
      from student_list)
--                 let response = {items: student_list, cnt: len(student_list)} // 组装成一个对象 返回
--                 export response
        select row_to_json(response)  -- 转json返回
        from response
       ) as items
from classes as cls; -- 返回客户端


```


使用多个临时变量，拆分 尽量减少嵌套
```postgresql

select (
   with recursive student_list as (select * from students where cls.id = students.class_id)
      -- Data => json
      -- count => len(json)
      -- return {items, count}
      , items as (select json_agg(student_list) as items from student_list)
      , response as (select (select items.items from items)              as items,
                            (select json_array_length(items) from items) as cnt
                     from items
        )
--                 let response = {items: student_list, cnt: len(student_list)} // 组装成一个对象 返回
--                 export response
   select row_to_json(s) -- 转json返回
   from response s
 ) as items
from classes as cls; -- 返回客户端

```


## 递归篇

```postgresql
CREATE TABLE employees
(
    employee_id serial PRIMARY KEY,
    full_name   VARCHAR NOT NULL,
    manager_id  INT
);
INSERT INTO employees (employee_id,
                       full_name,
                       manager_id)
VALUES (1, 'Michael North', NULL),
       (2, 'Megan Berry', 1),
       (3, 'Sarah Berry', 1),
       (4, 'Zoe Black', 1),
       (5, 'Tim James', 1),
       (6, 'Bella Tucker', 2),
       (7, 'Ryan Metcalfe', 2),
       (8, 'Max Mills', 2),
       (9, 'Benjamin Glover', 2),
       (10, 'Carolyn Henderson', 3),
       (11, 'Nicola Kelly', 3),
       (12, 'Alexandra Climo', 3),
       (13, 'Dominic King', 3),
       (14, 'Leonard Gray', 4),
       (15, 'Eric Rampling', 4),
       (16, 'Piers Paige', 7),
       (17, 'Ryan Henderson', 7),
       (18, 'Frank Tucker', 8),
       (19, 'Nathan Ferguson', 8),
       (20, 'Kevin Rampling', 8);


-- 递归查询1 查询返回平的数据， 需要自己组装
with recursive subordinates AS (
    SELECT employee_id,
           manager_id,
           full_name
    FROM employees e
    WHERE employee_id = 2
    UNION
    SELECT e.employee_id,
           e.manager_id,
           e.full_name
    FROM employees e
             INNER JOIN subordinates s ON s.employee_id = e.manager_id
)
SELECT *
FROM subordinates;

--  函数递归法 传入一个root_id 即可
create
    or replace function query(_id int) returns json as
$$
begin
  return (
    select jsonb_agg(r)
        from (
          SELECT *, query(e.employee_id) as children
          FROM employees e
          where e.manager_id = _id) as r);

end
$$
    language plpgsql;
select query(2);


```




