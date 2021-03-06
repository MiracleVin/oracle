# 实验2：用户及权限管理

## 实验目的

掌握用户管理、角色管理、权根维护与分配的能力，掌握用户之间共享对象的操作技能。

## 实验内容

Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：

- 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。
- 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
- 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。

## 实验步骤

1. 创建角色MiracleVin_view和用户MiracleVin,并对其进行授权和分配空间
![创建角色和用户](1.png)

2. 使用用户MiracleVin连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户
![创建表](2.png)

3. 用户hr连接到pdborcl，查询MiracleVin授予它的视图myview
![hr查看权限](3.png)

## 同学用户之间的表的共享

### 只读共享

1. 将mytable权限授权给new_zgq,kyrenexf

```sql
GRANT SELECT ON mytable TO new_zgq;
GRANT SELECT ON mytable TO kyrenexf;
```

![授权Select](5(gtk).png)

1. 我读对kyrenexf mytable的表进行查询

```sql
SELECT * FROM kyrenexf.mytable;
```

![查询](7sfk.png)

### 读写共享

1. 将mytable读写权限授权给new_zgq,kyrenexf

```sql
GRANT INSERT ON mytable TO new_zgq;
GRANT INSERT ON mytable TO kyrenexf;
```

![授权Insert](6(itk).png)

1. 我读对new_zgq zgqtable的表进行读写和查询
写：

```sql
INSERT INTO new_zgq.zgqtable(id,name)VALUES('2','Vin');
```

![写](8(itz).png)
查询:

```sql
SELECT * FROM new_zgq.zgqtable;
```

![查询](9(sfz).png)

## 数据库和表空间占用分析

> 当全班同学的实验都做完之后，数据库pdborcl中包含了每个同学的角色和用户。
> 所有同学的用户都使用表空间users存储表的数据。
> 表空间中存储了很多相同名称的表mytable和视图myview，但分别属性于不同的用户，不会引起混淆。
> 随着用户往表中插入数据，表空间的磁盘使用量会增加。

## 查看数据库的使用情况

以下样例查看表空间的数据库文件，以及每个文件的磁盘占用情况。

```sql
$ sqlplus system/123@pdborcl

SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```

- autoextensible是显示表空间中的数据文件是否自动增加。
- MAX_MB是指数据文件的最大容量。
![数据库的使用情况](4.png)

## 实验总结

通过本次实验了解了什么是角色和用户，掌握了如何创建角色和用户，以及对其进行权限分配。
