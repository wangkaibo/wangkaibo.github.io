---
title: "【LeetCode】SQL 统计部门前三名工资185. Department Top Three Salaries"
date: 2015-12-12T00:29:22+08:00
tags: ["LeetCode"]
draft: false
url: '/leetcode-sql-tong-ji-bu-men-qian-san-ming-gong-zi-185-department-top-three-salaries'
---

### 问题描述
员工表包含所有员工。每个员工都有一个ID，并且还有一列表示部门ID。

部门表包括公司所有的部门。

写一条 SQL 查询出每个部门薪酬排名前三的的员工。对于上述表，你的 SQL 应该返回下面几列。

LeetCode链接：[Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/)

<!--more-->

#### 员工表

```
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
+----+-------+--------+--------------+
```
#### 部门表

```
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```
#### 关系表

```
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
```

### 解决方案

```SQL
select 
d.Name Department, e.Name Employee, e.Salary 
from Employee e 
join Department d on d.Id=e.DepartmentId 
where (
    select count(distinct Salary) from Employee where Salary > e.Salary and DepartmentId=d.id 
) < 3 order by d.Name, e.Salary desc;
```