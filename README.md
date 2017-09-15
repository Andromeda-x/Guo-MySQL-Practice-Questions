# MySql常用命令总结
PS：感谢Dean Xu的第一个Start，值得纪念哈。
- 1、使用SHOW语句找出在服务器上当前存在什么数据库：<br>
mysql> SHOW DATABASES;
- 2、创建一个数据库MYSQLDATA<br>
mysql> CREATE DATABASE MYSQLDATA;
- 3、选择你所创建的数据库<br>
mysql> USE MYSQLDATA; (按回车键出现Database changed 时说明操作成功！)
- 4、查看现在的数据库中存在什么表<br>
mysql> SHOW TABLES;
- 5、创建一个数据库表<br>
mysql> CREATE TABLE MYTABLE (name VARCHAR(20), sex CHAR(1));
- 6、显示表的结构：<br>
mysql> DESCRIBE MYTABLE;
- 7、往表中加入记录<br>
mysql> insert into MYTABLE values (”hyq”,”M”);
- 8、用文本方式将数据装入数据库表中（例如D:/mysql.txt）<br>
mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt” INTO TABLE MYTABLE;
- 9、导入.sql文件命令（例如D:/mysql.sql）<br>
mysql>use database;
mysql>source d:/mysql.sql;
- 10、删除表<br>
mysql>drop TABLE MYTABLE;
- 11、清空表<br>
mysql>delete from MYTABLE;
- 12、更新表中数据<br>
mysql>update MYTABLE set sex=”f” where name=’hyq’;

匿名帐户删除、 root帐户设置密码:
```
use mysql;
delete from User where User=”";
update User set Password=PASSWORD(’newpassword’) where User=’root’;
```
GRANT的常用用法如下：
```
grant all on mydb.* to NewUserName@HostName identified by “password” ;
grant usage on *.* to NewUserName@HostName identified by “password”;
grant select,insert,update on mydb.* to NewUserName@HostName identified by “password”;
grant update,delete on mydb.TestTable to NewUserName@HostName identified by “password”;
```


#### 全局管理权限：
- FILE: 在MySQL服务器上读写文件。
- PROCESS: 显示或杀死属于其它用户的服务线程。
- RELOAD: 重载访问控制表，刷新日志等。
- SHUTDOWN: 关闭MySQL服务。

#### 数据库/数据表/数据列权限：
- ALTER: 修改已存在的数据表(例如增加/删除列)和索引。
- CREATE: 建立新的数据库或数据表。
- DELETE: 删除表的记录。
- DROP: 删除数据表或数据库。
- INDEX: 建立或删除索引。
- INSERT: 增加表的记录。
- SELECT: 显示/搜索表的记录。
- UPDATE: 修改表中已存在的记录。

#### 特别的权限：
- ALL: 允许做任何事(和root一样)。
- USAGE: 只允许登录–其它什么也不允许做。

# MySQL-Practice-Questions

## 1、取得每个部门最高薪水的人员名称
- 第一步：取得每个部门最高薪水『按照部门分组求最大值』<br>
``
mysql> select deptno,max(sal) as maxsal from emp group by deptno;
``

| deptno | maxsal  |
| :--------:|:--------:|
|     10 | 5000.00 |
|     20 | 3000.00 |
|     30 | 2850.00 |

- 第二步：将上面的查询结果当作临时表t，t表和emp e表进行连接<br>
条件：e.deptno=t.deptno and e.sal=t.sal

```
mysql> select
    ->   e.ename t.*
    -> from
    ->   emp e
    -> join
    ->  (select deptno,max(sal) as maxsal from emp group by deptno) t
    -> on
    ->  e.deptno=t.deptno and e.sal = t.maxsal;
```

| ename | deptno | maxsal  |
| :--------:|:--------:|:--------:|
| BLAKE |     30 | 2850.00 |
| SCOTT |     20 | 3000.00 |
| KING  |     10 | 5000.00 |
| FORD  |     20 | 3000.00 |

# 2、哪些人的薪水在部门的平均薪水之上
- 第一步：找出部门的平均薪水『按部门编号分组求平均薪水』<br>

``
select deptno,avg(sal) as avgsal from emp group by deptno;
``

