---
title: SQLAlchemy ORM 学习笔记
date: 2021-07-19 23:10:07
tags:
  - Python
  - 学习笔记
categories:
  - Python
---

## Object Relational Tutorial

所谓ORM（Object Relational Mapping），就是建立其由Python类到数据库表的映射关系：一个Python实例(*instance*) 对应数据库中的一行(*row*)。

这种映射包含两层含义，一是实现对象和与之关联的的行的状态同步，二是将涉及数据库的查询操作，表达为 Python 类的相互关系。

注意 ORM 和 SQLAlchemy 的 Expression Language 不同。后者可以视为对原始SQL的封装。ORM是基于Expression Language 而构建的，其抽象层次要高于 Expression Language。很多时候都是使用ORM，有时需要一些高度定制化的功能时，就需要使用到 Expression Language。

## 基础操作

### 查看Version

可以用下面的命令来查看 SQLAlchemy 的版本

```python
>>> import sqlalchemy
>>> sqlalchemy.__version___
1.4.20
```

### 连接

```python
>>> from sqlalchemy import create_engine
>>> engine = create_engine('sqlite:///:memory:', echo=True)
```

这里的 `echo` 设置为True可以使得后面可以在控制台看到操作涉及的 SQL 语言。

`create_engine` 返回的是一个 `Engine` 实例，它代表了指向数据库的一些非常核心的接口。他会根据你选择的数据库配置而调用对应的 `DBAPI`。

当第一次如 `Engine.execute()` 或者 `Engine.connect()` 的方法被调用时，`Engine` 才会真正的建立起到数据库的 `DBAPI` 连接。实际上，一般并不会直接使用 `Engine`。

### 声明映射

当使用 ORM 的时候，其配置过程主要分为两个部分：

1. 描述我们要处理的数据库表的信息，

2. 将我们的Python类映射到这些表上。

这两个过程在 SQLAlchemy 中是一起完成的，这个过程称之为 **Declarative**。

使用 Declarative 参与 ORM 映射的类需要被定义成为一个指定基类的子类，这个基类应当含有ORM映射中相关的类和表的信息。这样的基类称之为**declarative base class**。在应用中，一般只需要一个这样的基类。这个基类可以通过 `declarative_base` 来创建

```python
>>> from sqlalchemy.ext.declarative import declarative_base
>>> Base = declarative_base()
```

现在已经有了一个基类，就可以基于这个基类来创建自定义类了。以建立一个用户类为例子。从 `Base` 派生一个名为 `User` 的类，在这个类里面可以定义将要映射到数据库的表上的属性（主要是表的名字，列的类型和名称等）：

```python
from sqlalchemy import Column, Integer, String
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    password = Column(String)

    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" % (
                             self.name, self.fullname, self.password)
```

通过 Declarative 生成的类至少应该包含一个名为 **tablename** 的属性来给出目标表的名称，以及至少一个`Column` 来给出表的主键( Primary Key )。SQLAlchemy 不会对于类名和表名之间的关联做任何假设，也不会自动涉及数据类型以及约束的转换。一般可以自己创建一个模板来建立这些自动转换，这样可以减少很多重复劳动。

当类声明完成后，Declarative 将会将所有的 `Column` 成员替换成为特殊的 Python 访问器( accessors )，这称之为 **descriptors**。这个过程称为 **instrumentation**，经过 instrumentation 的映射类可以读写数据库的表和列。

**注意除了这些涉及ORM的映射意外，这些mapping类的其他部分仍然是不变的。**

### 创建模板

通过 Declarative 系统构建好 `User` 类之后，与之同时的关于表的信息也已经创建好了，其称之为 **table metadata**。描述这些信息的类为 `Table`。可以通过 `__table__` 这个类变量来查看表信息

```python
>>> User.__table__ 
Table('users', MetaData(bind=None),
            Column('id', Integer(), table=<users>, primary_key=True, nullable=False),
            Column('name', String(), table=<users>),
            Column('fullname', String(), table=<users>),
            Column('password', String(), table=<users>), schema=None)
```

当完成类声明时，Declarative 用一个 Python 的 metaclass 来为这个类进行了加工。在这个阶段，它依据给出的设置创建了 `Table` 对象，然后构造一个 `Mapper` 对象来与之关联。

`Table` 对象是一个更大家庭---- `MetaData` ----的一部分。当使用 Declarative 时，这个对象也可以在 Declarative base class 的 `.metadata` 属性中看到。

`MetaData` 是与数据库打交道的一个接口。对于的SQLite数据库而言，此时还没有一个名为`users`的表的存在，这时需要使用 `MetaData` 来发出 `CREATE TABLE` 的命令。下面使用 `MetaData.create_all()` 指令，将上面得到的 `Engine` 作为参数传入。如果上面设置了 echo 为 True 的话，应该可以看到这一过程中的SQL指令。首先检查了 `users` 表的存在性，如果不存在的话会执行表的创建工作。

```python
>>> Base.metadata.create_all(engine)
SELECT ...
PRAGMA table_info("users")
()
CREATE TABLE users (
    id INTEGER NOT NULL, name VARCHAR,
    fullname VARCHAR,
    password VARCHAR,
    PRIMARY KEY (id)
)
()
COMMIT
```

