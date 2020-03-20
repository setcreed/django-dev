# django之模型层

## 配置测试文件

第一种方法：

- 直接在某一个应用下的test文件中书写（前四行代码去manage.py中拷贝）：

```python
import os
  
if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
  
    import django
    django.setup()
```

第二种方法：

直接新建一个任意名称的py文件  在里面写上上面的配置

## ORM单表操作

### 先前操作

```python
# 模型类
class Books(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    publish_date = models.DateField()
```



#### 创建数据

```python
# 方式1：create方法
models.Books.objects.create(title='三国演义', price=345.66, publish_date='2019-11-27')

# 方式2：利用对象的绑定方法
book_obj = models.Books(title='红楼梦', price=235.66, publish_date='2018-09-12')
book_obj.save()
```

#### 修改数据

```python
# 方式1：利用queryset方法
res = models.Books.objects.filter(pk=1).update(price=320.23)
print(res)    # <QuerySet [<Books: Books object>]>


# 方式2：利用对象 (不推荐使用)   利用对象修改 内部其实是从头到尾将数据的所有字段都重新写一遍
book_obj = models.Books.objects.get(pk=2)
book_obj.price = 150.33
book_obj.save()
```

注：

- `pk`会自动查找当前表的主键字段
- filter查询出来的结果是一个`QuerySet`对象

#### QuerySet对象特点

- 只要是QuerySet对象就可以无限制的调用QuerySet的方法

  ```python
  book_obj = models.Books.objects.filter(pk=1).filter().filter()
  # 可以无限制 的删选
  ```

  

- 只要是QuerySet对象就可以用句点符  点query查看当前内部对应的sql语句

  ```python
  book_obj = models.Books.objects.filter(pk=1)
  print(book_obj.query)
  ```

#### get和filter区别

- filter获取到的是一个queryset对象，类似于一个列表
- get获取到的直接就是数据对象本身
- 当条件不存在时，filter不报错直接返回一个空，get直接报错

#### 删除数据

```python
# 方式1：
models.Books.objects.filter(pk=7).delete()

# 方式2：
book_obj = models.Books.objects.get(pk=6)
book_obj.delete()
```



### 必知必会13条操作

如果想要在终端查看 orm对应的sql语句，可以在settings.py中加配置

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```



#### 1. all()   查询所有

返回的是queryset对象

```python
res = models.Books.objects.all()
print(res)
```

注意：orm语句的查询默认是惰性查询，只有当你真正使用数据的时候才会执行orm语句

#### 2. filter()   筛选

相当于原生SQL语句的where

```python
res = models.Books.objects.filter(pk=3, title='人')  # 可以传多个参数，and关系
print(res)
```

#### 3. get()  

筛选，获取的是数据对象本身，条件不存在时报错, 而且查询条件只能是一个

```python
res = models.Books.object.get(pk=2)
print(res)
```

#### 4. first()

取queryset中第一个数据对象

```python
res = models.Books.objects.filter(title='西游记').first()
print(res.price)
```

#### 5. last()

取queryset中最后一个数据对象

```python
res = models.Books.objects.filter(title='西游记').last()
print(res)
```

#### 6. count()

统计数据的个数

```python
num = models.Books.objects.count()
print(num)
```

#### 7. values()   

获取对象中指定字段的值，可以传多个参数，返回的是  queryset对象 列表套字典

```python
res = models.Books.objects.values('title','price')
print(res)
```

#### 8. values_list()

获取对象中指定字段的值，可以传多个参数，返回的是  queryset对象 列表套元组

```python
res = models.Books.objects.values_list('title','price')
print(res)
```

#### 9. order_by()

按照指定的字段排序， 默认是升序

降序只要在字段前面加负号（-）

```python
res = models.Books.objects.order_by('price')
res = models.Books.objects.all().order_by('price')   # 两者等价，默认是升序

