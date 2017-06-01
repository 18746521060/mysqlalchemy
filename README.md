# Sqlalchemy文档

1. sqlalchemy的orm部分：
	1. 版本检查：
	```python
	import sqlalchemy
	print sqlalchemy.__version__
	``` 
	2. connecting:
	```
	# 导入模块
	from sqlalchemy import create_engine
	# 创建引擎
	engine = create_engine('...')
	```
	3. 声明映射-->可理解为创建一个基类
	```
	from sqlalchemy.ext.declarative import declarative_base
	Base = declarative_base()
	```
	然后就可以使用创建的基类来创建一个用户表
	```
	from sqlalchemy import Column,Integer,String
	class User(Base):
		__tablename__ = 'user'
		id = Column(Integer,primary_key=True)
		name = Column(String(10))
		pwd = Column(String(100))
		def __repr__(self):
			return '<User(id:%s,name:%s,pwd:%s)>'%(self.id,self.name,self.pwd)
	```
	在交互模式可以通过`类名+__table__`的形式查看sqlalchemy创建表的具体格式。如：`User.__table__`**注：这种操作仅限于交互模式下。**
	最后把模型映射到数据库中`Base.metadata.create_all(engine)`
	4. 创建一个映射实例：
	```
	use = User(name='python',pwd='123456')
	# 访问实例中的属性
	print user.name
	```
	**注：以上操作只存在于内存中**
	5. 创建一个会话
	```
	from sqlalchemy.orm import sessionmaker
	# 绑定会话引擎
	Session = sessionmaker(bind=engine)
	```
	或
	```
	Session = sessionmaker()
	Session.configure(bind=engine)
	```
	以上两者表示的是同一个意思，然后生成session。`session = Session()`
	6. 添加和更新Objects(实体)
	session通过add()添加对象实体。如：`session.add(user)`
	查询数据库中的内容：
	```
	user = session.query(User).filter_by(name='python').first()
	print user
	```
	session.add_all()可以添加多个实例对象，但其参数为list，如：
	```
	session.add_all([
		User(name='n1',pwd='p1'),
		User(name='n2',pwd='p2'),
		User(name='n3',pwd='p3'),...
	])
	```
	提交session添加的内容到数据库：
	```
	session.commit()
	```
	7. 回滚：(roll back)
	回滚只能操作session中的数据，而不能操作数据库中的数据。如：
	```
	user = User(name='flask',pwd='123')
	session.add(user)
	users = session.query(User).filter(User.name.in_(['python','flask'])).all()
	print user in users or 'fail'
	print 'success'
	session.rollback()
	users = session.query(User).filter(User.name.in_(['python','flask'])).all()
	print user in users or 'fail'
	print 'success'
	```
	如果在回滚前有提交`session.commit()`操作的话。回滚是失效的。
	8. Query对象返回的数据可迭代。
	9. 外键：
	```
	from sqlalchemy import ForeignKey
	from sqlalchemy.orm import relationship

	class Address(Base):
		__tablename__ = 'address'
		id = Column(Integer,primary_key=True)
		email = Column(String(20))
		user_id = Column(Integer,ForeignKey('user.id'))

		user = relationship('User',backref='addresses') 
	```
	10. 关联外键对象：
	```
	user = session.query(User).filter(User.name=='python').first()
	print user.addresses
	user.addresses = [
		Address(email='123@qq.com'),
		Address(email='456@qq.com'),
		Address(email='789@qq.com')
	]
	print user.addresses
	```
	11. 多表查询
	方法一：
	```
	for u,a in session.query(User,Address).filter(User.id==Address.id).filter(Address.email=='123@qq.com').all()
		print 'u:%s,a:%s'%(u,a)
	```
	方法二：
	```
	session.query(User).join(Address).filter(Address.email=='123@qq.com').all()
	```
	左外连接`outerjoin`。解释一下join与outerjoin：
	join的意思是匹配交集，outerjoin的意思是输出所有前面的并前面的匹配后面的。
	12. 别名：`Aliases`-->往往用于表的自身匹配。
	```
	from sqlalchemy.orm import aliased
	adalias1 = aliased(Address)
	adalias2 = aliased(Address)
	```
	13. 子查询
	```
	from sqlalchemy.sql import func
	rs = session.query(Address.user_id,func.count('*').label('address_count')).group_by(Address.user_id).subquery()
	```
	等同于
	```
	SELECT user.*,adr_count.address_count FROM users LEFT OUTER JOIN(SELECT user_id,count(*) AS address_count FROM addresses GROIP BY user_id) AS adr_count ON users.id=adr_count.user_id
	```
	14. EXISTS用法
	```
	from sqlalchemy.sql import exists
	rs = exists().where(Address.user_id==User.id)
	for name, in session.query(User.name).filter(rs):
		print name
	```
	等同于
	```
	SELECT user.name AS users_name FROM users WHERE EXISTS (SELECT * FROM addresses WHERE addresses.user_id = user.id)
	```
	15. 多对多
	多对多需要中间表
	关系属性需要更改，参数一都指向对方表，参数二用secondary指向第三张表，第三张表需要连接两张表的id作为主键
	16. '关系懒惰':
	是指relationship中添加了lazy属性并且赋值`dynamic`，返回的是query对象。默认等于`selector`。