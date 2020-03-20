# drf序列化组件实现十大接口

## 序列化字段了解配置

了解配置：

``` 
fields = '__all__'
exclude = ['name']   排除name字段

depth = 2   自动深度，值代表深度次数，但是被深度的外键采用__all__，显示所有字段

```



## response二次封装

```python
from rest_framework.response import Response
class APIResponse(Response):
    def __init__(self, status=0, msg='ok', results=None, http_status=None,
                 headers=None, exception=False, content_type=None, **kwargs):
        # 将status、msg、results、kwargs格式化成data
        data = {
            'status': status,
            'msg': msg,
        }
        # results只要不为空都是数据：False、0、'' 都是数据 => 条件不能写if results
        if results is not None:
            data['results'] = results
        # 将kwargs中额外的k-v数据添加到data中
        data.update(**kwargs)

        super().__init__(data=data, status=http_status, headers=headers, exception=exception, content_type=content_type)
```



## 连表深度查询

外键字段默认显示的是外键值（int类型），不会自己进行深度查询

深度查询方式：

- 子序列化：必须有子序列化类配合，不能反序列化
- 配置depth：自动深度查询的是关联表测所有字段，数据量太多
- 插拔式@property：名字不能与外键名同名

```python
from django.db import models

from django.contrib.auth.models import User
class BaseModel(models.Model):
    is_delete = models.BooleanField(default=False)
    created_time = models.DateTimeField(auto_now_add=True)

    class Meta:
        # 基表，为抽象表，是专门用来被继承，提供公有字段的，自身不会完成数据库迁移
        abstract = True

class Book(BaseModel):
    name = models.CharField(max_length=64)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    publish = models.ForeignKey(to='Publish', related_name='books', db_constraint=False, on_delete=models.DO_NOTHING, null=True)
    authors = models.ManyToManyField(to='Author', related_name='books', db_constraint=False)


    @property
    def publish_info(self):  # 单个数据
     
        return {
            'name': self.publish.name,
            'address': self.publish.address,
        }

    @property
    def author_list(self):
        author_list_temp = []  # 存放所有作者格式化成数据的列表
        authors = self.authors.all()  # 所有作者
        for author in authors:  # 遍历处理所有作者
            author_dic = {
                'name': author.name,
            }
            try:  # 有详情才处理详情信息
                author_dic['mobile'] = author.detail.mobile
            except:
                author_dic['mobile'] = '无'

            author_list_temp.append(author_dic)  # 将处理过的数据添加到数据列表中

        return author_list_temp  # 返回处理后的结果

    def __str__(self):
        return self.name

class Publish(BaseModel):
    name = models.CharField(max_length=64)
    address = models.CharField(max_length=64)

class Author(BaseModel):
    name = models.CharField(max_length=64)

class AuthorDetail(BaseModel):
    mobile = models.CharField(max_length=64)
    author = models.OneToOneField(to=Author, related_name='detail', db_constraint=False, on_delete=models.CASCADE)

```



## 单查群查

```python
class BookAPIView(APIView):
    # 单查群查
    def get(self, request, *args, **kwargs):
        pk = kwargs.get('pk')
        if pk:
            book_obj = models.Book.objects.filter(is_delete=False, pk=pk).first()
            book_ser = serializers.BookModelSerializer(book_obj)
        else:
            book_query = models.Book.objects.filter(is_delete=False).all()
            book_ser = serializers.BookModelSerializer(book_query, many=True)
        return APIResponse(results=book_ser.data)
```



## 单增群增

```python
 class BookAPIView(APIView):
    def post(self, request, *args, **kwargs):
        """
        单增：接口：/books/   数据：{...}
        群增：接口：/books/   数据：[{...}, ..., {...}]
        逻辑：将数据给系列化类处理，数据的类型关系到 many 属性是否为True
        """
        if isinstance(request.data, dict):
            many = False
        elif isinstance(request.data, list):
            many = True
        else:
            return Response(data={'detail': '数据有误'}, status=400)

        book_ser = serializers.BookModelSerializer(data=request.data, many=many)
        book_ser.is_valid(raise_exception=True)
        book_obj_or_list = book_ser.save()
        return APIResponse(results=serializers.BookModelSerializer(book_obj_or_list, many=many).data)
```

注意：ModelSerializer只能完成单增，需要借助ListSerializer才能完成群增。

```python
from rest_framework import serializers
from . import models
# 多表操作
class BookListSerializer(serializers.ListSerializer):
    # 自定义的群增群改辅助类，没有必要重写create方法
    def create(self, validated_data):
        return super().create(validated_data)

    def update(self, instance_list, validated_data_list):
                    return [
                        self.child.update(instance_list[index], attrs) for index, attrs in enumerate(validated_data_list)
                    ]


class BookModelSerializer(serializers.ModelSerializer):
    class Meta:
        # ModelSerializer默认配置了ListSerializer辅助类，帮助完成群增群改
        # list_serializer_class = serializers.ListSerializer
        # 如果只有群增，是不需要自定义配置的，但要完成群改，必须自定义配置
        list_serializer_class = BookListSerializer
        model = models.Book
        fields = ['name', 'price', 'publish', 'authors', 'publish_info', 'author_list']
        extra_kwargs = {
            'publish': {
                'write_only': True
            },
            'authors': {
                'write_only': True
            }
        }
```