### 创建映射类实例

创建 `User` 对象十分简单

```python
>>> ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
>>> ed_user.name
'ed'
>>> ed_user.password
'edspassword'
>>> str(ed_user.id)
'None'
```

### 创建会话

Session 是一个非常重要的概念，类似于iOS中的 NSManagedContext 的概念。

现在可以和数据库对话了。ORM 对数据库的入口即是 `Session`，当构建应用时，和 `create_engine` 的同一级别下，定义一个 `Session` 类来作为生成新的 Session 的 Factory 类。

```python
>>> from sqlalchemy.orm import sessionmaker
>>> Session = sessionmaker(bind=engine)
```

当试图在定义 `Engine` 之前定义 `Sesssion` 的话，这里的 `bind` 可以不设置

```python
>>> Session = sessionmaker()
```

后续定义好 `Engine` 后可以通过 `configure()` 来将其连接到 `Session`

```python
>>> Session.configure(bind=engine)  # once engine is available
```

这个自定义的工厂类就可以拿来构造新的 `Session` 了。

```python
>>> session = Session()
```

上面的 `Session` 已经和 SQLite 的数据库的 `Engine` 关联起来了，但是可以发现它还没有打开任何到数据库的连接( connection )。当一个 `Session` 被首次使用时，它会从 `Engine` 所维护的连接池中取出一个连接来操作数据库。这个连接在应用有所更改或者关闭 `Session` 时会被释放。

### 添加和更新对象

为了将 `User` 对象存入数据库，可以调用 `Sesson` 的 `add()` 函数

```python
>>> ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
>>> session.add(ed_user)
```

当这个操作完成之后，这个 `User` 实例的状态为 **pending**。目前实际上还没有执行SQL操作，也就是说数据库中还没有产生和这个 `User` 实例对应的行。`Session` 将会在需要的时候执行相应的SQL命令，这个过程称之为**flush**。如果试图查询 `Ed Jones`，所有处于 `pending` 状态的信息将会首先被 **flush**，然后负责进行查询的 SQL 语言在此之后立即被执行。

例如，创建一个查询来获取刚刚创建的用户。这个查询会返回一个和之前添加的用户相同的用户实例。

```python
>>> our_user = session.query(User).filter_by(name='ed').first() BEGIN (implicit)
INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
('ed', 'Ed Jones', 'edspassword')
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
 LIMIT ? OFFSET ?
('ed', 1, 0)
>>> our_user
<User(name='ed', fullname='Ed Jones', password='edspassword')>
```

事实上这里的 `Session` 判断出来了需要返回的行和已经存在内存中的一个映射实例应当是同一个，所以会得到一个和之前完全相同的实例

```python
>>> ed_user is our_user
True
```