res = models.Books.objects.order_by('-rice')
```

#### 10. reverse()

颠倒顺序的前提是    颠倒的对象必须要有顺序（提前排序之后才能颠倒顺序）

所以必须和order_by联用

```python
res = models.Books.objects.order_by('price')
res1 = models.Books.objects.all().order_by('price').reverse()
print(res1)
```

#### 11. exclude()

排除

```python
res = models.Books.objects.all().exclude(title='人间失格')
print(res)
```

#### 12. exists()

判断查询结果是否有值，返回结果是一个布尔值

```python
res = models.Books.objects.filter(pk=12).exists()
print(res)
```

#### 13. distinct()

对查询结果去重，但是必须要有完全相同的数据，才能去重

**注意：主键id不一样，会忽略**

```python
res = models.Books.objects.values('title','price').distinct()
print(res)
```

### 双下划线查询

- `__gt`  大于
- `__lt`  小于
- `__gte`  大于等于
- `__lte`  小于等于
- `__range`  范围查询   顾头顾尾
- `__in`   在里面
- `__year` 
- `__month`

```python
# 查询价格大于200的书籍
res = models.Books.objects.filter(price__gt=200)
print(res)

# 查询价格小于300 的书籍
res = models.Books.objects.filter(price__lt=200)
print(res)

# 查询价格大于等于200
res = models.Books.objects.filter(price__gte=200)
print(res)

# 查询价格小于等于200
res = models.Books.objects.filter(price__lte=200)
print(res)

# 查询价格在200~300之间的书籍
res = models.Books.objects.filter(price__range=(200, 300))  # 顾头顾尾
print(res)

# 查询价格是200 或 300 的书籍
res = models.Books.objects.filter(price__in=[200, 300])
print(res)

# 查询出版日期为2019年的书籍
res = models.Books.objects.filter(publish_date__year=2019)
print(res)


# 查询出版日期是11月份的书籍
res = models.Books.objects.filter(publish_date__month=11)
print(res)
```



- `__startswitch`  以....开头
- `__endswith`   以...结尾
- `__contains`   包含  如果查询字母，默认区分大小写
- `__icontains`    加 `i` 忽略大小写

```python
# 查询书籍是以“人”开头的书
res = models.Books.objects.filter(title__startswith='人')
print(res)

# 查询书籍是以“梦”结尾的书
res = models.Books.objects.filter(title__endswith='梦')
print(res)

# 查询书籍名称中包含“的”字的书籍
res = models.Books.objects.filter(title__contains='的')
print(res)

# 查询书籍名称中包含字母p的书籍
res = models.Books.objects.filter(title__contains='p')   # 默认区分大小写
res = models.Books.objects.filter(title__icontains='p')  # 加i不区分大小
```



## 一对多外键字段增删改查

### 准备工作

先建立图书表、出版社表、作者表、作者详情表

```python
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    publish_date = models.DateField(auto_now_add=True)

    publish = models.ForeignKey(to='Publish')
    authors = models.ManyToManyField(to='Author')


class Publish(models.Model):
    name = models.CharField(max_length=32)
    addr = models.CharField(max_length=64)


class Author(models.Model):
    name = models.CharField(max_length=32)
    email = models.EmailField()

    author_detail = models.OneToOneField(to='AuthorDetail')


class AuthorDetail(models.Model):
    phone = models.BigIntegerField()
    addr = models.CharField(max_length=64)

```

### 花式操作

增：

```python
# 方法1 ：直接传表里面的实际字段
models.Book.objects.create(title='人间失格', price=123.56, publish_id=1)

# 方法2 ：传虚拟字段  跟数据对象即可
publish_obj = models.Publish.objects.filter(pk=2).first()
models.Book.objects.create(title='水浒传', price=223.45, publish=publish_obj)
```

改：

```python
# 方法1： 直接传实际字段
models.Book.objects.filter(pk=1).update(publish_id=2)

# 方法2：传数据对象
publish_obj = models.Publish.objects.filter(pk=1).first()
models.Book.objects.filter(pk=1).update(publish=publish_obj)
```

删：

```python
# 默认就是级联删除 级联更新
models.Publish.objects.filter(pk=1).delete()
```

## 多对多外键字段的增删改查

其实就是操作第三张关系表

### 增：add

```python
book_obj = models.Book.objects.filter(pk=1).first()
book_obj.authors.add(1)   # book_obj.authors 就已经进入第三张表了

book_obj.authors.add(1, 2)  # 添加两个书籍作者



# 同样支持传数据对象
book_obj = models.Book.objects.filter(pk=2).first()
author_obj = models.Author.objects.filter(pk=2).first()
book_obj.authors.add(author_obj)

