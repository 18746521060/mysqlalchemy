# sqlalchemy连接mysql
> 一般有三种方法：
>
1. mysqldb：此方法一般用于python2，目前不提供python3的使用模块
2. mysqlconnector：此方法是mysql官方提供的方法，支持python3，但也仅仅支持到3.4版本。对于更高版本目前还不支持。
3. pymysql：使用方法和mysqldb几乎相同，而且支持python3.5及更高版本。 

# python连接mysql
**具体如例子：**
```python
import MySQLdb
conn = MySQLdb.connect(host='',port='',user='',passwd='',db='')
cur = conn.cursor()# 创建表
cur.excute("create table student(id int,name varchar(20),class varchar(30),age varchar(5))")
# 插入一条数据
cur.execute("insert into student values('2','Tom','3 year 2 class','9')")
# 修改查询条件的数据
cur.execute("update student set class='3 year 1 class' where name = 'Tom'")
# 删除查询条件的数据
cur.execute("delete from student where age = '9'")
# 查询数据
rs = cur.execute("select * from student")
print rs 或 cur.fetchone()
# 关闭cursor
cur.close()
# 提交
conn.commit()
# 关闭连接
conn.close()
```
  # python连接oracle > 下载对应版本的cx_Oracle。具体步骤如下 > 1. 引用cx_Oracle 2. 连接数据库 3. 获取cursor 4. 使用cursor进行各种操作 5. 关闭cursor 6. 关闭连接 **例子如下:** ``` import cx_Oracle conn = cx_Oracle.connect('用户名/密码@主机/数据库') c = conn.cursor() x = c.execute('select * from test1') # 输出查询结果-->先进先出 x.fetchone() c.close() conn.close() ``` 