这里ORM所表现的理念，称之为 [identity map](http://docs.sqlalchemy.org/en/rel_1_0/glossary.html#term-identity-map)。这个设计理念保证了在一个 `Session` 对于一个制定行的操作，作用于同一个内存实例上。当一个拥有特定主键的对象出现在 `Session` 中时，所有的查询操作对这个主键都会返回一个相同的 Python 对象。并且，如果试图引入重复了主键的新的对象时，系统会产生一个错误来阻止你的操作。

可以通过 `add_all()` 来一次加入多个对象

```python
>>> session.add_all([
...     User(name='wendy', fullname='Wendy Williams', password='foobar'),
...     User(name='mary', fullname='Mary Contrary', password='xxg527'),
...     User(name='fred', fullname='Fred Flinstone', password='blah')])
```

并且，如果希望改变Ed的密码，可以直接修改之：

```python
>>> ed_user.password = 'f8s7ccs'
```

这个修改会被 `Session` 记录下来

```python
>>> session.dirty
IdentitySet([<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>])
```

当然，上面的插入操作也被记录了

```python
>>> session.new 
IdentitySet([<User(name='wendy', fullname='Wendy Williams', password='foobar')>,
<User(name='mary', fullname='Mary Contrary', password='xxg527')>,
<User(name='fred', fullname='Fred Flinstone', password='blah')>])
```

可以使用 `commit()` 命令来将这些更改 **flush** 到数据库中

```python
>>> session.commit()
```

## 查询(Query)

`Session` 的 `query` 函数会返回一个 `Query` 对象。`query` 函数可以接受多种参数类型。可以是类，或者是类的instrumented **descriptor**。下面的这个例子取出了所有的 `User` 记录。

```python
>>> for instance in session.query(User).order_by(User.id):
...     print(instance.name, instance.fullname)
ed Ed Jones
wendy Wendy Williams
mary Mary Contrary
fred Fred Flinstone
```

`Query` 也接受 ORM-instrumented descriptors 作为参数。当多个参数传入时，返回结果为以同样顺序排列的tuples

```python
>>> for name, fullname in session.query(User.name, User.fullname):
...     print(name, fullname)
ed Ed Jones
wendy Wendy Williams
mary Mary Contrary
fred Fred Flinstone
```

`Query` 返回的 tuples 由 `KeyedTuple` 这个类提供，其成员除了用下标访问以外，还可以视为实例变量来获取。对应的变量的名称与被查询的类变量名称一样，如下例：

```python
>>> for row in session.query(User, User.name).all():
...    print(row.User, row.name)
<User(name='ed', fullname='Ed Jones', password='f8s7ccs')> ed
<User(name='wendy', fullname='Wendy Williams', password='foobar')> wendy
<User(name='mary', fullname='Mary Contrary', password='xxg527')> mary
<User(name='fred', fullname='Fred Flinstone', password='blah')> fred
```

可以通过 `label()` 来制定 descriptor 对应实例变量的名称

```python
>>> for row in session.query(User.name.label('name_label')).all():
...    print(row.name_label)
ed
wendy
mary
fred
```

而对于类参数而言，要实现同样的定制需要使用 `aliased`

```python
>>> from sqlalchemy.orm import aliased
>>> user_alias = aliased(User, name='user_alias')

>>> for row in session.query(user_alias, user_alias.name).all():
...    print(row.user_alias)
<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>
<User(name='wendy', fullname='Wendy Williams', password='foobar')>
<User(name='mary', fullname='Mary Contrary', password='xxg527')>
<User(name='fred', fullname='Fred Flinstone', password='blah')>
```

基本的查询操作除了上面这些之外，还包括 OFFSET 和 LIMIT，这个可以通过 Python 的 array slice 来完成。

```python
>>> for u in session.query(User).order_by(User.id)[1:3]:
...    print(u)
<User(name='wendy', fullname='Wendy Williams', password='foobar')>
<User(name='mary', fullname='Mary Contrary', password='xxg527')>
```

上述过程实际上只涉及了整体取出的操作，而没有进行筛选，筛选常用的函数是 `filter_by` 和 `filter`。其中后者比起前者要更灵活一些，可以在后者的参数中使用python的运算符。

```python
>>> for name, in session.query(User.name).filter_by(fullname='Ed Jones'):
...    print(name)
ed
>>> for name, in session.query(User.name).filter(User.fullname=='Ed Jones'):
...    print(name)
ed
```

注意 `Query` 对象是 **generative** 的，这意味可以把他们串接起来调用，如下：

```python
>>> for user in session.query(User).\
...          filter(User.name=='ed').\
...          filter(User.fullname=='Ed Jones'):
...    print(user)
<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>
```

**串接的 filter 之间是与的关系**

### 常用的filter操作符

下面的这些操作符可以应用在 `filter` 函数中

- **equals**

  ```python
  query.filter(User.name == 'ed')
  ```

- **not equals**

  ```python
  query.filter(User.name != 'ed')
  ```

- **like**

  ```python
  query.filter(User.name.like('%ed%'))
  ```

- **IN**

  ```python
  query.filter(User.name.in_(['ed', 'wendy', 'jack']))
  
  # 也适用于查询对象:
  query.filter(User.name.in_(
          session.query(User.name).filter(User.name.like('%ed%'))
  ))
  ```

- **NOT IN**

  ```python
  query.filter(~User.name.in_(['ed', 'wendy', 'jack']))
  ```

- **IS NULL**

  ```python
  query.filter(User.name == None)
  
  # 等同于, if pep8/linters are a concern
  query.filter(User.name.is_(None))
  ```

- **IS NOT NULL**

  ```python
  query.filter(User.name != None)
  
  # 等同于, if pep8/linters are a concern
  query.filter(User.name.isnot(None))
  ```

- **AND**

  ```python
  # 使用
  from sqlalchemy import and_
  query.filter(and_(User.name == 'ed', User.fullname == 'Ed Jones'))
  
  # 等同于
  query.filter(User.name == 'ed', User.fullname == 'Ed Jones')
  
  # 等同于
  query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')
  ```

- **OR**

  ```python
  from sqlalchemy import or_
  query.filter(or_(User.name == 'ed', User.name == 'wendy'))
  ```

- **MATCH**

  ```python
  query.filter(User.name.match('wendy'))
  ```

### 返回列表(List)和单项(Scalar)

`Query `有很多方法执行了 SQL 命令并返回了取出的数据库结果

- **all() 返回一个列表**

  ```python
  >>> query = session.query(User).filter(User.name.like('%ed')).order_by(User.id)
  SQL>>> query.all()
  [<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>,
        <User(name='fred', fullname='Fred Flinstone', password='blah')>]
  ```

- **first() 返回至多一个结果，而且以单项形式，而不是只有一个元素的 tuple 形式返回这个结果**

  ```python
  >>> query.first()
  <User(name='ed', fullname='Ed Jones', password='f8s7ccs')>
  ```

- **one() 返回且仅返回一个查询结果。当结果的数量不足一个或者多于一个时会报错**

  ```python
  >>> user = query.one()
  Traceback (most recent call last):
  ...
  MultipleResultsFound: Multiple rows were found for one()
  ```

  没有查找到结果时:

  ```python
  >>> user = query.filter(User.id == 99).one()
  Traceback (most recent call last):
  ...
  NoResultFound: No row was found for one()
  ```

- **one_or_none()：从名称可以看出，当结果数量为0时返回 None， 多于1个时报错**

- **scalar() 和 one() 类似，但是返回单项而不是 tuple**

### 嵌入使用SQL

可以在 `Query` 中通过 `text()` 使用SQL语句。例如：

```python
>>> from sqlalchemy import text
>>> for user in session.query(User).\
...             filter(text("id<224")).\
...             order_by(text("id")).all():
...     print(user.name)
ed
wendy
mary
fred
```

除了上面这种直接将参数写进字符串的方式外，还可以通过 `params()` 方法来传递参数

```python
>>> session.query(User).filter(text("id<:value and name=:name")).\
...     params(value=224, name='fred').order_by(User.id).one()
<User(name='fred', fullname='Fred Flinstone', password='blah')>
```

并且，可以直接使用完整的SQL语句，但是要注意将表名和列明写正确。

```python
>>> session.query(User).from_statement(
...                     text("SELECT * FROM users where name=:name")).\
...                     params(name='ed').all()
[<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>]
```

### 计数

`Query` 定义了一个很方便的计数函数 `count()`

```python
>>> session.query(User).filter(User.name.like('%ed')).count()
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
                users.name AS users_name,
                users.fullname AS users_fullname,
                users.password AS users_password
FROM users
WHERE users.name LIKE ?) AS anon_1
('%ed',)
2
```

注意上面同时列出了实际的 SQL 指令。在 SQLAlchemy 中，总是将被计数的查询打包成一个子查询，然后对这个子查询进行计数。即便是最简单的 `SELECT count(*) FROM table` ，也会如此处理。为了更精细的控制计数过程，可以采用 `func.count()` 这个函数。

```python
>>> from sqlalchemy import func
>>> session.query(func.count(User.name), User.name).group_by(User.name).all()
SELECT count(users.name) AS count_1, users.name AS users_name
FROM users GROUP BY users.name
()
[(1, u'ed'), (1, u'fred'), (1, u'mary'), (1, u'wendy')]
```

为了实现最简单的 `SELECT count(*) FROM table`，可以如下调用

```python
>>> session.query(func.count('*')).select_from(User).scalar()
SELECT count(?) AS count_1
FROM users
('*',)
4
```

如果对 `User` 的主键进行计数，那么 `select_from` 也可以省略。

```python
>>> session.query(func.count(User.id)).scalar()
SELECT count(users.id) AS count_1
FROM users
()
4
```

## 关系(Relationship)

『关系』是关系型数据库的一大特色，是在建模过程中的一个重要的抽象过程。

### 建立关系

之前已经建立了一个用户( User )表，现在考虑增加一个与用户关联的新的表。在系统里面，用户可以存储多个与之相关的 email 地址。这是一种基本的一对多的关系。这个新增加的存储 email 地址的表称为 `addresses`。应用 Declarative，按照如下方式定义这个新表：

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Address(Base):
    __tablename__ = 'addresses'
    id = Column(Integer, primary_key=True)
    email_address = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))

    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return "<Address(email_address='%s')>" % self.email_address

