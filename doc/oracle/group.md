## 聚合(Aggregate)函数

min,max,sum,count,avg,stddev(标准差),variance(方差)

注意表中无数据count返回0，而其他统计函数返回NULL

## 分组统计(GROUP BY)

分组条件：例如按照部门分组

不成文的规定：当数据重复(比如好多个人都是市场部)的时候分组才有意义

```sql
SELECT e2.deptno, 
       --e2.dname, 只能select分组字段
       Count(e2.sal), 
       Avg(e2.sal) 
FROM   (SELECT e.deptno, 
               d.dname, 
               e.sal 
        FROM   emp e,
               dept d 
        WHERE  e.deptno = d.deptno) e2 -- 子查询内不得有分号 
GROUP  BY e2.deptno; 
```

- 分组函数可以在不分组下使用，但此时select语句不能查询字段

- 进行分组时只能出现分组的字段和统计函数，其它字段不得出现

- 分组函数嵌套时如MAX(AVG(sal))不能select其它字段

## 部门名，人数，平均工资

```sql
SELECT e.deptno, 
       e.dname, 
       Count(e.empno), 
       Avg(e.sal) 
FROM   (SELECT e.deptno, 
               d.dname, 
               e.sal 
        FROM   emp e, 
               dept d 
        WHERE  e.deptno = d.deptno) e 
GROUP  BY e.deptno; 
```

## 【重要】分组后再分组

```sql
-- 各部门的各职业各有几人？
SELECT deptno,
       job,
       count(*)
FROM emp
GROUP BY deptno,
         job
ORDER BY deptno;
```

## HAVING子句

where子句中不得出现组函数

如果要对分组后的数据再次进行过滤，则用HAVING子句

业务逻辑：征兵

- 先用WHERE筛选出符合条件的
- GROUPBY为海军、陆军、空军
- 要想选中空军不对，就不能再用where了，where必须在group by前
- HAVING子句过滤出空军(HAVING最大优势可用分组函数)

## HAVING练习1

显示非销售人员职位名称及同职位工资总和(满足大于5000)

```sql
SELECT job, 
       SUM(sal) 
FROM   emp 
WHERE  job != 'SALESMAN' 
GROUP  BY job 
HAVING SUM(sal) > 5000 
ORDER  BY SUM(sal); 
```

## 工资数值有几种

```sql
SELECT COUNT(empno),
       COUNT(DISTINCT sal)
FROM emp;
```

## 练习题1

找出工资比SMITH或MARTIN多的员工 的 编号，姓名，工资，领导名，领导工资

```sql
SELECT emp.empno,
       emp.ename,
       emp.sal,
       boss.bossname,
       boss.sal
FROM emp,
  (SELECT empno,
          ename AS bossname,
          sal
   FROM emp) boss
WHERE emp.mgr = boss.empno(+)
  AND emp.sal >
    ( SELECT MIN(sal)
     FROM emp
     WHERE ename IN ('SMITH',
                     'MARTIN') )
ORDER BY emp.empno;

/*
EMPNO ENAME SAL  BOSSNAME SAL
----- ----- ---- -------- ----
 7499 ALLEN 1600 BLAKE    2850
 7521 WARD  1250 BLAKE    2850
 7566 JONES 2975 KING     5000
...
*/
```

## 练习题2

列出各个职位最低工资雇员姓名，工资

```sql
SELECT e.job,
       e.sal,
       e.ename
FROM emp e,

  (SELECT job,
          min(sal) AS MIN
   FROM emp
   GROUP BY job) job
WHERE e.sal = job.min
ORDER BY e.sal;
```