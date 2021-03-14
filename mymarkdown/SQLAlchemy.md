# SQLAlchemy 1.4/2.0

## mysql

### 查看sql执行记录

```sql
SET GLOBAL log_output = 'TABLE';SET GLOBAL general_log = 'ON';
SELECT * from mysql.general_log ORDER BY event_time DESC;
```

## 建立连接-引擎

### 语句执行基础

#### 获取行

```python
from sqlalchemy import create_engine, text

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

with engine.connect() as conn:
    # conn.execute(text("CREATE TABLE some_table (x int, y int)"))
    result = conn.execute(text("SELECT * from some_table"))
    # 返回的对象类似于python的命名元组类型，有以下几种访问方法
    r = result.all()
    # 1. 按照元组解包
    for x, y in r:
        print(x, y)
    # 2. 按照整数取索引
    for i in r:
        print(i[0], i[1])
    # 3. 按照名称取值
    for i in r:
        print(i.x, i.y)
    # 4. 按照映射访问
    for dict_w in result.mappings():
        print(dict_w["x"], dict_w["y"])
```

#### 发送参数

```python
from sqlalchemy import create_engine, text

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

with engine.connect() as conn:
    result = conn.execute(text("SELECT x, y FROM some_table WHERE y > :y"), {"y": 2})
    # 始终使用绑定参数,保证不会产生sql注入
    r = result.all()
    for x, y in r:
        print(x, y)

```

#### 发送多个参数

```python
from sqlalchemy import create_engine, text

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

with engine.connect() as conn:
    result = conn.execute(
        text("INSERT INTO some_table(x, y) VALUES(:x, :y)"),
        [{"x": 11, "y": 12}, {"x": 13, "y": 14}]
    )
    print(result)
```

#### 将参数与语句绑定

```python
from sqlalchemy import create_engine, text

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

with engine.connect() as conn:
    result = conn.execute(text("SELECT x, y FROM some_table WHERE y > :y").bindparams(y=2))
    r = result.all()
    for x, y in r:
        print(x, y)
```



### 使用ORM会话执行

```python
from sqlalchemy import create_engine, text
from sqlalchemy.orm import Session

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

with Session(bind=engine) as session:
    result = session.execute(text("SELECT x, y FROM some_table WHERE y > :y").bindparams(y=2))
    r = result.all()
    for x, y in r:
        print(x, y)
```

## 使用数据库元数据

### 元数据对象

* Metadata
* Table
* Column

### 设置元数据,DDL

```python
from sqlalchemy import create_engine
from sqlalchemy import Table, Column, Integer, String
from sqlalchemy import MetaData
from sqlalchemy import ForeignKey

metadata = MetaData()

engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

user_table = Table(
    "user_account", metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(50))
)

address_table = Table(
    "address",
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user_account.id'), nullable=False),
    Column('email_address', String(100), nullable=False)
)

metadata.create_all(engine)
# metadata.drop_all(engine)
```



## 用ORM定义表元数据

与上面的方法是等效的，这种orm的方式会自动生成table，可以使用__table__获取

```
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine
from sqlalchemy.orm import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)


class User(Base):
    __tablename__ = 'user_account'
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(50))
    addresses = relationship("Address", back_populates="user")

    def __repr__(self):
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"


class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email_address = Column(String(200), nullable=False)
    user_id = Column(Integer, ForeignKey('user_account.id'))
    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"


Base.metadata.create_all(engine)
```

## 基本操作数据

### 插入

#### insert语句和执行

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base
from sqlalchemy import insert

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()
user_table = Table(
    "user_account", metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(50))
)

address_table = Table(
    "address",
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user_account.id'), nullable=False),
    Column('email_address', String(200), nullable=False)
)

stmt = insert(user_table).values(name='spongebob', fullname="Spongebob Squarepants")
# print(stmt)
compiled = stmt.compile()
# print(compiled.params)

# with engine.connect() as conn:
#     result = conn.execute(stmt)
#     # 返回主键id
#     print(result.inserted_primary_key)