| deptno | avgsal      |
| :--------:|:--------:|
|     10 | 2916.666667 |
|     20 | 2175.000000 |
|     30 | 1566.666667 |

- 第二步：将上面的查询结果当作临时表t，与emp e表进行连接<br>
条件：t.deptno=t.deptno and e.sal > t.avgsal

```
select
  e.ename,e.sal,t.*
from  
  emp e
join
  (select deptno,avg(sal) as avgsal from emp group by deptno) t
on
  e.deptno=t.deptno and e.sal > t.avgsal;
```

| ename | sal     | deptno | avgsal      |
| :--------:|:--------:| :--------:|:--------:|
| ALLEN | 1600.00 |     30 | 1566.666667 |
| JONES | 2975.00 |     20 | 2175.000000 |
| BLAKE | 2850.00 |     30 | 1566.666667 |
| SCOTT | 3000.00 |     20 | 2175.000000 |
| KING  | 5000.00 |     10 | 2916.666667 |
| FORD  | 3000.00 |     20 | 2175.000000 |

# 3、1取得部门中(所有人)平均薪水的等级
- 第一步：取得部门中的平均薪水<br>
``
select deptno,avg(sal) as avgsal from emp group by deptno;
``

| deptno | avgsal      |
| :--------:|:--------:|
|  10 | 2916.666667 |
|  20 | 2175.000000 |
|  30 | 1566.666667 |

- 第二部：将上面的查询结果当作临时表t，t表和salgrade s表进行关联
条件：e.sal between s.losal and s.hisal

```
select
  t.*,s.grade
from
  salgrade s
join
  (select deptno,avg(sal) as avgsal from emp group by deptno) t
on
  t.avgsal between s.losal and s.hisal;
```
| deptno | avgsal      | grade |
| :--------:|:--------:|:--------:|
|     10 | 2916.666667 |     4 |
|     20 | 2175.000000 |     4 |
|     30 | 1566.666667 |     3 |

# 3、2取得部门中(所有人)薪水的平均等级
- 第一步：每个员工的薪水等级(oder by 以部门编号排序，为了好理解)

```
select
  e.ename,e.sal,e.deptno,s.grade
from
  emp e
join
  salgrade s
on
  e.sal between s.losal and s.hisal ;
```
| ename  | sal  | deptno | grade |
| :--------:|:--------:|:--------:|:--------:|
| MILLER | 1300.00 |     10 |     2 |
| KING   | 5000.00 |     10 |     5 |
| CLARK  | 2450.00 |     10 |     4 |
| ADAMS  | 1100.00 |     20 |     1 |
| SCOTT  | 3000.00 |     20 |     4 |
| FORD   | 3000.00 |     20 |     4 |
| JONES  | 2975.00 |     20 |     4 |
| SMITH  |  800.00 |     20 |     1 |
| MARTIN | 1250.00 |     30 |     2 |
| ALLEN  | 1600.00 |     30 |     3 |
| JAMES  |  950.00 |     30 |     1 |
| BLAKE  | 2850.00 |     30 |     4 |
| WARD   | 1250.00 |     30 |     2 |
| TURNER | 1500.00 |     30 |     3 |

- 第二步：在以上基础上继续以部门编号分组，求平均薪水等级

```
select
  e.deptno,s.grade
from
  emp e
join
  salgrade s
on
  e.sal between s.losal and s.hisal
group by
  e.deptno;
```
| deptno | grade |
| :--------:|:--------:|
|     10 |     4 |
|     20 |     1 |
|     30 |     3 |

# 4、不用组函数(MAX),取得最高薪水(给出两种解决方案)
- 方案一：按照薪水降序排，取得第一个

``
mysql> select sal from emp order by sal desc limit 1;
``

- 方案二：自连接

``
mysql>mysql> select sal from emp where sal not in(select a.sal from emp a join emp b on a.sal < b.sal);
``

| sal     |
| :--------:|
| 5000.00 |

# 5、取得平均薪水最高的部门的编号(至少给出两种解决方案)

- 第一种方案：平均薪水降序排取第一个<br>
第一步：取得每个部门的平均薪水

``
mysql> select deptno,avg(sal) avgsal from emp group by deptno;
``

