# django连接MySQL操作模型表

## pycharm连接数据库

pycharm也可以充当MySQL的客户端

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212153904.png)



## django连接MySQL

django默认的数据库是sqlite3，我们改为MySQL，需要两步操作

- 在配置文件中改数据库的配置：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 指定数据库
        'NAME': 'test',  # 指定库的名字
        'HOST': '127.0.0.1',   # 要注意键必须大写
        'PORT': 3306,
        'USER': 'root',
        'PASSWORD': '123',
        'CHARSET': 'utf8'
    }
}
```

- 主动告诉django 不要用默认的mysqldb连接 而是用pymysql

```python
# 你可以在项目名下的__init__.py中书写
# 也可以在应用名下的__init__.py中书写
import pymysql
pymysql.install_as_MySQLdb()
```



## django ORM简介

### ORM 对象关系映射

| 类        | 表           |
| --------- | :----------- |
| 对象      | 数据         |
| 对象.属性 | 字段对应的值 |

为什么使用ORM

- 能够让不会数据库操作的人也能够简单方便去操作数据库

缺点：

- 封装程度太高 有时候会出现查询效率偏低的问题



### django中操作ORM

去应用下的models.py中写数据模型类

```python
class User(models.Model):
    # id = models.AutoField(primary_key=True)
    # django当你不指定主键的时候，会自动帮你创建一个名为id字段，并作为主键
    # 如果你自己指定了主键 django就不会再帮你创建

    # 对应的是varchar(32)  django中默认没有char字段，但支持用户自定义
    username = models.CharField(max_length=32)
    password = models.IntegerField()
```

### 数据库迁移命名（同步）

```python
python manage.py makemigrations # 将数据库的修改 记录到小本本上（migrations文件夹内）
python manage.py migrate  # 将修改操作真正的同步到数据库中
```

**注意：**

- **上面两条命令必须是成双成对出现**
- **只要修改了models里面跟数据库相关的代码 你就必须重新执行上面两条命令**

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212153939.png)



## 模型表字段的增删改查

### 字段的修改

直接修改代码 然后执行数据库迁移命令即可

新增字段

- 方式1 设置默认值

```python
email = models.EmailField(default='123@qq.com')
```

- 方式2 允许字段为空

```python
phone = models.BigIntegerField(null=True)
```

- 方式3 直接在提示中给默认值

### 字段的删除

直接注释掉对应的字段，然后执行数据库迁移命令（谨慎使用）

## 模型表数据的增删改查

### 查

```python
data = models.User.objects.filter(username=username)  # <QuerySet [<User: User object>]>
# 相当于 select * from user where username='neo'


"""
filter返回的结果是一个"列表",里面才是真正数据对象
filer括号内可以放多个关键字参数 这多个关键字参数在查询的时候 是and关系
"""


user_list = models.User.objects.all()
"""
结果是一个"列表"，里面是一个个的数据对象
"""
```

### 增

```python
user_obj = models.User.objects.create(username=username,password=password)
print(user_obj,user_obj.username,user_obj.password)
# create方法会有一个返回值  返回值就是当前被创建的对象本身
```

### 改

```python
models.User.objects.filter(id=edit_id).update(username=username,password=password)
"""
批量操作  会将filter查询出来的列表中所有的对象全部更新
"""
```

### 删

```python
models.User.objects.filter(id=delete_id).delete()
"""
批量操作  会将filter查询出来的列表中所有的对象全部删除
"""
```



## orm表关系如何建立

### 一对一

一张表的字段信息太多，可以人为分出一张表

### 一对多

外键字段建在 多的那一方

### 多对多

多对多的外键关系需要建立第三张表来专门处理

以图书馆里系统为例，创建图书表，作者表，出版社表

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212162452.png)



以图书管理系统为例，在django orm 建立表关系：

- 一对一的表关系，外键字段建在任意一方都可以，但是建议建在查询频率较高的一方
- 书与出版社是一对多关系，并且书是多的一方，所以外键字段建在书表中
- 书与作者是多对多的关系， 外键字段建在任意一方都可以，建议建在查询频率较高的一方

```python
class Book(models.Model):
    title = models.CharField(max_length=32)
    # 小数总共八位，小数占两位
    price = models.DecimalField(max_digits=8, decimal_places=2)

    # 书与出版社是一对多关系，并且书是多的一方，所以外键字段建在书表中
    publish = models.ForeignKey(to='Publish')  # to用来指代和哪张表有关系，默认关联的就是主键字段

    # 书与作者是多对多的关系， 外键字段建在任意一方都可以，建议建在查询频率较高的一方
    author = models.ManyToManyField(to='Author')  # django orm会自动帮你创建书和作者的第三张关系表
    # author这个字段是一个虚拟字段 不能在表中展示出来，仅仅只是告诉orm，建立第三张表关系的作用


class Publish(models.Model):
    title = models.CharField(max_length=32)
    email = models.EmailField()


class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    # 一对一的表关系，外键字段建在任意一方都可以，但是建议建在查询频率较高的一方
    author_detail = models.OneToOneField(to='Author_detail')


class Author_detail(models.Model):
    phone = models.BigIntegerField()
    addr = models.CharField(max_length=32)
```

注意点：

```python
'''
一对多外键字段 创建的时候 ，同步到数据库中，表字段会自动加_id后缀；如果自己加了_id，会在后面再加一个_id


publish = models.ForeignKey(to='Publish')默认关联的是主键id，如果主键不是id，要自己关联， 可以加to_field= 与指定字段做关联
'''
```