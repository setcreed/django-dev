# ORM字段及参数

## orm常用字段



| 字段名              | 说明                                                         |
| ------------------- | :----------------------------------------------------------- |
| AutoField           | 如果自己没有定义主键id，django会默认自动创建一个id字段，并把它作为主键 |
| IntegerField        | 一个整数类型,范围在 -2147483648 到 2147483647                |
| BigterField         | bigint                                                       |
| EmailField          | varchar(254)                                                 |
| BooleanField        | 布尔值，该字段在存储的时候 你只需要传布尔值True或False，它会自动存成1/0 |
| TextField           | 专门用来存大段文本                                           |
| FileFiled           | 专门用来存文件    upload_to=文件路径                         |
| DecimalField(Field) | 参数：`max_digits`，小数总长度，    `decimal_places`，小数位长度 |
| CharField           | 对应MySQL中的varchar字段                                     |
| DateField           |                                                              |
| DateTimeFiled       |                                                              |

`DateTimeFiled`和`DateField`都有两个参数：`auto_now_add` 和 `auto_now`

`auto_now` 设为True ，每次更新数据记录的时候会更新该字段

`auto_now_add`设为True， 创建数据记录的时候会把当前时间添加到数据库。



## choices参数

```python
choices = (
(1,'male'),
(2,'female'),
(3,'others')
)
gender = models.IntegerField(choices=choices)



from app01 import models
user_obj = models.Userinfo.objects.filter(pk=4).first()
print(user_obj.username)
print(user_obj.gender)
# 针对choices字段 如果你想要获取数字所对应的中文 你不能直接点字段
# 固定句式   数据对象.get_字段名_display()  当没有对应关系的时候 该句式获取到的还是数字
print(user_obj.get_gender_display())
```



## 字段合集

```python
AutoField(Field)
        - int自增列，必须填入参数 primary_key=True

    BigAutoField(AutoField)
        - bigint自增列，必须填入参数 primary_key=True

        注：当model中如果没有自增列，则自动会创建一个列名为id的列
        from django.db import models

        class UserInfo(models.Model):
            # 自动创建一个列名为id的且为自增的整数列
            username = models.CharField(max_length=32)

        class Group(models.Model):
            # 自定义自增列
            nid = models.AutoField(primary_key=True)
            name = models.CharField(max_length=32)

    SmallIntegerField(IntegerField):
        - 小整数 -32768 ～ 32767

    PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正小整数 0 ～ 32767
    IntegerField(Field)
        - 整数列(有符号的) -2147483648 ～ 2147483647

    PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正整数 0 ～ 2147483647

    BigIntegerField(IntegerField):
        - 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807

    BooleanField(Field)
        - 布尔值类型

    NullBooleanField(Field):
        - 可以为空的布尔值

    CharField(Field)
        - 字符类型
        - 必须提供max_length参数， max_length表示字符长度

    TextField(Field)
        - 文本类型

    EmailField(CharField)：
        - 字符串类型，Django Admin以及ModelForm中提供验证机制

    IPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制

    GenericIPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6
        - 参数：
            protocol，用于指定Ipv4或Ipv6， 'both',"ipv4","ipv6"
            unpack_ipv4， 如果指定为True，则输入::ffff:192.0.2.1时候，可解析为192.0.2.1，开启此功能，需要protocol="both"

    URLField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证 URL

    SlugField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

    CommaSeparatedIntegerField(CharField)
        - 字符串类型，格式必须为逗号分割的数字

    UUIDField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

    FilePathField(Field)
        - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
        - 参数：
                path,                      文件夹路径
                match=None,                正则匹配
                recursive=False,           递归下面的文件夹
                allow_files=True,          允许文件
                allow_folders=False,       允许文件夹

    FileField(Field)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

    ImageField(FileField)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
            width_field=None,   上传图片的高度保存的数据库字段名（字符串）
            height_field=None   上传图片的宽度保存的数据库字段名（字符串）

    DateTimeField(DateField)
        - 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]

    DateField(DateTimeCheckMixin, Field)
        - 日期格式      YYYY-MM-DD

    TimeField(DateTimeCheckMixin, Field)
        - 时间格式      HH:MM[:ss[.uuuuuu]]

    DurationField(Field)
        - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

    FloatField(Field)
        - 浮点型

    DecimalField(Field)
        - 10进制小数
        - 参数：
            max_digits，小数总长度
            decimal_places，小数位长度

    BinaryField(Field)
        - 二进制类型

 字段合集
```



## 自定义char字段

```python
class MyCharField(models.Field):
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        # 重新调用父类的方法
        super().__init__(max_length=max_length, *args, **kwargs)


    def db_type(self, connection):
        return 'char(%s)' % self.max_length
```



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305085917.png)



## 字段参数

| 字段参数     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| null         | 用于表示某个字段可以为空。                                   |
| unique       | 如果设置为unique=True 则该字段在此表中必须是唯一的           |
| db_index     | db_index=True 则代表着为此字段设置索引。                     |
| default      | 为该字段设置默认值                                           |
| auto_now_add | <font color=red>创建数据的时候，会把当前时间自动添加上去，之后就不再修改时间了，除非人为修改</font> |
| auto_now     | <font color=red>每次修改数据的时候，会自动更新当前时间，实时更新</font> |



## 外键字段的参数

| 参数          | 说明                                       |
| ------------- | ------------------------------------------ |
| to            | 设置要关联的表                             |
| to_field      | 设置要关联的表字段，默认django自动关联主键 |
| on_delete     | 当删除关联表中的数据时，当前与其关联的行为 |
| db_constraint | 是否在数据库中创建外键约束，默认是True     |

```
on_delete = modeles.CASCADE   # 级联删除
```

注意：在django 2.x版本   需要手动指定  `db_constraint`等参数