>>> User.addresses = relationship(
        "Address", order_by=Address.id, back_populates="user")
```

上面的代码中使用了一个新的名为 `ForeignKey` 的构造。其含义为，其所在的列的值域应当被限制在另一个表的指定列的取值范围之类。这一特性是关系型数据库的核心特性之一。就上例而言，`addresses.user_id` 这一列的取值范围，应当包含在 `users.id` 的取值范围之内。

除了 `ForeignKey` 之外，还引入了一个 `relationship`，来告诉ORM，`Address` 类需要被连接到 `User` 类。 `relationship` 和 `ForeignKey` 这个两个属性决定了表之间关系的属性，决定了这个关系是多对一的。

在完成对 `Address` 类的声明之后，还定义另一个 `relationship`，将其赋值给了 `User.addresses`。在两个`relationship` 中，都有传入了一个 `relationship.back_populates` 的属性来为反向关系所对应的属性进行命名。

多对一的关系的反向永远都是一对多的关系。关于更多的 `relationship()` 的配置方法，可以参见这个链接[Basic Relationship Patterns](http://docs.sqlalchemy.org/en/rel_1_0/orm/basic_relationships.html#relationship-patterns)。

上述我们定义的两个互补的关系 `Address.user` 和 `User.addresses` 被称为双向关系( [bidirectional relationship](http://docs.sqlalchemy.org/en/rel_1_0/glossary.html#term-bidirectional-relationship) )，这是 SQLAlchemy 的核心特性这一。

`relationship()` 的参数配置中指向被连接的类的字符串，可以指向工程中任何位置所定义的，基于`declarative base` 的类，而无先后之分。Declarative 会在完成所有的映射以后的将这些字符串转换为适当的、实际使用的参数形式。

### 使用关联对象

现在，当创建一个 `User` 实例的时候，会同时创建一个空的 `addresses` 的 collection。这个 collection 可能是多种类型，如 list, set, 或是 dictionary。默认情况下，其应当为一个 Python 列表。

```python
>>> jack = User(name='jack', fullname='Jack Bean', password='gjffdd')
>>> jack.addresses
[]
```

此时可以自由的向这个列表里面插入 `User` 对象。

```python
>>> jack.addresses = [
...                 Address(email_address='jack@google.com'),
...                 Address(email_address='j25@yahoo.com')]
```

当使用 bidirectional relationship 时，通过其中一个方向的关系（如上例）会自动出现在另一个方向的关系上。

```python
>>> jack.addresses[1]
<Address(email_address='j25@yahoo.com')>

>>> jack.addresses[1].user
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