| deptno | avgsal      |
| :--------:|:--------:|
|     10 | 2916.666667 |
|     20 | 2175.000000 |
|     30 | 1566.666667 |

第二步：取得平均薪水的最大值

``
mysql> select avg(sal) avgsal from emp group by deptno order by avgsal desc limit 1;
``

| avgsal      |
| :--------:|
| 2916.666667 |

第三步：将第一步和第二步结合

```
select
  deptno,avg(sal) as avgsal
from
  emp
group by
    deptno
having
    avg(sal)=( select avg(sal) avgsal from emp group by deptno order by avgsal desc limit 1);
```

| deptno | avgsal      |
| :--------:|:--------:|
|     10 | 2916.666667 |

- 第二种方案：MAX函数<br>

```
select
  deptno,avg(sal) as avgsal
from
  emp
group by
    deptno
having
    avg(sal)=( select max(t.avgsal) from (select avg(sal) avgsal from emp group by deptno) t);
```

| deptno | avgsal      |
| :--------:|:--------:|
|     10 | 2916.666667 |

# 6、取得平均薪水最高的部门的部门名称

```
select
  d.dname,avg(e.sal) as avgsal
from
  emp e
join
  dept d
on e.deptno=d.deptno
group by
    d.dname
having
    avg(e.sal)=( select max(t.avgsal) from (select avg(sal) avgsal from emp group by deptno) t);
```

| dname      | avgsal      |
| :--------:|:--------:|
| ACCOUNTING | 2916.666667 |

# 7、求平均薪水的等级最高的部门的部门名称

第一步：求各个部门平均薪水的等级

```
select
  t.dname,t.avgsal,s.grade
from
  (select d.dname,avg(e.sal) as avgsal from emp e join dept d on e.deptno=d.deptno group by d.dname) t
join
  salgrade s
on
  t.avgsal between s.losal and s.hisal;
```

| dname      | avgsal      | grade |
| :--------:|:--------:|:--------:|
| ACCOUNTING | 2916.666667 |     4 |
| RESEARCH   | 2175.000000 |     4 |
| SALES      | 1566.666667 |     3 |

第二步：获得最高等级

```
select
  max(s.grade)
from
  (select avg(sal) as avgsal from emp  group by deptno) t
join
  salgrade s
on
  t.avgsal between s.losal and s.hisal;

```

第三步：将第一步和第二步联合

```
select
  t.dname,t.avgsal,s.grade
from
  (select d.dname,avg(e.sal) as avgsal from emp e join dept d on e.deptno=d.deptno group by d.dname) t
join
  salgrade s
on
  t.avgsal between s.losal and s.hisal
where
  s.grade=(select
            max(s.grade)
          from
            (select avg(sal) as avgsal from emp  group by deptno) t
          join
            salgrade s
          on
            t.avgsal between s.losal and s.hisal);
```

| dname      | avgsal      | grade |
| :--------:|:--------:|:--------:|
| ACCOUNTING | 2916.666667 |     4 |
| RESEARCH   | 2175.000000 |     4 |

## 8、取得比普通员工的最高薪水还要高的领导人姓名

第一步：取得普通员工<br>
``
select * from emp where empno not in (select distinct mgr from emp);
``

** 以上语句无法查村到结果，因为not in 不会自动忽略NULL，需要自己手动排除NULL。 in 自动忽略NULL**

``
select * from emp where empno not in (select distinct mgr from emp where mgr is not null);
``

| empno | ename  | job      | mgr  | hiredate   | sal     | comm    | deptno |
| :--------:|:--------:|:--------:| :--------:|:--------:|:--------:| :--------:|:--------:|
|  7369 | SMITH  | CLERK    | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7654 | MARTIN | SALESMAN | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7844 | TURNER | SALESMAN | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK    | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK    | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7934 | MILLER | CLERK    | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |

第二步：找出员工最高薪水的人

``
select max(sal) from emp where empno not in (select distinct mgr from emp where mgr is not null);
``

| max(sal) |
| :--------:|
|  1600.00 |

第三步：找出薪水大于1600即可

```
select ename,sal from emp where sal > (select max(sal) from emp where empno not in (select distinct mgr from emp where mgr is not null));
```

