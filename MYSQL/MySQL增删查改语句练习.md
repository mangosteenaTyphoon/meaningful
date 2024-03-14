#### 1.if null 函数

ifnull(a,b)函数解释：如果value1不是空，结果返回a   如果value1是空，结果返回b；



#### 2. dense_rank() over

作用：查出指定条件后的进行排名，条件相同排名相同，排名间断不连续。
说明：和rank() over 的作用相同，区别在于dense_rank() over 排名是密集连续的。例如学生排名，使用这个函数，成绩相同的两名是并列，下一位同学接着下一个名次。即：1 1 2 3 4 5 5 6



#### 3.rank() over 

作用：查出指定条件后的进行排名，条件相同排名相同，排名间断不连续。
说明：例如学生排名，使用这个函数，成绩相同的两名是并列，下一位同学空出所占的名次。即：1 1 3 4 5 5 7



#### 4.row_number() over

作用：查出指定条件后的进行排名，条件相同排名也不相同，排名间断不连续。
说明：这个函数不需要考虑是否并列，即使根据条件查询出来的数值相同也会进行连续排序。即：1 2 3 4 5 6



#### 5. 使用 NOW() 、 CURDATE()、CURTIME() 获取当前时间

在 SQL 中，我们可以通过使用 **NOW()**、**CURDATE()**、**CURTIME()** 来获取当前的时间

- `NOW()` 可以用来返回当前日期和时间 格式：YYYY-MM-DD hh:mm:ss
- `CURDATE()` 可以用来返回当前日期 格式：YYYY-MM-DD
- `CURTIME()` 可以用来返回当前时间 格式：hh:mm:ss  

#### 6. 将一张表的查询结果插入到另一张表中

要根据 WHERE 条件将一张表的值插入到另一张表中，你可以使用 INSERT INTO SELECT 结合子句。以下是一个示例查询:

``` mysql
INSERT INTO table2 (column1, column2, ...)
SELECT column1, column2, ...
FROM table1
WHERE your_condition_here;
```

在这个示例中:

- `table1` 是源表的名称，`table2` 是目标表的名称；
- `column1`, `column2`, ... 是要插入的列；
- `your_condition_here` 是根据哪些条件筛选记录。 举个例子，假设你有两张表 `employees` 和 `new_employees`，你想根据员工部门将 `employees` 表中指定部门的员工插入到 `new_employees` 表中，可以这样做：

``` mysql
INSERT INTO new_employees (employee_id, employee_name, department)
SELECT employee_id, employee_name, department
FROM employees
WHERE department = 'Sales';
```



## 分组 子查询联系

### 创建数据库表

```mysql
DROP DATABASE IF EXISTS judge;
CREATE DATABASE judge;
use judge;

DROP TABLE IF EXISTS `courses`;
CREATE TABLE `courses`
(
    `id`            INT UNSIGNED NOT NULL AUTO_INCREMENT,
    `name`          VARCHAR(64),
    `student_count` INT,
    `created_at`    DATETIME,
    `teacher_id`    INT UNSIGNED,
    PRIMARY KEY (`id`)
);

DROP TABLE IF EXISTS `teachers`;
CREATE TABLE `teachers`
(
    `id`      INT UNSIGNED NOT NULL AUTO_INCREMENT,
    `name`    VARCHAR(64)  NOT NULL,
    `email`   VARCHAR(64),
    `age`     INT UNSIGNED NOT NULL,
    `country` VARCHAR(32)  NOT NULL,
    PRIMARY KEY (`id`)
);


```

#### 表中数据