# 数据对象也支持传多个
book_obj = models.Book.objects.filter(pk=2).first()
author_obj1 = models.Author.objects.filter(pk=1).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
book_obj.authors.add(author_obj1, author_obj2)
```

add方法  能够向第三张关系表添加数据

- 支持传数字
- 也支持传对象，并且两者都可以是多个



### 改：set

```python
# 方法1：传字段
book_obj = models.Book.objects.filter(pk=2).first()
book_obj.authors.set([1, 3])
book_obj.authors.set([1,])   # set里面要传可迭代对象

# 方法2：传数据对象
author_obj1 = models.Author.objects.filter(pk=1).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
book_obj.authors.set((author_obj1,author_obj2))
```

set方法  能修改多对多关系表中的数据

- 可以传数字，也可以传对象，都支持传多个
- **set里面传的是可迭代对象**

### 删：remove

```python
# 方法1：传字段
book_obj = models.Book.objects.filter(pk=2).first()
book_obj.authors.remove(1)
book_obj.authors.remove(1, 2)   # 支持多个


# 方法2：传数据对象
book_obj = models.Book.objects.filter(pk=1).first()
author_obj = models.Author.objects.filter(pk=1).first()
book_obj.authors.remove(author_obj)
```

remove方法   能删除多对多关系表中的数据

- 可以传数字，也可以传对象，
- 都支持传多个， 不需要传迭代对象

### 清空：clear

```python
book_obj = models.Book.objects.filter(pk=2).first()
book_obj.authors.clear()
```

clear清空数据，括号里面不需要传参数

## 跨表查询

### 正反向查询

- 正向查询    关系字段在谁那，由有关系字段的表查，就是正向查询
- 反向查询    关系字段不在那    

正向查询按字段，反向查询按表名小写 + `_set`

### 基于对象的跨表查询（子查询）

```python
# 1.查询书籍主键为2的出版社名称
book_obj = models.Book.objects.filter(pk=2).first()
print(book_obj.publish.name)

# 2.查询书籍主键为1的作者姓名
book_obj = models.Book.objects.filter(pk=1).first()
# print(book_obj.authors)    # app01.Author.None
print(book_obj.authors.all())  # 当正向查询点击外键字段数据有多个的情况下 需要.all()

# 3.查询作者是式微的手机号码
author_obj = models.Author.objects.filter(name='式微').first()
print(author_obj.author_detail.phone)


# 4.查询出版社是东方出版社出版过的书籍
publish_obj = models.Publish.objects.filter(name='东方出版社').first()
print(publish_obj.book_set.all())

# 5.查询手机号是120的作者姓名
author_detail_obj = models.AuthorDetail.objects.filter(phone=120).first()
print(author_detail_obj.author.name)

# 6.查询作者是无名写过的书籍
author_obj = models.Author.objects.filter(name='无名').first()
print(author_obj.book_set.all())
```

总结：

- 正向查询，`点击外键字段数据有多个的情况下 需要.all()`
- 反向查询，一对多和多对多需要`表名 + _set`，一对一不需要加`_set`



### 基于双下划线的跨表查询（联表查询）

```python
# 1.查询书籍pk为2的出版社名称
res = models.Book.objects.filter(pk=2).values('publish__name')
print(res)
res = models.Publish.objects.filter(book__pk=2).values('name')

# 2.查询书籍pk为2的作者姓名和邮箱
res = models.Book.objects.filter(pk=2).values('authors__name', 'authors__email')
print(res)
res = models.Author.objects.filter(book__pk=2).values('name', 'email')

# 3.查询作者是邶风的家庭地址
res = models.Author.objects.filter(name='邶风').values('author_detail__addr')
print(res)
res = models.AuthorDetail.objects.filter(author__name='邶风').values('addr')


# 4.查询出版社是东方出版社出版过的书的名字
res = models.Publish.objects.filter(name='东方出版社').values('book__title')
print(res)
res = models.Book.objects.filter(publish__name='东方出版社').values('title')


# 5. 查询书籍pk是2的作者的手机号
res = models.Book.objects.filter(pk=2).values('authors__author_detail__phone')
print(res)
res = models.Author.objects.filter(book__pk=2).values('author_detail__phone')
```