## 单删群删

```python
class BookAPIView(APIView):
    def delete(self, request, *args, **kwargs):
            """
            单删：接口：/books/(pk)/   数据：空
            群删：接口：/books/   数据：[pk1, ..., pkn]
            逻辑：修改is_delete字段，修改成功代表删除成功，修改失败代表删除失败
            """
            pk = kwargs.get('pk')
            if pk:
                pks = [pk]  # 将单删格式化成群删一条
            else:
                pks = request.data  # 群删
            try:  # 数据如果有误，数据库执行会出错
               	rows = models.Book.objects.filter(is_delete=False, pk__in=pks).update(is_delete=True)
            except:
                return APIResponse(1, '数据有误')

            if rows:
                return APIResponse(0, '删除成功')
            return APIResponse(1, '删除失败')
```



## 整体单改群改

```python
class BookAPIView(APIView):    
    def put(self, request, *args, **kwargs):
        """
        单改：接口：/books/(pk)/   数据：{...}
        群改：接口：/books/   数据：[{pk, ...}, ..., {pk, ...}]
        逻辑：将数据给系列化类处理，数据的类型关系到 many 属性是否为True
        """
        pk = kwargs.get('pk')
        if pk:  # 单改
            try:
                # 与增的区别在于，需要明确被修改的对象，交给序列化类
                book_instance = models.Book.objects.get(is_delete=False, pk=pk)
            except:
                return Response({'detail': 'pk error'}, status=400)

            book_ser = serializers.BookModelSerializer(instance=book_instance, data=request.data)
            book_ser.is_valid(raise_exception=True)
            book_obj = book_ser.save()
            return APIResponse(results=serializers.BookModelSerializer(book_obj).data)
        else:  # 群改
            # 分析（重点）：
            # 1）数据是列表套字典，每个字典必须带pk，就是指定要修改的对象，如果有一条没带pk，整个数据有误
            # 2）如果pk对应的对象已被删除，或是对应的对象不存在，可以认为整个数据有误(建议)，可以认为将这些错误数据抛出即可
            request_data = request.data
            try:
                pks = []
                for dic in request_data:
                    pk = dic.pop('pk')  # 解决分析1，没有pk pop方法就会抛异常
                    pks.append(pk)

                book_query = models.Book.objects.filter(is_delete=False, pk__in=pks).all()
                if len(pks) != len(book_query):
                    raise Exception('pk对应的数据不存在')
            except Exception as e:
                return Response({'detail': '%s' % e}, status=400)

            book_ser = serializers.BookModelSerializer(instance=book_query, data=request_data, many=True)
            book_ser.is_valid(raise_exception=True)
            book_list = book_ser.save()
            return APIResponse(results=serializers.BookModelSerializer(book_list, many=True).data)
```



## 局部单改群改

```python
# 局部单改群改
    def patch(self, request, *args, **kwargs):
        pk = kwargs.get('pk')
        if pk:  # 单改
            try:
                book_instance = models.Book.objects.get(is_delete=False, pk=pk)
            except:
                return Response({'detail': 'pk error'}, status=400)
            # 设置partial=True的序列化类，参与反序列化的字段，都会置为选填字段
            # 1）提供了值的字段发生修改。
            # 2）没有提供的字段采用被修改对象原来的值

            # 设置context的值，目的：在序列化完成自定义校验(局部与全局钩子)时，可能需要视图类中的变量，如请求对象request
            # 可以通过context将其传入，在序列化校验方法中，self.context就能拿到传入的视图类中的变量
            book_ser = serializers.BookModelSerializer(instance=book_instance, data=request.data, partial=True, context={'request': request})
            book_ser.is_valid(raise_exception=True)
            book_obj = book_ser.save()
            return APIResponse(results=serializers.BookModelSerializer(book_obj).data)
        else:  # 群改
            request_data = request.data
            try:
                pks = []
                for dic in request_data:
                    pk = dic.pop('pk')
                    pks.append(pk)

                book_query = models.Book.objects.filter(is_delete=False, pk__in=pks).all()
                if len(pks) != len(book_query):
                    raise Exception('pk对应的数据不存在')
            except Exception as e:
                return Response({'detail': '%s' % e}, status=400)

            book_ser = serializers.BookModelSerializer(instance=book_query, data=request_data, many=True, partial=True)
            book_ser.is_valid(raise_exception=True)
            book_list = book_ser.save()
            return APIResponse(results=serializers.BookModelSerializer(book_list, many=True).data)

```

设置context的值，目的：在序列化完成自定义校验(局部与全局钩子)时，可能需要视图类中的变量，这是就可以通过context将变量传入，  如把request传入

序列化类

```python
class BookModelSerializer(serializers.ModelSerializer):


    class Meta:
        list_serializer_class = BookListerSerializer
        model = models.Book
        fields = ['name', 'price', 'publish', 'authors', 'publish_info', 'author_list']
        extra_kwargs = {
            'publish': {
                'write_only': True
            },.
            'authors': {
                'write_only': True
            }
        }

    # 验证视图类是否将request请求参数通过context传入
    def validate(self, attrs):
        print("传入的request参数：%s" % self.context.get('request'))
        return attrs
```