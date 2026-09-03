# SQL 七天进阶学习指南

> 适合已了解基本查询语法、希望系统性进阶的学习者。每天包含：核心知识点、示例讲解、练习题、当日小结。建议每天配合真实数据库（MySQL / PostgreSQL / SQLite 均可）动手实践。

---

## 准备工作：搭建练习环境

建议使用以下两张示例表贯穿全部案例（可在任意数据库中建表并插入数据）：

```sql
-- 员工表
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    manager_id INT,
    hire_date DATE
);

-- 订单表
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    emp_id INT,
    customer VARCHAR(50),
    amount DECIMAL(10,2),
    order_date DATE
);
```

---

## Day 1：查询基础巩固与筛选进阶

**目标**：不满足于 `SELECT * FROM table`，掌握精确筛选与排序的组合技巧。

### 知识点
- `SELECT` 列裁剪、`DISTINCT` 去重
- `WHERE` 中的 `BETWEEN`、`IN`、`LIKE`、`IS NULL`
- 多条件排序 `ORDER BY`（含 `NULLS FIRST/LAST`）
- `LIMIT` / `OFFSET` 分页

### 示例
查询薪资在 8000~15000 之间、部门为技术部或市场部、按薪资降序排列的员工：

```sql
SELECT emp_name, department, salary
FROM employees
WHERE salary BETWEEN 8000 AND 15000
  AND department IN ('技术部', '市场部')
ORDER BY salary DESC;
```

模糊匹配姓名中含"王"的员工，并分页显示第 2 页（每页 5 条）：

```sql
SELECT emp_name FROM employees
WHERE emp_name LIKE '%王%'
LIMIT 5 OFFSET 5;
```

### 练习
1. 查询入职日期在 2023 年之后、薪资高于全公司平均薪资的员工（先不用子查询，用固定数值代替均值练手）。
2. 找出 `manager_id` 为空的员工（说明其为最高管理者）。

---

## Day 2：聚合函数与分组统计

**目标**：从"看明细"过渡到"看统计结果"，这是数据分析类需求的基础。

### 知识点
- 聚合函数：`COUNT`、`SUM`、`AVG`、`MAX`、`MIN`
- `GROUP BY` 分组统计
- `HAVING` 对分组结果二次筛选（区别于 `WHERE`）

### 示例
统计每个部门的人数与平均薪资：

```sql
SELECT department,
       COUNT(*) AS headcount,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

找出平均薪资超过 12000 的部门（注意 `HAVING` 而非 `WHERE`，因为条件基于聚合结果）：

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 12000;
```

### 易错点
`WHERE` 在分组前过滤原始行，`HAVING` 在分组后过滤聚合结果——两者不可互换。

### 练习
1. 统计每位员工的订单总金额和订单数量（需要连接 `orders` 表，可先尝试用子查询实现）。
2. 找出订单数超过 3 笔的客户。

---

## Day 3：多表连接（JOIN）

**目标**：这是 SQL 进阶的分水岭，真实业务数据几乎都涉及多表关联。

### 知识点
- `INNER JOIN`：只保留两表都匹配的行
- `LEFT JOIN`：保留左表全部行，右表无匹配则为 `NULL`
- `RIGHT JOIN` / `FULL OUTER JOIN`
- 自连接（同一张表连接自己，常用于查上下级关系）

### 示例
查询每个员工及其订单明细（无订单的员工也要显示）：

```sql
SELECT e.emp_name, o.order_id, o.amount
FROM employees e
LEFT JOIN orders o ON e.emp_id = o.emp_id;
```

自连接查询每位员工及其直属经理姓名：

```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

找出"从未下过订单"的员工（LEFT JOIN + IS NULL 是经典写法）：

```sql
SELECT e.emp_name
FROM employees e
LEFT JOIN orders o ON e.emp_id = o.emp_id
WHERE o.order_id IS NULL;
```

### 练习
1. 用 `INNER JOIN` 结合 `GROUP BY`，统计每个部门的订单总金额。
2. 尝试写出同样效果的 `RIGHT JOIN` 版本，体会两者的等价关系。

---

## Day 4：子查询与公用表表达式（CTE）

**目标**：用子查询和 `WITH` 语句拆解复杂逻辑，让 SQL 更清晰。

### 知识点
- 标量子查询、`IN`/`EXISTS` 子查询
- 相关子查询（子查询引用外部表字段）
- `WITH ... AS (...)`（CTE）替代嵌套子查询，提升可读性

### 示例
查询薪资高于本部门平均薪资的员工（相关子查询）：

```sql
SELECT e1.emp_name, e1.department, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department
);
```

用 CTE 重写"每位员工订单总额"，使主查询更简洁：

```sql
WITH order_summary AS (
    SELECT emp_id, SUM(amount) AS total_amount
    FROM orders
    GROUP BY emp_id
)
SELECT e.emp_name, COALESCE(o.total_amount, 0) AS total_amount
FROM employees e
LEFT JOIN order_summary o ON e.emp_id = o.emp_id;
```

`EXISTS` 判断是否存在关联记录（通常比 `IN` 在大表上更高效）：

```sql
SELECT emp_name FROM employees e
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.emp_id = e.emp_id
);
```

### 练习
1. 用 CTE 分两步：先算部门平均薪资，再筛选出高于本部门均值的员工（对比 Day4 第一个例子，体会可读性差异）。
2. 用 `NOT EXISTS` 重写 Day3 中"从未下过订单的员工"查询。

---

## Day 5：窗口函数（进阶核心）

**目标**：窗口函数是从"能写查询"到"会写分析型 SQL"的关键分水岭。

### 知识点
- `ROW_NUMBER()`、`RANK()`、`DENSE_RANK()`
- `PARTITION BY`（分组内独立计算）+ `ORDER BY`
- 聚合函数配合 `OVER()` 实现"明细行 + 汇总值"共存
- `LAG()` / `LEAD()` 取上下行数据

### 示例
按部门统计薪资排名（同部门内排名）：

```sql
SELECT emp_name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

