# LeetCode免费SQL题秋招复习笔记

## 1. 上升的温度

考察点：`自联查询`、`时间函数`

给定一个 Weather 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。

| Id(INT) | RecordDate(DATE) | Temperature(INT) |
| :-----: | :--------------: | :--------------: |
|    1    |    2015-01-01    |        10        |
|    2    |    2015-01-02    |        25        |
|    3    |    2015-01-03    |        20        |
|    4    |    2015-01-04    |        30        |

例如，根据上述给定的 Weather 表格，返回如下 Id:

|  Id  |
| :--: |
|  2   |
|  4   |

```mysql
SELECT w1.Id
FROM Weather as w1,Weather as w2
WHERE w1.RecordDate = DATE_ADD(w2.RecordDate,INTERVAL 1 DAY) #这一天和前一天的联表
and w1.Temperature > w2.Temperature#比前一天温度高的
```



## 2. 交换工资

考察点：`case语句`

给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。

注意，您必只能写一个 Update 语句，请不要编写任何 Select 语句。

例如：

|  id  | name | sex  | salary |
| :--: | :--: | :--: | :----: |
|  1   |  A   |  m   |  2500  |
|  2   |  B   |  f   |  1500  |
|  3   |  C   |  m   |  5500  |
|  4   |  D   |  f   |  500   |

运行你所编写的更新语句之后，将会得到以下表:

|  id  | name | sex  | salary |
| :--: | :--: | :--: | :----: |
|  1   |  A   |  f   |  2500  |
|  2   |  B   |  m   |  1500  |
|  3   |  C   |  f   |  5500  |
|  4   |  D   |  m   |  500   |

```mysql
UPDATE salary
SET sex = 
CASE sex
WHEN 'f'
THEN 'm'
ELSE 'f'
end;
```



## 3. 组合两个表

考察点：`外联查询`

表1: Person

|   列名    |  类型   |
| :-------: | :-----: |
| PersonId  |   int   |
| FirstName | varchar |
| LastName  | varchar |

PersonId 是上表主键
表2: Address

|   列名    |  类型   |
| :-------: | :-----: |
| AddressId |   int   |
| PersonId  |   int   |
|   City    | varchar |
|   State   | varchar |

AddressId 是上表主键


编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

FirstName, LastName, City, State

```mysql
SELECT FirstName,LastName,City,State
FROM Person LEFT OUTER JOIN Address
ON Person.PersonId = Address.PersonId
```



## 4. 超过5名学生的课

考察点：`GROUPBY`、`HAVING`

有一个courses 表 ，有: student (学生) 和 class (课程)。

请列出所有超过或等于5名学生的课。

例如,表:

| student |  class   |
| :-----: | :------: |
|    A    |   Math   |
|    B    | English  |
|    C    |   Math   |
|    D    | Biology  |
|    E    |   Math   |
|    F    | Computer |
|    G    |   Math   |
|    H    |   Math   |
|    I    |   Math   |

应该输出:

| class |
| :---: |
| Math  |

Note:
学生在每个课中不应被重复计算。

```mysql
SELECT
    class
FROM
    courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5
```



## 5. 连续出现的数字

考察点：`自定义变量`

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

|  Id  | Num  |
| :--: | :--: |
|  1   |  1   |
|  2   |  1   |
|  3   |  1   |
|  4   |  2   |
|  5   |  1   |
|  6   |  2   |
|  7   |  2   |

例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。

| ConsecutiveNums |
| :-------------: |
|        1        |

```mysql
SELECT DISTINCT a.Num ConsecutiveNums
FROM (
SELECT t.Num ,
       # 数据数据相等则cnt+1
       @cnt:=IF(@pre=t.Num, @cnt+1, 1) cnt,# 三元表达式，相当于java中的x?y:z
       @pre:=t.Num pre
  FROM Logs t, (SELECT @pre:=null, @cnt:=0) b) a#把数据结果，即Num，cnt，pre作为一个新表供外部select处理
WHERE a.cnt >= 3
```



## 6. 交换位置

考察点：`IF`

小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。

其中纵列的 id 是连续递增的

小美想改变相邻俩学生的座位。

你能不能帮她写一个 SQL query 来输出小美想要的结果呢？

示例：

|  id  | student |
| :--: | :-----: |
|  1   |  Abbot  |
|  2   |  Doris  |
|  3   | Emerson |
|  4   |  Green  |
|  5   | Jeames  |

假如数据输入的是上表，则输出结果如下：

|  id  | student |
| :--: | :-----: |
|  1   |  Doris  |
|  2   |  Abbot  |
|  3   |  Green  |
|  4   | Emerson |
|  5   | Jeames  |

注意：

如果学生人数是奇数，则不需要改变最后一个同学的座位。

```mysql
select 
    if(id%2=0,
        id-1,#如果是偶数，id-1
        if(id=(select count(distinct id) from seat),#如果是奇数，判断是否最后一个
            id,#是最后一个，不变
            id+1)) #不是最后一个，id+1
    as id,student 
from seat 
order by id;
```



## 7. 体育馆的人流量

X 市建了一个新的体育馆，每日人流量信息被记录在这三列信息中：序号 (id)、日期 (visit_date)、 人流量 (people)。

请编写一个查询语句，找出人流量的高峰期。高峰期时，至少连续三行记录中的人流量不少于100。

例如，表 stadium：

|  id  | visit_date | people |
| :--: | :--------: | :----: |
|  1   | 2017-01-01 |   10   |
|  2   | 2017-01-02 |  109   |
|  3   | 2017-01-03 |  150   |
|  4   | 2017-01-04 |   99   |
|  5   | 2017-01-05 |  145   |
|  6   | 2017-01-06 |  1455  |
|  7   | 2017-01-07 |  199   |
|  8   | 2017-01-08 |  188   |

对于上面的示例数据，输出为：

|  id  | visit_date | people |
| :--: | :--------: | :----: |
|  5   | 2017-01-05 |  145   |
|  6   | 2017-01-06 |  1455  |
|  7   | 2017-01-07 |  199   |
|  8   | 2017-01-08 |  188   |

提示：
每天只有一行记录，日期随着 id 的增加而增加。

```mysql
select distinct s.* 
from stadium s,
(select id,visit_date,people,(
    @cnt:=IF(people>99,@cnt+1,0)
) as cnt from stadium,(select @cnt:=0) b) c 
where c.cnt>2 and s.id between c.id-c.cnt+1 and c.id
```

