# 实验 4：对象管理

## 个人信息

201810414103 吴文杰 18 软工 1 班

## 实验目的

了解 Oracle 表和视图的概念，学习使用 SQL 语句 Create Table 创建表，学习 Select 语句插入，修改，删除以及查询数据，学习使用 SQL 语句创建视图，学习部分存储过程和触发器的使用。

## 实验场景

假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
实验内容：

## 实验内容

### 录入数据

要求至少有 1 万个订单，每个订单至少有 4 个详单。至少有两个部门，每个部门至少有 1 个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如 A 领导 B，B 领导 C）。

### 序列的应用

插入 ORDERS 和 ORDER_DETAILS 两个表的数据时，主键 ORDERS.ORDER_ID, ORDER_DETAILS.ID 的值必须通过序列 SEQ_ORDER_ID 和 SEQ_ORDER_ID 取得，不能手工输入一个数字。

### 触发器的应用

维护 ORDER_DETAILS 的数据时（insert,delete,update）要同步更新 ORDERS 表订单应收货款 ORDERS.Trade_Receivable 的值。

### 查询数据

```
1.查询某个员工的信息
2.递归查询某个员工及其所有下属，子下属员工。
3.查询订单表，并且包括订单的订单应收货款: Trade_Receivable= sum(订单详单表.ProductNum\*订单详单表.ProductPrice)- Discount。
4.查询订单详表，要求显示订单的客户名称和客户电话，产品类型用汉字描述。
5.查询出所有空订单，即没有订单详单的订单。
6.查询部门表，同时显示部门的负责人姓名。
7.查询部门表，统计每个部门的销售总金额。
```

### 表结构

![表 1](1.png)
![表 2](2.png)
![表 3](3.png)
![表 4](4.png)

## 实验步骤

### 1、创建文件夹 wwj

![步骤1](pict4-1.png)

### 2、使用系统用户创建表空间 Users04(截图被覆盖了，这是重新运行脚本的结果)

![步骤2](pict4-2.png)

### 3、查看文件夹 wwj

![步骤3](pict4-3.png)

### 4、给用户 user1 授予表空间 Users04 的使用权限

![步骤4](pict4-4.png)

### 5、删除表和序列（删除表的同时会一起删除主外键、触发器、程序包）

![步骤5](pict4-5.png)

### 6、创建表 DEPARTMENTS、EMPLOYEES、PRODUCTS、ORDER_ID_TEMP、ORDERS

![步骤6](pict4-6.png)
![步骤6](pict4-7.png)

### 7、创建本地分区索引 ORDERS_INDEX_DATE

![步骤7](pict4-8.png)

- 这个索引是必须的，可以使整个订单的详单存放在一起
  ![步骤7](pict4-9.png)
- 创建 3 个触发器
  ![步骤7](pict4-10.png)
- 批量插入订单数据之前，禁用触发器
  ![步骤7](pict4-11.png)

### 8、插入数据

![步骤8](pict4-12.png)
![步骤8](pict4-13.png)

## 测试

### 1、一切就绪，开始测试：以下时间在 0.02 秒以内才正常：（id 取值从 1 到 20000000）

![test1](test1.png)

### 2、递归查询某个员工及其所有下属，子下属员工

![test2](test2.png)

### 3、特殊查询语句：

```sql
--查询分区表情况:
select TABLE_NAME,PARTITION_NAME,HIGH_VALUE,PARTITION_POSITION,TABLESPACE_NAME from user_tab_partitions
--查询分区索引情况：
select * from USER_IND_PARTITIONS;
```

![test3](test3.png)

### 4、查询一个分区中的数据

![test4](test4.png)

### 5、统计完成后，查询表的统计信息

![test5](test5.png)
