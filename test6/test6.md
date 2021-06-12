# 基于 Oracle 的七号路书籍交易系统数据库设计

### 1.创建表空间

#### space_qhl001

```sql
Create Tablespace space_qhl001
datafile
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_qhl001_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED,
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_qhl001_2.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

#### space_qhl002

```sql
Create Tablespace space_qhl002
datafile
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_qhl002_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED,
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_qhl002_2.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

![pict1](pict1.png)

### 2. 创建角色及用户

用户默认使用表空间 space_qhl001

#### 创建第一个角色和用户

- 创建角色 qhl1 将 connect,resource,create view 授权给 qhl1
- 创建用户 qhl_1
- 分配 60M 空间给 qhl_1 并将角色 qhl1 授权给用户 qhl_1

```sql
CREATE ROLE qhl1;
GRANT connect,resource,CREATE VIEW TO qhl1;
CREATE USER qhl_1 IDENTIFIED BY 123 DEFAULT TABLESPACE space_qhl001 TEMPORARY TABLESPACE temp;
ALTER USER qhl_1 QUOTA 60M ON space_qhl001;
GRANT qhl1 TO qhl_1;
```

![pict2](pict2.png)

#### 创建第二个角色和用户

- 创建角色 qhl2，将 connect,resource 权限给 qhl2
- 创建用户 qhl_2
- 分配 60M 空间给 qhl_2 并将角色 qhl2 授权给用户 qhl_2

```sql
CREATE ROLE qhl2;
GRANT connect,resource TO qhl2;
CREATE USER qhl_2 IDENTIFIED BY 123 DEFAULT TABLESPACE space_qhl001 TEMPORARY TABLESPACE temp;
ALTER USER qhl_2 QUOTA 60M ON space_qhl001;
GRANT qhl2 TO qhl_2;
```

![pict3](pict3.png)

### 3. 在用户 qhl_1 下创建表

#### 创建管理员表

- id 为主键

```sql
CREATE TABLE ADMINISTRATOR
(
  ID NUMBER(*, 0) NOT NULL
, PASSWORD VARCHAR2(20 BYTE) NOT NULL
, ADMIN VARCHAR2(20 BYTE) NOT NULL
, CONSTRAINT ADMINISTRATOR_PK PRIMARY KEY
  (
    ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX ADMINISTRATOR_PK ON ADMINISTRATOR (ID ASC)
      LOGGING
      TABLESPACE SPACE_QHL001
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
LOGGING
TABLESPACE SPACE_QHL001
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NO INMEMORY
NOPARALLEL;
```

#### 创建用户表

- id 为主键
- 根据注册日期按范围分区
- 分为 2018 和 2019 年两个分区，每年按季度划 4 个子分区

```sql
CREATE TABLE BOOKUSER
(
  ID NUMBER(*, 0) NOT NULL
, PASSWORD VARCHAR2(20 BYTE) NOT NULL
, USERNAME VARCHAR2(50 BYTE) NOT NULL
, PHONE VARCHAR2(20 BYTE) NOT NULL
, ADDRESS VARCHAR2(30 BYTE) NOT NULL
, REGISTRATIONDATE DATE NOT NULL
, CART_ID NUMBER(*, 0) NOT NULL
, CONSTRAINT U_PK PRIMARY KEY
  (
    ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX U_PK ON BOOKUSER (ID ASC)
      LOGGING
      TABLESPACE SPACE_QHL001
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
TABLESPACE SPACE_QHL001
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NOPARALLEL
PARTITION BY RANGE (REGISTRATIONDATE)
SUBPARTITION BY RANGE (REGISTRATIONDATE)
(
  PARTITION DATE2018 VALUES LESS THAN (TO_DATE(' 2018-12-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
  (
    SUBPARTITION DATE2018_3 VALUES LESS THAN (TO_DATE(' 2018-03-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2018_6 VALUES LESS THAN (TO_DATE(' 2018-06-30 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2018_9 VALUES LESS THAN (TO_DATE(' 2018-09-30 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2018_12 VALUES LESS THAN (TO_DATE(' 2018-12-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  )
, PARTITION DATE2019 VALUES LESS THAN (TO_DATE(' 2019-12-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
  (
    SUBPARTITION DATE2019_3 VALUES LESS THAN (TO_DATE(' 2019-03-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2019_6 VALUES LESS THAN (TO_DATE(' 2019-06-30 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2019_9 VALUES LESS THAN (TO_DATE(' 2019-09-30 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  , SUBPARTITION DATE2019_12 VALUES LESS THAN (TO_DATE(' 2019-12-31 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    NOCOMPRESS NO INMEMORY
  )
);
```