| ename | sal     |
|:--------:|:--------:|
| JONES | 2975.00 |
| BLAKE | 2850.00 |
| CLARK | 2450.00 |
| SCOTT | 3000.00 |
| KING  | 5000.00 |
| FORD  | 3000.00 |

### 补充：case ... when ... then ... when  ... then ... else ... end 类似于Java中的switch..case

```
select
  ename,sal,(case job when 'MANAGER' then sal*1.1 when 'CLERK' then sal*1.5 end) as newsal
from
  emp;
```

| ename  | sal     | newsal  |
|:--------:|:--------:|:--------:|
| SMITH  |  800.00 | 1200.00 |
| ALLEN  | 1600.00 |    NULL |
| WARD   | 1250.00 |    NULL |
| JONES  | 2975.00 | 3272.50 |
| MARTIN | 1250.00 |    NULL |
| BLAKE  | 2850.00 | 3135.00 |
| CLARK  | 2450.00 | 2695.00 |
| SCOTT  | 3000.00 |    NULL |
| KING   | 5000.00 |    NULL |
| TURNER | 1500.00 |    NULL |
| ADAMS  | 1100.00 | 1650.00 |
| JAMES  |  950.00 | 1425.00 |
| FORD   | 3000.00 |    NULL |
| MILLER | 1300.00 | 1950.00 |

```
select
  ename,sal,(case job when 'MANAGER' then sal*1.1 when 'CLERK' then sal*1.5 else sal end) as newsal
from
  emp;
```

| ename  | sal     | newsal  |
|:--------:|:--------:|:--------:|
| SMITH  |  800.00 | 1200.00 |
| ALLEN  | 1600.00 | 1600.00 |
| WARD   | 1250.00 | 1250.00 |
| JONES  | 2975.00 | 3272.50 |
| MARTIN | 1250.00 | 1250.00 |
| BLAKE  | 2850.00 | 3135.00 |
| CLARK  | 2450.00 | 2695.00 |
| SCOTT  | 3000.00 | 3000.00 |
| KING   | 5000.00 | 5000.00 |
| TURNER | 1500.00 | 1500.00 |
| ADAMS  | 1100.00 | 1650.00 |
| JAMES  |  950.00 | 1425.00 |
| FORD   | 3000.00 | 3000.00 |
| MILLER | 1300.00 | 1950.00 |

## 9、取得薪水最高的前五名员工

``
mysql> select ename,sal from emp order by sal desc limit 5;
``

| ename | sal     |
|:--------:|:--------:|
| KING  | 5000.00 |
| FORD  | 3000.00 |
| SCOTT | 3000.00 |
| JONES | 2975.00 |
| BLAKE | 2850.00 |

## 10、取得薪水最高的第六名到第十名。

``
mysql> select ename,sal from emp order by sal desc limit 5,5;
``

| ename  | sal     |
|:--------:|:--------:|
| CLARK  | 2450.00 |
| ALLEN  | 1600.00 |
| TURNER | 1500.00 |
| MILLER | 1300.00 |
| WARD   | 1250.00 |

## 11、取得最后入职的五名员工

``
mysql> select ename,hiredate from emp order by hiredate desc limit 5;
``

| ename  | hiredate   |
|:--------:|:--------:|
| ADAMS  | 1987-05-23 |
| SCOTT  | 1987-04-19 |
| MILLER | 1982-01-23 |
| JAMES  | 1981-12-03 |
| FORD   | 1981-12-03 |

## 12、取得每个薪水等级有多少员工
第一步：找出每个员工的薪水的等级

``
mysql> select e.ename,e.sal,s.grade from emp e join salgrade s on e.sal between s.losal and s.hisal;
``

| ename  | sal     | grade |
|:--------:|:--------:|:--------:|
| SMITH  |  800.00 |     1 |
| ALLEN  | 1600.00 |     3 |
| WARD   | 1250.00 |     2 |
| JONES  | 2975.00 |     4 |
| MARTIN | 1250.00 |     2 |
| BLAKE  | 2850.00 |     4 |
| CLARK  | 2450.00 |     4 |
| SCOTT  | 3000.00 |     4 |
| KING   | 5000.00 |     5 |
| TURNER | 1500.00 |     3 |
| ADAMS  | 1100.00 |     1 |
| JAMES  |  950.00 |     1 |
| FORD   | 3000.00 |     4 |
| MILLER | 1300.00 |     2 |

