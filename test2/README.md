## 实验2：用户及权限管理
### 王年发    201810414318     软工18-3

### 实验目的
 - 掌握用户管理、角色管理、权根维护与分配的能力。
 - 掌握用户之间共享对象的操作技能。
### 实验内容
Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：
- 在pdborcl插接式数据中创建一个新的本地角色BroFaa，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有BroFaa的用户就同时拥有这三种权限。
- 创建角色之后，再创建用户wnf，给用户分配表空间，设置限额为50M，授予BroFaa角色。
- 最后测试：用新用户wnf连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。
### 实验步骤：
- 第1步：以system登录到pdborcl，创建角色BroFaa和用户wnf，并授权和分配空间：
 
```
SQL> CREATE ROLE BroFaa;

角色已创建。

SQL> GRANT connect,resource,CREATE VIEW TO BroFaa;

授权成功。

SQL> CREATE USER wnf IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

用户已创建。

SQL> ALTER USER wnf QUOTA 50M ON users;

用户已更改。

SQL> GRANT BroFaa TO wnf;

授权成功。

SQL> exit

```

- 新用户wnf连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户：
```
$ sqlplus wnf/123@pdborcl

SQL*Plus: Release 12.2.0.1.0 Production on 星期一 3月 22 11:25:42 2021

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

上次成功登录时间: 星期一 3月  22 2021 11:15:20 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> show user;
USER 为 "WNF"
SQL> CREATE TABLE mytable (id number,name varchar(50));

表已创建。

SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');

已创建 1 行。

 INSERT INTO mytable(id,name)VALUES (2,'wang');

已创建 1 行。

SQL> CREATE VIEW myview AS SELECT name FROM mytable;

视图已创建。

SQL> SELECT * FROM myview;

NAME
--------------------------------------------------
zhang
wang

SQL> GRANT SELECT ON myview TO hr;

授权成功。

SQL> exit

```
- 第3步：用户hr连接到pdborcl，查询new_user授予它的视图myview
```
[student@deep02 ~]$ sqlplus hr/123@pdborcl

SQL*Plus: Release 12.2.0.1.0 Production on 星期一 3月 22 11:59:45 2021

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

上次成功登录时间: 星期一 3月  22 2021 11:47:22 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> SELECT * FROM wnf.myview;

NAME
--------------------------------------------------
zhang
wang

SQL> exit

```


#### 查看数据库的使用情况  
```
SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

TABLESPACE_NAME
------------------------------
FILE_NAME
--------------------------------------------------------------------------------
        MB     MAX_MB AUT
---------- ---------- ---
USERS
/home/oracle/app/oracle/oradata/orcl/pdborcl/users01.dbf
         5 32767.9844 YES

SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;

 表空间名                           大小MB     剩余MB     使用MB    使用率%
------------------------------ ---------- ---------- ---------- ----------
SYSAUX                                660    40.5625   619.4375      93.85
UNDOTBS1                              100    61.1875    38.8125      38.81
USERS                                   5       .375      4.625       92.5
SYSTEM                                270      8.125    261.875      96.99

```