#### 创建商品表

```sql
CREATE TABLE COMMODITY
(
  ID NUMBER(*, 0) NOT NULL
, PID NUMBER(*, 0) NOT NULL
, BOOKSNAME VARCHAR2(20 BYTE) NOT NULL
, PRICE NUMBER NOT NULL
, DESCRIBE VARCHAR2(50 BYTE) NOT NULL
, NUM NUMBER(*, 0) NOT NULL
, ADMIN_ID NUMBER(*, 0) NOT NULL
, CONSTRAINT COMMODITY_PK PRIMARY KEY
  (
    ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX COMMODITY_PK ON COMMODITY (ID ASC)
      LOGGING
      TABLESPACE SPACE_QHL001
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
LOGGING
TABLESPACE SPACE_QHL001
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NO INMEMORY
NOPARALLEL;
```

#### 创建购物车表

- 用户表字段 BOOKUSER_ID 为购物车表的外键
- 购物车采用引用分区

```sql
CREATE TABLE CART
(
  ID NUMBER(*, 0) NOT NULL
, AMOUNT NUMBER(*, 0) NOT NULL
, PID NUMBER(*, 0) NOT NULL
, BOOKUSER_ID NUMBER(*, 0) NOT NULL
, CONSTRAINT CART_PK PRIMARY KEY
  (
    ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX CART_PK ON CART (ID ASC)
      LOGGING
      TABLESPACE SPACE_QHL001
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
, CONSTRAINT CART_BOOKUSER FOREIGN KEY
  (
  BOOKUSER_ID
  )
  REFERENCES BOOKUSER
  (
  CART_ID
  )
  ENABLE
)
PCTFREE 10
PCTUSED 40
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NOPARALLEL
PARTITION BY REFERENCE (CART_BOOKUSER)
(
  PARTITION DATE2018_3
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2018_6
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2018_9
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2018_12
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2019_3
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2019_6
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2019_9
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION DATE2019_12
  LOGGING
  TABLESPACE SPACE_QHL001
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
);
```

#### 创建论坛表

```sql
CREATE TABLE TABLE1
(
  ID INT NOT NULL
, CONTENT NVARCHAR2(50) NOT NULL
, CONSTRAINT TABLE1_PK PRIMARY KEY
  (
    ID
  )
  ENABLE
);
```

#### 完整表结构如下图所示

![pict4](pict4.png)

#### 创建视图

- 计算每个用户购车中单个商品的总价

```sql
create or replace  view view_SinglePriceSum
as
select b.id,b.username,co.booksname,(co.price*ca.amount) pricesum from COMMODITY co,cart ca,BOOKUSER b where co.pid=ca.pid and ca.BOOKUSER_ID =b.id;
select * from view_SinglePriceSum;
```

![pict5](pict5.png)

#### 用户 qhl_1 空间不足，修改 qhl_1 空间大小

```sql
ALTER USER qhl_1 QUOTA 90M ON space_qhl001;
```

### 4. 插入用户、商品、购物车数据