第二步：在以上结果的基础上，按照grade进行分组，count计数
```
select
  s.grade,count(*)
from
  emp e
join
  salgrade s
on
  e.sal between s.losal and s.hisal
group by
  s.grade;
```

| grade | count(*) |
|:--------:|:--------:|
|     1 |        3 |
|     2 |        3 |
|     3 |        2 |
|     4 |        5 |
|     5 |        1 |

## 13、面试题(建议自己动手设计下)
![](https://i.imgur.com/3vHlqxK.png)

S 学生表

| sno(pk) | sname |
|:--------:|:--------:|
|     1 |        张三 |
|     2 |        李四 |
|     3 |        王五 |
|     4 |        赵六 |


C 课程表

| cno(pk) | cname | cteacher |
|:------:|:-------:|:------:|
|     1 |  linux  | 张老师 |
|     2 |  MySQL  | 李老师 |
|     3 |  Git    | 王老师 |
|     4 |  Java   | 赵老师 |
|     5 |  Redis   | 黎明 |

SC 学生选课表 <br>
【sno+cno是复合主键，主键只有一个，同时sno、cno又是外键。外键可以有两个】

| sno | cno | scgrede |
|:------:|:------:|:-------:|
|   1   |     1   |   50  |
|   1   |     2   |   50   |
|   1   |     3   |   50   |
|   2   |     2   |   80  |
|   2   |     3   |   70  |
|   2   |     4   |    59  |
|   3   |      1  |    60  |
|   3   |      2  |    61  |
|   3   |      3  |    99  |
|   3   |      4  |   100   |
|   3   |      5  |   52   |
|   4   |      3  |   82   |
|   4   |      4  |   99   |
|   4   |      5  |   46   |


## 1、找出没选过“黎明”老师的所有学生姓名
第一步：找出黎明老师所授课的编号

``
select cno from  C where cteacher = '黎明';
``

| cno |
|:------:|
| 5   |

第二步：通过学生选课表查询cno=上面结果的sno，这些sno是选黎明老师课程的学号

``
select sno from SC where cno = (select cno from  C where cteacher = '黎明');
``

| sno |
|:------:|
| 3   |
| 4   |
第三步：在学生表中查询sno not in 上面结果的数据

`select
  sname
from
  S  
where
    sno not in (select sno from SC where cno = (select cno from  C where cteacher = '黎明'));
`

| sname  |
|:------:|
| 张三   |
| 李四   |

## 2、列出2门以上(含2门)不及格学生姓名及平均成绩
第一步：找出分数小于60并且按sno分组，计数大于2的

```
select
  sc.sno
from
  SC sc
where
  sc.scgrade < 60
group by
  sc.sno
having
  count(*) >=2;
```

| sno |
|:------:|
| 1   |

第二步：与学生表S进行连接

```
select
  sc.sno,s.sname
from
  SC sc
join
  S s
on
  sc.sno=s.sno
where
  sc.scgrade < 60
group by
    sc.sno,s.sname
having
  count(*) >=2;
```

| sno | sname  |
|:------:|:----:|
| 1   | 张三   |

第三步：找出每个学生的平均成绩

`
select sno,avg(scgrade) as avggrade from SC group by sno;
`

| sno | avggrade      |
|:------:|:--------------:|
| 1   |                50 |
| 2   | 69.66666666666667 |
| 3   |              74.4 |
| 4   | 75.66666666666667 |

第四步：第二步当作临时表t1和第三步当作临时表t2进行联合

`
select t1.sname,t2.avggrade from t1 join t2 on t1.sno=t2.sno;
`

```
select
  t1.sno,t1.sname,t2.avggrade
from
  (select
    sc.sno,s.sname
  from
    SC sc
  join
    S s
  on
    sc.sno=s.sno
  where
    sc.scgrade < 60
  group by
      sc.sno,s.sname
  having
    count(*) >=2) t1
join
  (select sno,avg(scgrade) as avggrade from SC group by sno) t2
on
  t1.sno=t2.sno;
```

| sno | sname  | avggrade |
|:------:|:------:|:------:|
| 1   | 张三   |       50 |


## 3、即学过1号课又学过2号课所有学生的姓名

第一步：找出学过1号课程的学生

`select sno from SC where cno=1;
`

| sno |
|:------:|
| 1   |
| 3   |

第二步：找出学过2号课程的学生

`select sno from SC where cno=2;
`

| sno |
|:------:|
| 1   |
| 2   |
| 3   |

第三步：将第一步和第二部进行联合

``
select sno from SC where cno=1 and sno in(select sno from SC where cno=2);
``

| sno |
|:------:|
| 1   |
| 3   |


第四步：将上面结果和S表进行联合

```
select
  sc.sno,s.sname
from
  SC sc
join
  S s
on
  sc.sno=s.sno
where
  sc.cno=1 and sc.sno in(select sno from SC where cno=2);
```

| sno | sname  |
|:------:|:------:|
| 1   | 张三   |
| 3   | 王五   |

## 14、列出所有员工及领导名字
表的自关联emp a<员工表> emp b <领导表>

```
select
  a.ename empname,b.ename leardername
from
  emp a
left join
  emp b
on
  a.mgr=b.empno;

  +---------+-------------+
| empname | leardername |
+---------+-------------+
| SMITH   | FORD        |
| ALLEN   | BLAKE       |
| WARD    | BLAKE       |
| JONES   | KING        |
| MARTIN  | BLAKE       |
| BLAKE   | KING        |
| CLARK   | KING        |
| SCOTT   | JONES       |
| TURNER  | BLAKE       |
| ADAMS   | SCOTT       |
| JAMES   | BLAKE       |
| FORD    | JONES       |
| MILLER  | CLARK       |
+---------+-------------+
13 rows in set (0.06 sec)
```

## 15、列出受雇日期早于其直接上级领导的所有员工编号，姓名、部门名称
第一步：表的自关联emp a<员工表> emp b <领导表>找出所有员工

```
select
  a.empno '员工编号', a.ename '员工姓名',a.hiredate '员工入职日期',
  b.empno '领导编号',b.ename '领导姓名',b.hiredate '领导入职日期'
from
  emp a
join
  emp b
on
  a.mgr=b.empno
where
  a.hiredate<b.hiredate;
```

| 员工编号     | 员工姓名     | 员工入职日期       | 领导编号     | 领导姓名     | 领导入职日期       |
|:------:|:------:|:------:|:------:|:------:|:------:|
|         7369 | SMITH        | 1980-12-17         |         7902 | FORD         | 1981-12-03         |
|         7499 | ALLEN        | 1981-02-20         |         7698 | BLAKE        | 1981-05-01         |
|         7521 | WARD         | 1981-02-22         |         7698 | BLAKE        | 1981-05-01         |
|         7566 | JONES        | 1981-04-02         |         7839 | KING         | 1981-11-17         |
|         7698 | BLAKE        | 1981-05-01         |         7839 | KING         | 1981-11-17         |
|         7782 | CLARK        | 1981-06-09         |         7839 | KING         | 1981-11-17         |

第二步：与dept表进行关联

```
select
  a.empno '员工编号', a.ename '员工姓名',a.hiredate '员工入职日期',
  b.empno '领导编号',b.ename '领导姓名',b.hiredate '领导入职日期',
  d.dname '部门名称'
from
  emp a
join
  emp b
on
  a.mgr=b.empno
join
  dept d
on
  a.deptno=d.deptno
where
  a.hiredate<b.hiredate;
```

| 员工编号     | 员工姓名     | 员工入职日期       | 领导编号     | 领导姓名     | 领导入职日期       | 部门名称     |
|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
|         7369 | SMITH        | 1980-12-17         |         7902 | FORD         | 1981-12-03         | RESEARCH     |
|         7499 | ALLEN        | 1981-02-20         |         7698 | BLAKE        | 1981-05-01         | SALES        |
|         7521 | WARD         | 1981-02-22         |         7698 | BLAKE        | 1981-05-01         | SALES        |
|         7566 | JONES        | 1981-04-02         |         7839 | KING         | 1981-11-17         | RESEARCH     |
|         7698 | BLAKE        | 1981-05-01         |         7839 | KING         | 1981-11-17         | SALES        |
|         7782 | CLARK        | 1981-06-09         |         7839 | KING         | 1981-11-17         | ACCOUNTING   |

## 16、列出部门名称和这些员工信息同时列出那些没有员工的部门
PS；使用表关联和右外连接emp e <员工表> dept d <部门表>

```
select
  e.*,d.dname
from
  emp e
right join
  dept d
on
  e.deptno=d.deptno;
```
空间利用，不展示了。

## 17、列出至少有五个员工的部门详细信息
PS:分组可以使用多个字段联合起来。

```
select
  d.deptno,d.dname,d.loc,count(e.ename)
from
  emp e
join
  dept d
on
  e.deptno=d.deptno
group by
  d.deptno,d.dname,d.loc
having
  count(e.ename)>=5;

  +--------+----------+---------+----------------+
| deptno | dname    | loc     | count(e.ename) |
+--------+----------+---------+----------------+
|     20 | RESEARCH | DALLAS  |              5 |
|     30 | SALES    | CHICAGO |              6 |
+--------+----------+---------+----------------+
2 rows in set (0.07 sec)
```

## 18、列出薪金比“SMITH”多的所有员工
`select * from emp where sal > (select sal from emp where ename='SMITH');
` <br>
空间利用，不展示了。

## 19、列出所有“CLERK”(办事员)的姓名及其部门名称，部门人数
第一步：找出工作是“CLERK”所有员工
```
select ename from emp where job='CLERK';

+--------+
| ename  |
+--------+
| SMITH  |
| ADAMS  |
| JAMES  |
| MILLER |
+--------+
4 rows in set (0.00 sec)
```

第二步：进行表关联，得出部门名称

```
select
  e.ename,d.dname
from
  dept d
join
  emp e
on
  e.deptno=d.deptno
where
  e.job='CLERK';

+--------+------------+
| ename  | dname      |
+--------+------------+
| SMITH  | RESEARCH   |
| ADAMS  | RESEARCH   |
| JAMES  | SALES      |
| MILLER | ACCOUNTING |
+--------+------------+
4 rows in set (0.00 sec)

```

第三步：按部门编号分组，求每个部门人数

```
select deptno,count(*) as totalEmp from emp group by deptno;

+--------+----------+
| deptno | totalEmp |
+--------+----------+
|     10 |        3 |
|     20 |        5 |
|     30 |        6 |
+--------+----------+
3 rows in set (0.00 sec)
```
第四步： 将第二步和第三步(当作临时表t)进行关联

```
select
  e.ename,d.dname,t.totalEmp
from
  dept d
join
  emp e
on
  e.deptno=d.deptno
join
  (select deptno,count(*) as totalEmp from emp group by deptno)  t
on
  t.deptno=d.deptno
where
  e.job='CLERK';

  +--------+------------+----------+
| ename  | dname      | totalEmp |
+--------+------------+----------+
| SMITH  | RESEARCH   |        5 |
| ADAMS  | RESEARCH   |        5 |
| JAMES  | SALES      |        6 |
| MILLER | ACCOUNTING |        3 |
+--------+------------+----------+
4 rows in set (0.00 sec)
```

## 20、列出 最低薪金大于1500的各种工作及从事此工作的全部雇员人数

```
mysql> select job,min(sal),count(*) as totalEmp from emp group by job having min(sal)>1500;

+-----------+----------+----------+
| job       | min(sal) | totalEmp |
+-----------+----------+----------+
| ANALYST   |  3000.00 |        2 |
| MANAGER   |  2450.00 |        3 |
| PRESIDENT |  5000.00 |        1 |
+-----------+----------+----------+
3 rows in set (0.00 sec)
```

## 21、列出部门在“SALES”<销售部>工作的姓名，假定不知道销售部的部门的部门编号
第一步：查处部门编号(30)

`select deptno from dept where dname='SALES';
 `

第二步：表关联

```
select ename from emp where deptno=(select deptno from dept where dname='SALES');

+--------+
| ename  |
+--------+
| ALLEN  |
| WARD   |
| MARTIN |
| BLAKE  |
| TURNER |
| JAMES  |
+--------+
6 rows in set (0.00 sec)
```