# 自动生成values子句，不需要使用text的sql
with engine.connect() as conn:
    result = conn.execute(
        insert(user_table),
        [
            {"name": "sandy", "fullname": "Sandy Cheeks"},
            {"name": "patrick", "fullname": "Patrick Star"}
        ]
    )

```



#### insert....select 

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base
from sqlalchemy import insert, select, bindparam

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()
user_table = Table(
    "user_account", metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(50))
)

address_table = Table(
    "address",
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user_account.id'), nullable=False),
    Column('email_address', String(200), nullable=False)
)

scalar_subquery = (
    select(user_table.c.id).
        where(user_table.c.name == bindparam('username')).limit(1).scalar_subquery()
)

with engine.connect() as conn:
    result = conn.execute(
        insert(address_table).values(user_id=scalar_subquery),
        [
            {"username": 'spongebob', "email_address": "spongebob@sqlalchemy.org"},
            {"username": 'sandy', "email_address": "sandy@sqlalchemy.org"},
            {"username": 'sandy', "email_address": "sandy@squirrelpower.org"},
        ]
    )

```

下面这种代码，可以避免将数据同步到本地然后再插入的现象，避免了网络IO

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base
from sqlalchemy import insert, select, bindparam

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()
user_table = Table(
    "user_account", metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(50))
)

address_table = Table(
    "address",
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user_account.id'), nullable=False),
    Column('email_address', String(200), nullable=False)
)

select_stmt = select(user_table.c.id, user_table.c.name + "@aol.com")

insert_stmt = insert(address_table).from_select(
    ["user_id", "email_address"], select_stmt
)

with engine.connect() as conn:
    result = conn.execute(insert_stmt)

```



#### insert...returning

```python
插入并返回数据
```



### 查询数据

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base, Session
from sqlalchemy import insert, select, bindparam

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()
user_table = Table(
    "user_account", metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(50))
)

address_table = Table(
    "address",
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user_account.id'), nullable=False),
    Column('email_address', String(200), nullable=False)
)

stmt = select(user_table).where(user_table.c.name == 'spongebob')

with engine.connect() as conn:
    result = conn.execute(stmt)
    for i in result.all():
        print(i)

```



#### 选择ORM实体和列

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base, Session, relationship
from sqlalchemy import insert, select, bindparam

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()


class User(Base):
    __tablename__ = 'user_account'
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(50))
    addresses = relationship("Address", back_populates="user")

    def __repr__(self):
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"


class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email_address = Column(String(200), nullable=False)
    user_id = Column(Integer, ForeignKey('user_account.id'))
    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"

# 自由选择需要查询的实体和列
stmt = select(User)
stmt = select(User.id, User.name)
# 笛卡尔积
stmt = select(User, Address)
print(select(User.id, Address.id))


with Session(bind=engine) as session:
    result = session.execute(stmt)
    for i in result.all():
        print(type(i))
        print(i)

```

#### label

```python
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine, Table, MetaData
from sqlalchemy.orm import declarative_base, Session, relationship
from sqlalchemy import insert, select, bindparam

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)

metadata = MetaData()


class User(Base):
    __tablename__ = 'user_account'
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(50))
    addresses = relationship("Address", back_populates="user")

    def __repr__(self):
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"


class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email_address = Column(String(200), nullable=False)
    user_id = Column(Integer, ForeignKey('user_account.id'))
    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"


stmt = (select(("Username: " + User.name).label("username"), ).order_by(User.name))
with engine.connect() as conn:
    for row in conn.execute(stmt):
        print(row.username)
#  或者     
with Session(bind=engine) as session:
    stmt = session.query(("Username: " + User.name).label("username")).order_by(User.name)
    for row in stmt.all():
        print(row.username)
```



#### join

* join_from():手动控制join表的左侧和右侧,print(select(User).join_from(Address, User))
* Join():指示联接的右侧,print(select(User).join(Address))
* select_from:显式指示from
  * print(select(User.id, Address.email_address).select_from(User).join(Address))
  * print(select(func.count("*")).select_from(User))

* on条件，join(address_table, isouter=True)
  * isouter：左连接
  * full：全连接

#### order by

#### group by/having

```python
with Session(bind=engine) as session:
    stmt = session.query(User.name, func.count(Address.id).label('count')).join(Address).group_by(User.name).having(
        func.count(Address.id) > 1)
    for row in stmt.all():
        print(row)