把 jack 添加进入 `Session`。

```python
>>> session.add(jack)
>>> session.commit()
INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
('jack', 'Jack Bean', 'gjffdd')
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
('jack@google.com', 5)
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
('j25@yahoo.com', 5)
COMMIT
```

可以发现上面执行了三个 `INSERT` 命令，也就是说与 jack 关联的两个 `Address` 对象也被提交了。现在通过查询来取出jack。

```python
>>> jack = session.query(User).filter_by(name='jack').one()
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
('jack',)

>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

可以发现目前只有针对 `User` 表的查询，而没有对 `Address` 表的查询。此时访问 `addresses` 属性，相关的SQL才会执行

```python
>>> jack.addresses
SELECT addresses.id AS addresses_id,
        addresses.email_address AS
        addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses
WHERE ? = addresses.user_id ORDER BY addresses.id
(5,)
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

上面这种方式我们称之为 [lazy loading](http://docs.sqlalchemy.org/en/rel_1_0/glossary.html#term-lazy-loading) ,其实还可以在查询时就加载

#### [joinedload](https://www.osgeo.cn/sqlalchemy/orm/loading_relationships.html)

```python
>>> jack = session.query(User).options(joinedload(User.addresses)).\
    filter_by(name='jack').one()

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

### 使用join进行查询

现在有了两个彼此关联的数据表了

为了在 `User` 和 `Address` 之间构造一个简单的 join，可以通过 `Query.filter()` 来连接其相关列（本质是隐式写法的JOIN）。下面是一个简单的例子：

```python
>>> for u, a in session.query(User, Address).\
...                     filter(User.id==Address.user_id).\
...                     filter(Address.email_address=='jack@google.com').\
...                     all():
...     print(u)
...     print(a)
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
<Address(email_address='jack@google.com')>
```

而实际的 SQL JOIN 语法，可以通过 `Query.join()` 来想实现

```python
>>> session.query(User).join(Address).\
...         filter(Address.email_address=='jack@google.com').\
...         all()
users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users JOIN addresses ON users.id = addresses.user_id
WHERE addresses.email_address = ?
('jack@google.com',)
[<User(name='jack', fullname='Jack Bean', password='gjffdd')>]
```

在上面的例子中由于只存在一个 ForeignKey，`Query.join` 知道如何选取合适的列进行JOIN。如果没有定义ForeignKey，或者存在多个，此时你需要手动指明你参与 JOIN 的列。`Query.join()` 以如下方式进行：

```python
query.join(Address, User.id==Address.user_id)    # 显式条件
query.join(User.addresses)                       # 指定从左到右的关系
query.join(Address, User.addresses)              # 相同，有明确的目标
query.join('addresses')  
```

对于 OUTER JOIN，只需要使用 `Query.outerjoin()` 就可以了。

```python
query.outerjoin(User.addresses)   # 左外连接
```

关于 `join()` 更为详细的用法，还是请参考 [join 官方文档](http://docs.sqlalchemy.org/en/rel_1_0/orm/query.html#sqlalchemy.orm.query.Query.join)

### 使用别名

当查询涉及多个表，而其中同一个表出现了多次时，此时需要为重复的表创建一个别名来避免冲突。下面是关于 `aliased` 的一个例子：

```python
>>> from sqlalchemy.orm import aliased
>>> adalias1 = aliased(Address)
>>> adalias2 = aliased(Address)
>>> for username, email1, email2 in \
...     session.query(User.name, adalias1.email_address, adalias2.email_address).\
...     join(adalias1, User.addresses).\
...     join(adalias2, User.addresses).\
...     filter(adalias1.email_address=='jack@google.com').\
...     filter(adalias2.email_address=='j25@yahoo.com'):
...     print(username, email1, email2)
SELECT users.name AS users_name,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_2.email_address AS addresses_2_email_address
FROM users JOIN addresses AS addresses_1
        ON users.id = addresses_1.user_id
JOIN addresses AS addresses_2
        ON users.id = addresses_2.user_id
WHERE addresses_1.email_address = ?
        AND addresses_2.email_address = ?
('jack@google.com', 'j25@yahoo.com')
jack jack@google.com j25@yahoo.com
```

#### 使用子查询(Subqueries)

`Query` 适合于用来构造子查询。假如想要取出 `User` 记录，并且同时计算各个用户的 `Address` 的数量。产生这种功能的 SQL 指令最好的办法是按照 user 的 id 分组统计地址的数量，然后 join 到外层查询。此时需要 LEFT JOIN，这样可以使得没有地址的用户也会出现在查询结果中（地址数量为0）。 

期望的SQL命令是这样的：

```sql
SELECT users.*, adr_count.address_count FROM users LEFT OUTER JOIN
    (SELECT user_id, count(*) AS address_count
        FROM addresses GROUP BY user_id) AS adr_count
    ON users.id=adr_count.user_id
```

使用 `Query`，可以从内到外来构造上面的语句。

```python
>>> from sqlalchemy.sql import func
>>> stmt = session.query(Address.user_id, func.count('*').\
...         label('address_count')).\
...         group_by(Address.user_id).subquery()
```

`func` 已经在之前就认识过了。`subquery()` 可以产生一个内嵌了 alias（是一个`query.statement.alias()`）的查询( SELECT )语句的表达。

当生成了 statement 之后，其完全可以视为一个 `Table` 来使用。可以通过 `c` 来访问它的属性。

```python
>>> for u, count in session.query(User, stmt.c.address_count).\
...     outerjoin(stmt, User.id==stmt.c.user_id).order_by(User.id):
...     print(u, count)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        anon_1.address_count AS anon_1_address_count
FROM users LEFT OUTER JOIN
    (SELECT addresses.user_id AS user_id, count(?) AS address_count
    FROM addresses GROUP BY addresses.user_id) AS anon_1
    ON users.id = anon_1.user_id
ORDER BY users.id
('*',)
<User(name='ed', fullname='Ed Jones', password='f8s7ccs')> None
<User(name='wendy', fullname='Wendy Williams', password='foobar')> None
<User(name='mary', fullname='Mary Contrary', password='xxg527')> None
<User(name='fred', fullname='Fred Flinstone', password='blah')> None
<User(name='jack', fullname='Jack Bean', password='gjffdd')> 2
```

#### 从子查询中取出Entity

在前一个例子中，从子查询获得的是一个临时性的 JOIN 后的表，但是这个表并未定义 ORM 中定义的 Entity。如果想将这个临时表映射到 ORM 中的类呢？此时可以使用 `aliased` 这个函数来完成这个映射。

```python
>>> stmt = session.query(Address).\
...                 filter(Address.email_address != 'j25@yahoo.com').\
...                 subquery()
>>> adalias = aliased(Address, stmt)
>>> for user, address in session.query(User, adalias).\
...         join(adalias, User.addresses):
...     print(user)
...     print(address)
SELECT users.id AS users_id,
            users.name AS users_name,
            users.fullname AS users_fullname,
            users.password AS users_password,
            anon_1.id AS anon_1_id,
            anon_1.email_address AS anon_1_email_address,
            anon_1.user_id AS anon_1_user_id
FROM users JOIN
    (SELECT addresses.id AS id,
            addresses.email_address AS email_address,
            addresses.user_id AS user_id
    FROM addresses
    WHERE addresses.email_address != ?) AS anon_1
    ON users.id = anon_1.user_id
('j25@yahoo.com',)
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
<Address(email_address='jack@google.com')>
```

### 使用EXISTS

EXISTS 关键字是一个 BOOL 型操作符。当查询结果存在至少一行时返回 True。EXISTS 可以常常和 JOIN 搭配使用。

下面是一个显式的 EXISTS 构造方法：

```python
>>> from sqlalchemy.sql import exists
>>> stmt = exists().where(Address.user_id==User.id)
>>> for name, in session.query(User.name).filter(stmt):
...     print(name)
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT *
FROM addresses
WHERE addresses.user_id = users.id)
()
jack
```

`Query` 还定义了若干个自动使用了 EXISTS 的操作。上面的例子可以用 `any()` 来完成：

```python
>>> for name, in session.query(User.name).\
...         filter(User.addresses.any()):
...     print(name)
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT 1
FROM addresses
WHERE users.id = addresses.user_id)
()
jack
```

`any()`也接受筛选条件来限制匹配的行：

```python
>>> for name, in session.query(User.name).\
...     filter(User.addresses.any(Address.email_address.like('%google%'))):
...     print(name)
jack
```

`has()`对于的many-to-one的关系，起到的是和`any()`同样的作用（注意这里`~`表示NOT）：

```python
>>> session.query(Address).\
...         filter(~Address.user.has(User.name=='jack')).all()
[]
```

### 常用的关系操作

下面只是简单的列出了一些常用的操作。

- **eq() (多对一“等于”):**

  ```python
  query.filter(Address.user == someuser)
  ```

- **ne() (多对一“不等于”):**

  ```python
  query.filter(Address.user != someuser)
  ```

- **IS NULL (多对一比较, 用 eq()):**

  ```python
  query.filter(Address.user == None)
  ```

- **contains() (用于一对多集合):**

  ```python
  query.filter(User.addresses.contains(someaddress))
  ```

- **any() :**

  ```python
  query.filter(User.addresses.any(Address.email_address == 'bar'))
  
  # 也可以接受关键字参数:
  query.filter(User.addresses.any(email_address='bar'))
  ```

- **has() :**

  ```python
  query.filter(Address.user.has(name='ed'))
  ```

- **Query.with_parent() :**

  ```python
  session.query(Address).with_parent(someuser, 'addresses')
  ```

### Eager Loading

前面当查询取出用户时，与之关联的地址并没有取出来。当试图获取 `User.addresses` 时，相关的 SQL 查询才起作用。如果想要减少 query 的次数的话，就需要使用 Eager Loading 了。SQLAlchemy 提供了三种 Eager Loading 的方式，其中两种是自动的，而第三种涉及到自定义的筛选条件。所有的这三种 Eager Loading 方式都会通过调用 `Query.options()` 来影响查询的过程，促使 `Query` 生成需要的额外配置来取出期望的内容。

#### 子查询加载

在上面的例子中，我们希望在取出用户的时候就同步取出对应的地址。此时可以此采用 `orm.subqueryload()`。这个函数可以发起第二个 SELECT 查询来取出与结果相关的另一个表的信息。这里取名为"**子查询加载**"的原因是：此处的 `Query `在发起第二个查询时作为子查询而被复用了。详细过程参加下面的程序：

```python
>>> from sqlalchemy.orm import subqueryload
>>> jack = session.query(User).\
...                 options(subqueryload(User.addresses)).\
...                 filter_by(name='jack').one()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
('jack',)
SELECT addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id,
        anon_1.users_id AS anon_1_users_id