```mysql
teachers 表的数据
INSERT INTO judge.teachers (id, name, email, age, country) VALUES (1, 'Eastern Heretic', 'eastern.heretic@gmail.com', 20, 'UK');
INSERT INTO judge.teachers (id, name, email, age, country) VALUES (2, 'Northern Beggar', 'northern.beggar@qq.com', 21, 'CN');
INSERT INTO judge.teachers (id, name, email, age, country) VALUES (3, 'Western Venom', 'western.venom@163.com', 28, 'USA');
INSERT INTO judge.teachers (id, name, email, age, country) VALUES (4, 'Southern Emperor', 'southern.emperor@qq.com', 21, 'JP');
INSERT INTO judge.teachers (id, name, email, age, country) VALUES (5, 'Linghu Chong', 'NULL', 18, 'CN');


courses表的数据

INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (1, 'Senior Algorithm', 880, '2020-06-01 00:00:00', 4);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (2, 'System Design', 1350, '2020-07-18 00:00:00', 3);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (3, 'Django', 780, '2020-02-29 00:00:00', 3);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (4, 'Web', 340, '2020-04-22 00:00:00', 4);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (5, 'Big Data', 700, '2020-09-11 00:00:00', 1);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (6, 'Artificial Intelligence', 1660, '2018-05-13 00:00:00', 3);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (7, 'Java P6+', 780, '2019-01-19 00:00:00', 3);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (8, 'Data Analysis', 500, '2019-07-12 00:00:00', 1);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (10, 'Object Oriented Design', 300, '2020-08-08 00:00:00', 4);
INSERT INTO judge.courses (id, name, student_count, created_at, teacher_id) VALUES (12, 'Dynamic Programming', 2000, '2018-08-18 00:00:00', 1);

```

#### 问题1:    请编写 SQL 语句，查询教师表 teachers 和课程表 courses，查询最年长的老师所开课程的学生数，最后返回学生数均超过该教师的课程的课程信息。

- 先获取最年长的教师年龄

```mysql
select max(age)
From teachers;
```

- 再查询最年长老师所开课程的学生数

```mysql
SELECT max(c.student_count)
FROM courses c
         JOIN teachers t ON t.id = c.teacher_id
where t.age = (select max(age) From teachers);
```

- 然后使用课程表与他比较 大于的放出来

```mysql
select *
FROM courses c
WHERE student_count
          > (SELECT max(c.student_count)
             FROM courses c
                      JOIN teachers t ON t.id = c.teacher_id
             where t.age = (select max(age) From teachers));
```

#### 问题2:  找出每个国家的老师数量，并按照老师数量降序排列。

1.方法一（嵌套查询)

```mysql
SELECt t.name, c.teacher_id, AVG(c.student_count)
FROM courses c
         JOIN teachers t on c.teacher_id = t.id
GROUP BY c.teacher_id;
```

2.方法二 (使用Having 对 分组后的条件进行查询 )

```mysql
SELECT t.*,count(1)
FROM teachers t
JOIN courses c ON t.id = c.teacher_id
GROUP BY t.id
HAVING COUNT(c.id) >= 2;
```

#### 问题3   请编写 SQL 语句，对于 Southern Emperor 教师的 所有 课程 temp_courses，从 courses 表和 teachers 表中查询：
除了 temp_courses 外的其他课程

- 并且课程的创建时间晚于 temp_courses 中的 任意一门 课程
- 返回满足以上要求的课程名称，查询结果的列名为 name。

使用**any** 来子查询

```mysql
select c.name from courses c
where created_at >
    any(select created_at from courses c
        inner join teachers t on c.teacher_id = t.id
        where t.name = 'Southern Emperor')
and teacher_id!=(select id from teachers t where t.name = 'Southern Emperor')
```

#### 问题4 请编写 SQL 语句，从 teachers 表中，筛选出同一国家的教师平均年龄大于所有教师平均年龄的 国家，并获取这些国家的所有教师信息。

第一步、先获取国家的平均教师年龄大于所有教师平均年龄的国家

```mysql
SELECT country
FROM teachers
GROUP BY country
HAVING AVG(age) > (SELECT AVG(age) FROM teachers);
```

第二步 通过子查询来比较

```mysql
SELECT *
FROM teachers
WHERE country IN
      (SELECT country
       FROM teachers
       GROUP BY country
       HAVING AVG(age) > (SELECT AVG(age) FROM teachers));
```