```

#### 按标签排序或分组

在给列起别名后还需要排序时比较有用

```python
with Session(bind=engine) as session:
    stmt = session.query(User.name, func.count(Address.id).label('count')) \
        .join(Address) \
        .group_by(User.name).order_by(desc('count'))
```

#### 使用别名

使用自连接时

```python
user_alias_1 = alias(User)
user_alias_2 = alias(User)

with Session(bind=engine) as session:
    stmt = session.query(user_alias_1.c.id,
                         user_alias_2.c.id)\
        .select_from(user_alias_1)\
        .join(user_alias_2, user_alias_1.c.id != user_alias_2.c.id)
```

#### 子查询和CTE

```python
# subquery
with Session(bind=engine) as session:
    sub_query = session.query(Address.user_id).group_by(Address.user_id).subquery()
    print(sub_query)
    stmt = session.query(User).join(sub_query, sub_query.c.user_id == User.id)
```

```python
# cte
with Session(bind=engine) as session:
    sub_query = session.query(Address.user_id).group_by(Address.user_id).cte()
    print(sub_query)
    stmt = session.query(User).join(sub_query, sub_query.c.user_id == User.id
```

```sql
# CTE生成的sql语句
WITH anon_1 AS ( SELECT count( address.id ) AS count, address.user_id AS user_id FROM address GROUP BY address.user_id ) 
SELECT
user_account.NAME,
user_account.fullname,
anon_1.count 
FROM
	user_account
	JOIN anon_1 ON user_account.id = anon_1.user_id
```

##### **还可以进行递归查询**，查看官方文档

#### 标量和相关子查询



#### exists

```python

```



### 更新和删除

#### 更新1：普通更新

```python
with Session(bind=engine) as session:
    stmt = (
             update(User).where(User.name == 'patrick').
             values(fullname='Patrick the Star')
         )
    print(stmt)
    result = session.execute(stmt)
```



#### 更新2：与原字段计算的更新

```python
with Session(bind=engine) as session:
    stmt = (
             update(User).
             values(fullname="Username: " + User.name)
         )
    print(stmt)
    result = session.execute(stmt)
```



#### 更新3：多条更新

```python
with Session(bind=engine) as session:
    stmt = (
        update(User).where(User.name == bindparam('oldname')).values(name=bindparam('newname'))
    )
    session.execute(stmt,
                    [
                        {'oldname': 'jack', 'newname': 'ed'},
                        {'oldname': 'wendy', 'newname': 'mary'},
                        {'oldname': 'jim', 'newname': 'jake'},
                    ]
                    )
    print(stmt)
```

#### 更新4:子查询

```python
with Session(bind=engine) as session:
    scalar_subq = (
           select(Address.email_address).
           where(Address.user_id == User.id).
           order_by(Address.id).
           limit(1).
           scalar_subquery()
         )
    update_stmt = update(User).values(fullname=scalar_subq)
    print(update_stmt)
```



#### 更新5：update。。。from。发现这种语句无法执行

```python
with Session(bind=engine) as session:
    update_stmt = (
            update(User).
            where(User.id == Address.user_id).
            where(Address.email_address == 'patrick@aol.com').
            values(fullname='Pat')
          )
    print(update_stmt)
```



#### 更新6：一次性更新多个表

```python
with Session(bind=engine) as session:
    update_stmt = (
            update(User).
            where(User.id == Address.user_id).
            where(Address.email_address == 'patrick@aol.com').
            values(
                {
                    User.fullname: "Pat",
                    Address.email_address: "pat@aol.com"
            }
    )
  )
    from sqlalchemy.dialects import mysql
    print(update_stmt.compile(dialect=mysql.dialect()))
```



#### 更新7：更新多个参数，并按照顺序更新

```python
with Session(bind=engine) as session:
    update_stmt = (
             update(User).
             ordered_values(
                 (User.name, '20'),
                 (User.fullname, User.name + '10')
                 )
         )
    print(update_stmt)
```



#### 普通删除

```python
with Session(bind=engine) as session:
    stmt = delete(User).where(User.name == 'patrick')
    print(stmt)
```



#### 关联多个表删除

```python
with Session(bind=engine) as session:
    delete_stmt = (
            delete(User).
            where(User.id == Address.user_id).
            where(Address.email_address == 'patrick@aol.com')
          )
    from sqlalchemy.dialects import mysql
    print(delete_stmt.compile(dialect=mysql.dialect()))
```



#### 使用return with UPDATE、DELETE



## 使用ORM操作数据

### 使用ORM插入行

```python
with Session(bind=engine) as session:
    squidward = User(name="squidward", fullname="Squidward Tentacles")
    krabs = User(name="ehkrabs", fullname="Eugene H. Krabs")
    # 此时对象处于pending(预备态)
    print(squidward, krabs)
    session.add(squidward)
    session.add(krabs)
    # 查看被挂起的对象，存储在内存中
    print(session.new)
    # flush,发送sql语句到数据库
    session.flush()
    # 调用commit时会自动flush,并且发起一个select查询
    session.commit()
    # 此时，对象处于persistent(持久态)
    print(1)
    # 身份(identity),被加载到内存中的对象会有一个id作为标识，当使用id获取对象时，会从内存获取，否则会向数据库发起一个查询
    session.get(User, krabs.id)
    print(1)
```



### 更新ORM对象

##### 更新1：先查询，再更新

```python
with Session(bind=engine) as session:
    user = session.query(User).filter(User.id == 15).first()
    print(session.dirty)
    user.name = '12'
    # 一旦对数据做出任何变动，此时会加入到脏数据集合
    print(session.dirty)
    # 一旦再次发起查询，这里会先去flush执行脏数据更新语句，然后查询一次
    new_user = session.query(User).filter(User.id == 15).first()
    # 这里name还是未修改的，因为update并未commit
    print(new_user.name)
    # 但是脏数据已经没了，因为update已经执行，只是未commit
    print(session.dirty)
```

##### 更新2：执行更新语句

```python

```



### 删除ORM对象

##### 删除1：先查询，再删除，直到发起下次语句前，delete才会被执行

##### 删除2：使用delete语句删除





### 回滚

### 关闭会话

close以后，所有的对象会变成分离状态

# ORM教程

## 1. 添加和更新机制

```python
from sqlalchemy.orm import declarative_base
from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine("mysql+pymysql://root:123456@localhost/abc", echo=True, echo_pool=True)
Session = sessionmaker(bind=engine)


class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(30))
    nickname = Column(String(30))

    def __repr__(self):
        return "<User(name='%s', fullname='%s', nickname='%s')>" % (
            self.name, self.fullname, self.nickname)


