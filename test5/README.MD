# 实验 5：PL/SQL 编程

## 个人信息

201810414103 吴文杰 18 软工 1 班

## 实验目的

```
了解 PL/SQL 语言结构
了解 PL/SQL 变量和常量的声明和使用方法
学习条件语句的使用方法
学习分支语句的使用方法
学习循环语句的使用方法
学习常用的 PL/SQL 函数
学习包，过程，函数的用法。
```

## 实验场景

- 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
- 本实验以实验四为基础

## 实验内容

1. 创建一个包(Package)，包名是 MyPack。

```sql
create or replace PACKAGE MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END MyPack;
```

![步骤1](pict5-1.png)

2. 在 MyPack 中创建一个函数 SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销售额之和。函数 SaleAmount 要求输入的参数是部门号，输出部门的销售金额。

```sql
create or replace PACKAGE BODY MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
    N NUMBER(20,2);
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;
```

3. 在 MyPack 中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的 ID,姓名，销售总金额。信息用 dbms_output 包中的 put 或者 put_line 函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。比如下面显示 5 个员工的信息：

```sql
PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                             V.EMPLOYEE_ID||' '||v.NAME);
      END LOOP;
    END;
END MyPack;
```

![步骤2、3](pict5-2.png)

4. 由于订单只是按日期分区的，上述统计是全表搜索，因此统计速度会比较慢，如何提高统计的速度呢？

```
ID 姓名 销售总金额
=======================
2 张三   5000元
3 李四   6000元
4 王五   1000元
5 孙强   2000元
6 赵强东 5000 元
```

上述输出结果表示 2，6 号两个员工是一个级别的，没有领导，3，4 号员工的领导是 2 号，5 号员工的领导是 4 号。 Oracle 递归查询的语句格式是：

```sql
SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID
```

![测试1](test1.png)
![测试2](test2.png)