```sql
declare
  id number(38,0);
  username varchar2(50);
  phone varchar2(20);
  address varchar2(30);
  REGISTRATIONDATE date;
  booksname varchar2(50);
  price number(5,2);
  num number(38,0);
  amount number(38,0);

begin
  for i in 1..50000
  loop
    if i mod 2 =0 then
      REGISTRATIONDATE:=to_date('2018-5-6','yyyy-mm-dd')+(i mod 60);
    else
      REGISTRATIONDATE:=to_date('2019-5-6','yyyy-mm-dd')+(i mod 60);
    end if;

    --插入用户
    id:=SEQ_ORDER_ID.nextval; --应该将SEQ_ORDER_ID.nextval保存到变量中。
    username := 'aa'|| 'aa';
    username := 'wang' || i;
    phone := '131785693' || i;
	booksname := '唐诗三百首版本号' || i;
	address :='成都'|| '四川';
	price :=(dbms_random.value() * 100);
	num :=(i mod 5);
    insert /*+append*/ into bookuser (id,password,username,phone,address,REGISTRATIONDATE,cart_id)
      values (id,username,username,phone,address,REGISTRATIONDATE,id);
	--插入货品

	insert into commodity(id,pid,booksname,price,describe,num,admin_id)
		values (id,id,booksname,price,'good',num,1);
	--插入购物车
	amount :=(id mod 3 ) + 1;
	insert into cart(id,amount,pid,bookuser_id)
	 	values (id,amount,id,id);

    IF I MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
end;
```

### 5.创建程序包、存储过程、函数执行分析计划

#### 创建程序包

- 函数 getcartsumprice 计算每个用户的购物车商品总金额
- 存储过程 adduser 插入用户信息

```sql
create or replace PACKAGE book_package Is
   function getcartsumprice(user_id number) return number;
   procedure adduser(password varchar2,username varchar2,phone varchar2,address varchar2,registerdate VARCHAR2);
end book_package;
```

#### 创建函数、存储过程

```sql
create or replace PACKAGE body book_package Is

       function getcartsumprice(user_id number) return number as
          begin
            declare cart_sum number;
			query_sql varchar2(200);
            begin
			query_sql:='select sum(pricesum) from view_SinglePriceSum where ID=' || user_id;
              execute immediate query_sql into cart_sum;
			  return cart_sum;
            end;
        end getcartsumprice;
                  procedure addUser(password varchar2,username varchar2,phone varchar2,address varchar2,registerdate varchar2) as
            begin
              declare maxId number;
              begin
                select max(id) into maxId from bookuser;
                insert into bookuser values(maxId+1,password,username,phone,address,to_date(registerdate,'yyyy-mm-dd'),maxId+1);
                commit;
              end;
            end adduser;
    end book_package;
```

![pict6](pict6.png)

#### 存储过程、函数执行分析

- 使用自定义函数 getcartsumprice（）查询 id 号为 20011 的用户购物车商品总价

```sql
select BOOK_PACKAGE.getcartsumprice(20011) from dual;
```

![pict7](pict7.png)

#### 使用存储过程 adduser 插入用户数据

```sql
set serveroutput on
declare
begin
BOOK_PACKAGE.addUser('131','wwj','123456','hongkong','2021-06-12');
end;
```

![pict8](pict8.png)

#### 执行计划分析

```sql
select * from BOOKUSER b,COMMODITY co,CART ca where b.id=ca.BOOKUSER_ID and ca.PID=co.PID and
b.REGISTRATIONDATE between to_date('2018-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');
```

#### 表空间使用状况

```sql
SELECT a.tablespace_name "表空间名",
total "表空间大小",
free "表空间剩余大小",
(total - free) "表空间使用大小",
total / (1024 * 1024 * 1024) "表空间大小(G)",
free / (1024 * 1024 * 1024) "表空间剩余大小(G)",
(total - free) / (1024 * 1024 * 1024) "表空间使用大小(G)",
round((total - free) / total, 4) * 100 "使用率 %"
FROM (SELECT tablespace_name, SUM(bytes) free
FROM dba_free_space
GROUP BY tablespace_name) a,
(SELECT tablespace_name, SUM(bytes) total
FROM dba_data_files
GROUP BY tablespace_name) b
WHERE a.tablespace_name = b.tablespace_name
```

![pict11](pict11.png)

### 6.备份恢复

#### 备份./rman_level0.sh

![pict12](pict12.png)

#### 查看备份内容

![pict13](pict13.png)
![pict14](pict14.png)

#### 删除数据

![pict15](pict15.png)

#### 恢复备份

![pic16](pict16.png)
![pict17](pict17.png)

#### 恢复完成

![pict18](pict18.png)