FROM (SELECT users.id AS users_id
    FROM users WHERE users.name = ?) AS anon_1
JOIN addresses ON anon_1.users_id = addresses.user_id
ORDER BY anon_1.users_id, addresses.id
('jack',)
>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

**注意**：当 `subqueryload() `和涉及 limiting 的函数一起使用的时候（如 `Query.first()`, `Query.limit()`, `Query.offset()` 等），应当加上一个以 Unique 的行作为参数的 `Query.order_by()` 来确保结果的正确性。

#### 连接加载

这种方式要更为常用一些。Joined Loading 发起了一个 JOIN（默认是LEFT OUTER JOIN），故而查询结果和制定的与之关联的行可以被同时取出。这里以和上面的Subquery Loading中同样的查询目的为例。

```python
>>> from sqlalchemy.orm import joinedload

>>> jack = session.query(User).\
...                        options(joinedload(User.addresses)).\
...                        filter_by(name='jack').one()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        addresses_1.id AS addresses_1_id,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_1.user_id AS addresses_1_user_id
FROM users
    LEFT OUTER JOIN addresses AS addresses_1 ON users.id = addresses_1.user_id
WHERE users.name = ? ORDER BY addresses_1.id
('jack',)

>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

**注意**，如果是在命令行运行了前一个 Subquery Loading 的例子的话，在这里 jack 的 addresses 实际上已经填充了的，但是这里的 Joined Load 仍然是会发起 JOIN。另外，LEFT OUTER JOIN 指令实际上有可能导致重复的User出现，但是在结果中实际得到的 User 却不会重复。这是因为 `Query` 实际上是基于 Object Identity 采用了一种 "uniquing" 的策略。

`joinedload()` 出现的更早一些。`joinedloading()` 更加适合于处理Many-to-one的关系。

#### 显式的Join + EagerLoad

第三种方式是自己显式的调用 join 来定位 JOIN 连接主键，并接着关联表的信息填充到查询结果中对应对象或者列表中。这个特性需要使用到 `orm.contains_eager()` 函数。这个机制最典型的用途是 pre-loading many-to-one 关系，同时添加对这个关系的筛选。

假设我们需要筛选出用户的名字为jack的邮件地址，进行这个查询的方法如下：

```python
>>> from sqlalchemy.orm import contains_eager
>>> jacks_addresses = session.query(Address).\
...                             join(Address.user).\
...                             filter(User.name=='jack').\
...                             options(contains_eager(Address.user)).\
...                             all()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses JOIN users ON users.id = addresses.user_id
WHERE users.name = ?
('jack',)