session = Session()

ed_user = User(name='ed', fullname='Ed Jones', nickname='edsnickname')
session.add(ed_user)
# 一旦发起查询，则所有的挂起会刷新到数据库
obj = session.query(User).filter(User.name == 'ed').first()
# 这里查询到的对象是一个的原因是，sqlalchemy本身创建了缓存
print(obj is ed_user)
# 在这里重新add三个对象,此时应该有三个new对象
session.add_all([
    User(name='wendy', fullname='Wendy Williams', nickname='windy'),
    User(name='mary', fullname='Mary Contrary', nickname='mary'),
    User(name='fred', fullname='Fred Flintstone', nickname='freddy')])
print(session.new)
# 此时更新ed_user的name，查看dirty数量会变成1个
ed_user.nickname = 'eddie'
print(session.dirty)
# 如果提交，则全部刷新,带着4个对象提交
session.flush()

# r = session.execute('SELECT * from mysql.general_log ORDER BY event_time DESC;')
# for i in r:
#     print(i)

```

## 2. 查询的几种方式

```python

# 传递实体类的方式,返回实体对象
for instance in session.query(User).order_by(User.id):
    print(type(instance))
    # print(instance.name, instance.fullname)

# 传递实体的列，返回命名元组
for row in session.query(User.name, User.fullname):
    print(type(row))

