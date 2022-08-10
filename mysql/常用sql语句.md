create  database mydb1;  创建一个名称为mydb1的 数据库。
use  db_name;  切换数据库 ;
show databases;  查看所有的数据库:
select  database(); 查看 当前数据库 ;
show  create database mydb2; 查看 数据库的创建 的具体的 信息;
show   create table table_name; 查看 表的创建 的具体的 信息;
show  tables; 查看数据 库内的所以表格;
desc 查看表的 数据的属性类型 ;
select*from table_name; 查看 表的数据;
show  character set; 查看可用 字符集;

创建一个使用utf8字符集的mydb2数据库。
create database mydb2 character set utf8;
创建一个使用utf8字符集，并带校对规则的mydb3数据库。
create database mydb3 character set utf8 collate utf8_general_ci;

删除数据库语句 
语法:  drop  database  数据库名;
drop database db_name;

查看服务器中的数据库，并把其中某一个库的字符集修改为utf8;
alter database mydb2  character set gbk;

删表:
drop table table_name;
添加列:
alter teble 表名 add 列名 列类型
修改列类型:
alter table 表名 modify 列名 列类型
删除列:
alter table 表名 drop 列名 列类型
删除行:
delete from 表名 where 行名=行值;