>>> jacks_addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]

>>> jacks_addresses[0].user
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

### 关系中的删除问题

尝试删除jack，来看结果：

```python
>>> session.delete(jack)
>>> session.query(User).filter_by(name='jack').count()
UPDATE addresses SET user_id=? WHERE addresses.id = ?
((None, 1), (None, 2))
DELETE FROM users WHERE users.id = ?
(5,)
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?) AS anon_1
('jack',)
0
```

那么与jack关联的地址呢?

```python
>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
...  ).count()
2
```

地址记录仍然在这里。如果 commit 的话，可以从上面的 SQL 语句中发现，相关的 `Address` 的 `user_id` 属性被设置成了NULL。这不符合要求。这就需要自己来设置关系的删除规则。

#### 配置delete/delete-orphan Cascade

通过配置 `User.addresses` 关系的 **cascade** 选项来控制删除行为。尽管 SQLAlchemy 允许你在任何时候给 ORM 添加属性或者关系。此时还是需要移除现存的关系并且重新开始。

首先关闭当前的session

```python
>>> session.close()
```

使用一个新的 `declarative_base()`:

```python
>>> Base = declarative_base()
```

下面重新声明 `User` 类，注意 `addresses` 中的配置：

```python
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    password = Column(String)

    addresses = relationship("Address", back_populates='user',
                        cascade="all, delete, delete-orphan")

    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" % (
                               self.name, self.fullname, self.password)
```

接下来重新声明 `Address`

```python
class Address(Base):
    __tablename__ = 'addresses'
    id = Column(Integer, primary_key=True)
    email_address = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship("User", back_populates="addresses")
    def __repr__(self):
        return "<Address(email_address='%s')>" % self.email_address
```

现在取出 jack(下面使用了一个之前没有提到的函数 `get()`，其参数为查询目标的主键)，现在从 `addresses` 中删除一个地址的话，会导致这个 `Address` 被删除。

```python
>>> jack = session.query(User).get(5)

>>> del jack.addresses[1]

>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
1
```

删除 jack 也会导致剩下 jack 以及其所有的 `Address` 都会被删除:

```python
>> session.delete(jack)

>>> session.query(User).filter_by(name='jack').count()
0

>>> session.query(Address).filter(
...    Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
0
```