查询每笔订单金额，同时显示该员工的订单总额（不聚合成一行，保留明细）：

```sql
SELECT order_id, emp_id, amount,
       SUM(amount) OVER (PARTITION BY emp_id) AS emp_total
FROM orders;
```

计算每位员工按入职时间排列后，前一位入职员工的薪资差（`LAG`）：

```sql
SELECT emp_name, hire_date, salary,
       salary - LAG(salary) OVER (ORDER BY hire_date) AS salary_diff
FROM employees;
```

### 练习
1. 找出每个部门薪资最高的前 2 名员工（提示：用 `RANK()` 或 `ROW_NUMBER()` 结合外层 `WHERE`/CTE 过滤）。
2. 用 `DENSE_RANK()` 和 `RANK()` 分别处理有并列薪资的情况，对比两者结果差异。

---

## Day 6：索引原理与查询优化

**目标**：知道"怎么写"之外，理解"为什么慢"，这是进阶到中高级的必经之路。

### 知识点
- 索引的基本原理（B+树，为何能加速等值/范围查询）
- 哪些操作会导致索引失效：对索引列做函数运算、隐式类型转换、`LIKE '%xxx'` 前置通配符
- 执行计划：`EXPLAIN` 的基本用法
- 避免 `SELECT *`、合理使用覆盖索引

### 示例
查看某条查询的执行计划（不同数据库语法略有差异，以 MySQL/PostgreSQL 通用写法为例）：

```sql
EXPLAIN
SELECT emp_name FROM employees WHERE department = '技术部';
```

索引失效的典型反例（对索引列做了函数运算，索引可能无法生效）：

```sql
-- 不推荐：对 hire_date 做了函数处理
SELECT * FROM employees WHERE YEAR(hire_date) = 2023;

-- 推荐：改写为范围查询，可以命中索引
SELECT * FROM employees
WHERE hire_date >= '2023-01-01' AND hire_date < '2024-01-01';
```

### 练习
1. 找一张业务表，为常用查询条件的字段建立索引，用 `EXPLAIN` 对比建索引前后的执行计划差异。
2. 尝试把一个包含 `LIKE '%关键词%'` 的模糊查询，思考除了索引优化外还有哪些替代方案（如全文索引）。

---

## Day 7：事务、综合案例与知识串联

**目标**：把前六天的知识点串联起来，处理一个接近真实业务的综合场景。

### 知识点
- 事务基本概念：`BEGIN` / `COMMIT` / `ROLLBACK`，ACID 特性简述
- `CASE WHEN` 条件表达式在查询中的应用
- 综合案例：多表 JOIN + 聚合 + 窗口函数 + CTE 一起使用

### 示例：事务保证转账场景的原子性

```sql
BEGIN;

UPDATE employees SET salary = salary - 1000 WHERE emp_id = 1;
UPDATE employees SET salary = salary + 1000 WHERE emp_id = 2;

-- 确认无误后提交，若中途出错应 ROLLBACK
COMMIT;
```

### 综合案例
需求：生成一份"部门业绩报告"，要求展示每个部门的员工数、总订单金额，并标记该部门是否为"高业绩部门"（总订单金额超过 50000）。

```sql
WITH dept_orders AS (
    SELECT e.department,
           COUNT(DISTINCT e.emp_id) AS headcount,
           COALESCE(SUM(o.amount), 0) AS total_amount
    FROM employees e
    LEFT JOIN orders o ON e.emp_id = o.emp_id
    GROUP BY e.department
)
SELECT department,
       headcount,
       total_amount,
       CASE WHEN total_amount > 50000 THEN '高业绩部门'
            ELSE '普通部门' END AS performance_tag,
       RANK() OVER (ORDER BY total_amount DESC) AS dept_rank
FROM dept_orders
ORDER BY total_amount DESC;
```

这个案例综合运用了：CTE（简化逻辑）、LEFT JOIN（保留无订单员工）、聚合函数（COUNT/SUM）、CASE WHEN（业务标签）、窗口函数（排名）。

### 练习（结业挑战）
1. 在此基础上，增加一列：每个部门内订单金额最高的员工姓名（提示：需要另一个 CTE + `ROW_NUMBER()`）。
2. 尝试将整个查询包装进一个事务中，模拟"生成报告前先锁定数据快照"的场景（仅作语法练习，理解事务隔离级别的意义）。

---

## 学习建议

- 每天的练习题不要只在脑内推演，务必在数据库中实际执行、查看结果。
- 遇到报错优先看 `EXPLAIN`/错误信息，逐步定位问题，而非直接搜索整段代码。
- 七天结束后，建议找一份公开数据集（如电商订单数据）重新完整走一遍本指南的知识点，检验是否能独立完成。