# 返回的元组是命名元组，并且可以像普通的Python对象一样进行处理。这些名称与属性的名称和类的类名相同
for row in session.query(User, User.name).all():
    print(type(row))
    # print(row.User, row.name)

# 使用label构造,返回命名元组
for row in session.query(User.name.label('name_label')).all():
    print(row)
    # print(row.name_label)

# 使用昵称构造，返回命名元组
from sqlalchemy.orm import aliased
user_alias = aliased(User, name='user_alias')
for row in session.query(user_alias, user_alias.name).all():
    print(row)
    # print(row.user_alias)
```

## 3. 返回标量的几种方式

```python

# 返回标量,如果无结果，则返回None
obj = session.query(User).filter(User.name == 'ed').first()
print(obj)

# 返回标量,如果无结果或者多个结果，则报错
obj = session.query(User).filter(User.name == 'ed').one()
print(obj)

# 返回标量,底层调用one方法，并返回第一列，但并不错
obj = session.query(User).filter(User.name == 'edi').scalar()
print(obj)
```

## 4. 使用text方式构造SQL

```python
from sqlalchemy import text
for user in session.query(User). \
        filter(text("id<224")). \
        order_by(text("id")).all():
    print(user.name)

# 基于字符串的SQL，使用冒号指定绑定参数
session.query(User).filter(text("id<:value and name=:name")).params(value=224, name='fred').order_by(User.id).one()

# 要使用完全基于字符串的语句, text无法表示完整语句的构造传递给 Query.from_statement() . 如果没有进一步的说明，ORM将根据列名将ORM映射中的列与SQL语句返回的结果相匹配：
session.query(User).from_statement(text("SELECT * FROM users where name=:name")).params(name='ed').all()

# 为了更好地将映射列定位到文本选择，以及以任意顺序匹配列的特定子集，将按所需顺序将各个映射列传递给 TextClause.columns() ：
stmt = text("SELECT name, id, fullname, nickname "
            "FROM users where name=:name")
stmt = stmt.columns(User.name, User.id, User.fullname, User.nickname)
session.query(User).from_statement(stmt).params(name='ed').all()

# 从中选择时 text() 构建 Query 仍可以指定要返回的列和实体；而不是 query(User) 我们也可以单独要求列，在任何其他情况下：
stmt = text("SELECT name, id FROM users where name=:name")
stmt = stmt.columns(User.name, User.id)
session.query(User.id, User.name).from_statement(stmt).params(name='ed').all()
```

## 5.count

```python
session = Session()
print(session.query(User).filter(User.name.like('%ed')).count())
# 如果只是返回条数，我们可以指明列，或者使用*
print(session.query(func.count('*')).select_from(User).scalar())
print(session.query(func.count(User.id)).scalar())
```

## 6.case的使用

```python
from sqlalchemy import case

c = case(
    (User.name == 'wendy', 'W'),
    (User.name == 'jack', 'J'),
    else_='E'
)

r = session.query(c, User.id, User.fullname).all()
```



# 映射配置

## 1. 使用mixin类组合

```python
class MyMixin(object):

    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    id = Column(Integer, primary_key=True)

# MyModel将包括id列，并生成指定的tablename
class MyModel(MyMixin, Base):
    name = Column(String(1000))
```

## 2. 扩展Base

```python
class MyBase:

    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    id = Column(Integer, primary_key=True)

Base = declarative_base(cls=MyBase)
# MyModel将包括id列，并生成指定的tablename
class MyModel(Base):
    name = Column(String(1000))
```

## 3. 更改列属性行为

```python
class EmailAddress(Base):
    __tablename__ = 'email_address'

    id = Column(Integer, primary_key=True)

    # name the attribute with an underscore,
    # different from the column name
    _email = Column("email", String)

    # then create an ".email" attribute
    # to get/set "._email"
    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, email):
        self._email = email
```

# session的使用



# SQL表达式语言教程



# 内核