### 建立多对多关系ManyToMany Relationship

现在需要引入一个新的模型来阐述多对多的关系了。假设需要完成一个博客应用。在这个应用里面我们可以书写`BlogPost`，每个博客都有若干 `Keyword`。

对于一个多对多的关系，需要建立一个未映射的（也就是没有一个 Python 类与之对应的）表 `Table` 来作为中间联系的表。

```python
>>> from sqlalchemy import Table, Text

>>> post_keywords = Table('post_keywords', Base.metadata,
...     Column('post_id', ForeignKey('posts.id'), primary_key=True),
...     Column('keyword_id', ForeignKey('keywords.id'), primary_key=True)
... )
```

不同于之前的典型的ORM方法，在上面的代码中直接声明了一个 `Table`，而没有制定与之对应的 Python 类。`Table` 是一个构造函数，其参数中的每个 `Colomn` 以逗号分隔。

下面来定义 `BlogPost` 和 `Keyword`。这里需要使用 `relationship()` 在这两个类中定义一对互补的关系，其中每个关系的都指向 `post_keyword` 这个表。

```python
class BlogPost(Base):
    __tablename__ = 'posts'

    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    headline = Column(String(255), nullable=False)
    body = Column(Text)

    # many to many BlogPost<->Keyword
    keywords = relationship('Keyword',
                            secondary=post_keywords,
                            back_populates='posts')

    def __init__(self, headline, body, author):
        self.author = author
        self.headline = headline
        self.body = body

    def __repr__(self):
        return "BlogPost(%r, %r, %r)" % (self.headline, self.body, self.author)


class Keyword(Base):
    __tablename__ = 'keywords'
    id = Column(Integer, primary_key=True)
    keyword = Column(String(50), nullable=False, unique=True)
    posts = relationship('BlogPost',
                         secondary=post_keywords,
                         back_populates='keywords')

    def __init__(self, keyword):
        self.keyword = keyword
```

在上面的定义中，可以发现和 One-To-Many关系不同，`relationship()` 中多了一个 `secondary` 的参数，这个参数指向了中间表。这个中间表只包含了指向多对多关系两侧的表的主键的列。如果这个表包含了其他属性，甚至是自身的主键，SQLAlchemy 需要你使用另一种，称为 `association object` 的机制来处理。

还希望的 `BlogPost` 能够拥有一个 `author` 属性，这个属性指向先前定义的 `User`。此时需要再定义一个双向关系。由于一个作者可能拥有很多文章，访问 `User.posts `的时候可以加以筛选而不是载入全部的相关文章。为此在定义 `User.posts` 的时候，设置 `lazy='dynamic'`，来控制载入策略。

```python
>>> BlogPost.author = relationship(User, back_populates="posts")
>>> User.posts = relationship(BlogPost, back_populates="author", lazy="dynamic")
```

然后创建数据库中对应的表

```python
>>> Base.metadata.create_all(engine)
PRAGMA...
CREATE TABLE keywords (
    id INTEGER NOT NULL,
    keyword VARCHAR(50) NOT NULL,
    PRIMARY KEY (id),
    UNIQUE (keyword)
)
()
COMMIT
CREATE TABLE posts (
    id INTEGER NOT NULL,
    user_id INTEGER,
    headline VARCHAR(255) NOT NULL,
    body TEXT,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES users (id)
)
()
COMMIT
CREATE TABLE post_keywords (
    post_id INTEGER NOT NULL,
    keyword_id INTEGER NOT NULL,
    PRIMARY KEY (post_id, keyword_id),
    FOREIGN KEY(post_id) REFERENCES posts (id),
    FOREIGN KEY(keyword_id) REFERENCES keywords (id)
)
()
COMMIT
```

多对多关系的使用方法道也没有太大的不同之处。先来给windy添加博文。

```python
>>> wendy = session.query(User).\
...                 filter_by(name='wendy').\
...                 one()
>>> post = BlogPost("Wendy's Blog Post", "This is a test", wendy)
>>> session.add(post)
```

给博文添加一些关键字。目前数据库里面还没有关键字存在，先创建一些：

```python
>>> post.keywords.append(Keyword('wendy'))
>>> post.keywords.append(Keyword('firstpost'))
```

此时可以开始查询了。先以 'firstpost' 为关键字来检索所有的博文。使用 `any` 来查询拥有关键词 'firstpost' 的博文：

```python
>>> session.query(BlogPost).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
```

如果希望将查询范围限制在wendy用户所拥有的博文之内，

```python
>>> session.query(BlogPost).\
...             filter(BlogPost.author==wendy).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
SELECT posts.id AS posts_id,
        posts.user_id AS posts_user_id,
        posts.headline AS posts_headline,
        posts.body AS posts_body
FROM posts
WHERE ? = posts.user_id AND (EXISTS (SELECT 1
    FROM post_keywords, keywords
    WHERE posts.id = post_keywords.post_id
        AND keywords.id = post_keywords.keyword_id
        AND keywords.keyword = ?))
(2, 'firstpost')
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
```

或者可以直接在wendy的`posts`属性上进行查询：

```python
>>> wendy.posts.\
...         filter(BlogPost.keywords.any(keyword='firstpost')).\
...         all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
```

