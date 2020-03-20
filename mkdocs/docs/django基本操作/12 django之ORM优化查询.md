# django之ORM优化查询

## only与defer

```python
res = models.Book.objects.all()
```

这样是不会有任何返回结果，因为ORM是惰性查询，减少不必要的数据库操作，降低数据库的压力。

也就是说**能少走一次数据库就少走一次**，最好是一次数据库都不要走或者说之走一次。

### only优化：

```python
res = models.Book.objects.only('title')     # 括号内查询的字段可以有多个
print(res)        # 查询一次，打印一条sql查询语句
for i in res:
    print(i.title)  # 查询一次，打印一条sql查询语句
    print(i.price)  # 有几个对象，就查询几次，打印几条sql查询语句
```

only会把括号内字段对应的值，封装到查询返回的对象中，通过对象点括号字段，**不需要再走数据库查询，直接返回结果**，**一旦你点了不是括号内的字段 就会频繁的去走数据库查询**

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305091647.png)

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305091857.png)

### defer优化

```python
res = models.Book.objects.defer('title')
# print(res)
for i in res:
    # print(i.title)
    print(i.title)
```

**和 only相反，defer会将括号内的字段排除之外将其他字段对应的值， 直接封装到返回给你的对象中， 点其他字段 不需要再走数据库查询，一旦你点了括号内的字段就会有多少值，就会查询几次**



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305091957.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305092116.png)

## select_related和prefetch_related

### select_related优化

```python
res = models.Book.objects.select_related('publish')
# print(res)
for i in res:
    print(i.publish)
```

- select_related括号内放外键字段，并且外键字段的类型只能是一对一和一对多，不能是多对多，

- 内部自动做联表操作，会将括号内外键字段所关联的表与当前表自动拼接成一张表，然后将表中的数据一个个查询出来封装成一个个的对象。 这样做 就不会重复的走数据库，减轻数据库的压力。

- select_related括号内可以放多个外键字段，用逗号给开，会将多个外键字段关联的表拼接成一张大表

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305092408.png)

### prefetch_related优化

```python
res = models.Book.objects.prefetch_related('publish','authors')
for i in res:
    print(i.publish)
```

- prefetch_related内部是子查询，会自动按照步骤查询多张表，然后将查询的结果封装到对象中，这样给用户的感觉还是联表操作。
- 括号内支持传多个外键字段，并且没有类型限制。
- 每放一个外键字段，就会多走一条sql语句，多查询一张表

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305092925.png)


