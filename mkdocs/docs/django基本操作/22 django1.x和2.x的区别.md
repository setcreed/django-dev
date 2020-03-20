# django1.x和2.x的区别

## 路由的区别

### django1.x中的url

```python
from django.conf.urls import url

# 使用url关键字
urlpatterns = [
    url('article-(\d+).html',views.article),
    url('article-(?P<article_id>\d+).html',views.article)
]
# url请求地址为：http://127.0.0.1:8000/article-1.html
```

Django1的url支持正则匹配：

- `article-(\d+).html`：使用正则表达式的分组匹配来获取URL中的参数，并以位置参数形式传递给视图article。
- `article-(?P<article_id>\d+).html`：使用正则表达式的分组命名匹配来获取URL中的参数，并以关键字参数的形式传递给视图article。

分组命名正则表达式组的语法是：`(?P<name>pattern)`，其中name是组的名称(视图中的关键字参数必须跟组名一致)，pattern是正则表达式。

### django2.x中的url

#### django2中特有的url

url规则:

- path写的是绝对字符串,请求地址必须与路由地址完全匹配
- 使用尖括号 <> 从url中获取参数值
- 可以使用转换器指定参数类型，例如： <int:age> 捕获一个整数参数age, 若果没有转化器，将匹配任何字符串，也包括路径分隔符 /
- path拥有5个转换器:
  - str:匹配除路径分隔符 / 外的字符串
  - int:匹配自然数
  - slug:匹配字母,数字,横杠及下划线组成的字符串
  - uuid:匹配uuid形式的数据
  - path:匹配任何字符串,包括路径分隔符 /

```python
from django.urls import path

# 使用path关键字
urlpatterns = [
    path('article-<int:article_id>.html',views.article),
]
# url请求地址为：http://127.0.0.1:8000/article-1.html
```



自定义转换器：

- 自定义一个类
- 类中必须有：类属性regex，to_python方法，to_url方法
- regex：类属性，字符串类型
- to_python(self, value)方法：value是由类属性 regex 所匹配到的字符串，返回具体的Python变量值，以供Django传递到对应的视图函数中。
- to_url(self, value)方法：和 to_python 相反，value是一个具体的Python变量值，返回其字符串，通常用于url反向引用。

例子：

```python
class Date:
    regex = '^0?[1-9]$|^1[0-2]$'
    
    def to_python(self, value):
        # 可以写你的逻辑，对匹配到的字符串进行处理
        value = '2019/' + value
        return value
        
    def to_url(self, value):
        return '%2s' % value

# 在主路由下导入,生成转换器
from django.urls import register_converter

register_converter(Date,'date')

path('full-year/<date:full_year>/',views.full_year, name="full_year")
```



#### django2.x的url兼容了django1.x的写法

```python
from django.urls import re_path
#　这里的re_path的用法跟Django1的url用法视完全一样的，匹配正则。

urlpatterns = [
    path('articles/2003/', views.special_case_2003),  # Django2的写法
    re_path('articles/(?P<year>[0-9]{4})/', views.year_archive),  # 兼容Django1的写法
    re_path('articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/', views.month_archive),
    re_path('articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<slug>[^/]+)/', views.article_detail),
]
```



参考：<https://www.cnblogs.com/Zzbj/p/11150041.html>