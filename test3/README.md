## 实验3：创建分区表
### 王年发    201810414318     软工18-3
### 实验目的
掌握分区表的创建方法，掌握各种分区方式的使用场景。
### 实验内容
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。
- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。
### 实验步骤
#### 1. 更改用户的状态
   ![avatar](1.png)
#### 2. 执行test3.sql，创建两个分区
   ![avatar](2.png)
#### 3. 查询插入的条数
   ![avatar](3.png)
#### 4.编写两条查询语句
   - 查询语句1
     ```
        set autotrace on
        select * from wnf.orders where order_date
        between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');
     ```
![avatar](4.png)
   - 查询语句2
     ```
        select a.ORDER_ID,a.CUSTOMER_NAME,
        b.product_name,b.product_num,b.product_price
        from wnf.orders a,wnf.order_details b where
        a.ORDER_ID=b.order_id and
        a.order_date between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');
     ```
![avatar](5.png)
### 查看数据库的使用情况
```
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
![avatar](6.png)
![avatar](7.png)
### 实验总结
通过本次的实验，学习了Oracle对表的分区，掌握了分区表的创建方法，还有一些数据的操作，实验过程中也遇到一些小问题，但都能顺利解决。