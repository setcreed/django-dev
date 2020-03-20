# django之路由层

## django请求生命周期流程图

wsgi—中间件—路由----视图—中间件—wsgi

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200303080915.png)



## url.py路由层

```python
urlpatterns = [
    url(r'^admin/', admin.site.urls), 
    url(r'test/', views.test), 
    url(r'testadd/', views.testadd)
]
```

如果url.py这样写，`test`和`testadd`后缀的访问路径，返回的内容是一样的，原因如下：

- url第一个参数是一个正则表达式
- 一旦正立刻结束则表达式能够匹配到内容，会立刻结束匹配关系，直接执行后面对应的函数

## 路由匹配

启动django，在浏览器输入`127.0.0.1:8000/test`，django会自动加斜杠。

### django匹配路由规律

不加斜杠(`127.0.0.1:8000/test`)，先匹配一次试试，如果匹配不上，会让浏览器重定向，加一个斜杠(`127.0.0.1:8000/test/`)再来一次匹配，如果还匹配不上，才会报错。

### 取消django自动让浏览器加斜杠的功能

在配置文件中settings.py中添加:

```python
APPEND_SLASH = False   # 该参数默认为True
```

### 限制指定输入的url

```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),   # url第一个参数是一个正则表达式
    url(r'^test/$', views.test),  # 一旦正立刻结束则表达式能够匹配到内容，会立刻结束匹配关系，直接执行后面对应的函数
    url(r'^testadd/$', views.testadd)
]
```

这样设置，只能输入`127.0.0.1:8000/test/`或`127.0.0.1:8000/testadd/`

**注意：路由匹配只是匹配URL部分，不匹配get携带的参数 ?后面的参数**

## 无名分组

正则表达式的无名分组

```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),   
    url(r'^test/([0-9]{4})', views.test),   # 表示test后面跟4个数字
    url(r'^testadd/', views.testadd)
]
```

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212162737.png)



当你的路由中有分组的正则表达式，那么在匹配到内容执行视图函数的时候，会将分组内正则表达式匹配到的内容当作**位置参数**传递给视图函数。

```python
# 在视图函数
def test(request, xxx):
    print('多余的参数:', xxx)
    return HttpResponse('test view')
```

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212162830.png)

当你的路由中有分组并且给分组起了别名，那么在匹配内容的时候，会将分组内的正则表达式匹配到的内容当作**关键字参数**传递给视图函数

```python
# 在视图函数
def test(request, year):
    print('多余的参数:', year)
    return HttpResponse('test view')
```

这样就可以利用有名和无名分组，我们就可以在调用视图函数之前给函数传递额外的参数

**注意：有名分组和无名分组不能混合使用，但是同一情况下，无名分组可以使用多次，有名分组也可以使用多次**

## 反向解析

举个例子：

```python
# urls.py
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^test/(\d+)/', views.test),
    url(r'^testadd/(?P<year>\d+)/', views.testadd),
    url(r'^index/', views.index),
    url(r'^home/', views.home),
]


# views.py
def index(request):
    return render(request, 'index.html')


def home(request):
    return HttpResponse('home')
```

在上述代码中，`index.html`页面中有很多跳转的链接，都指向`home`路由。如果像改变`home`的url地址，那么`index.html`页面中的很多跳转`home`的链接都有改变，有没有动态绑定url地址的方法呢？反向解析就是。

### 定义

反向解析：根据一个别名，动态解析出一个结果，该结果可以直接访问对应的url

### 路由中没有正则表达式，直接就是写死的

```python
url(r'^home/', views.home,name='xxx'),   # 给路由与视图函数对应关系起别名
```

#### 前端反向解析

```html
<p><a href="{% url 'xxx'%}">111</a></p>
```

#### 后端反向解析

```python
from django.shortcuts import render,HttpResponse,redirect,reverse

def get_url(request):
    url = reverse('xxx')
    print(url)
    return HttpResponse('get_url')
```

### 无名分组的反向解析

在解析的时候，你需要手动指定正则匹配内容的是什么

```python
url(r'^home/(\d+)/', views.home, name='xxx'),
```

#### 前端反向解析

```html
<p><a href="{% url 'xxx' 12 %}">111</a></p>
```

#### 后端反向解析

```python
def get_url(request):
    url = reverse('xxx', args=(1,))
    url2 = reverse('xxx', args=(1231,))
    print(url)
    print(url2)
    return HttpResponse('get_url')
```

手动传入的参数 只需要满足 能够被正则表达式匹配即可

### 有名分组的反向解析

```
url(r'^home/(?P<year>\d+)/', views.home, name='xxx'),
```

#### 前端反向解析

可以直接用无名分组的情况

```html
<p><a href="{% url 'xxx' 12 %}">111</a></p>
```

规范的写法：

```html
<p><a href="{% url 'xxx' year=121 %}">111</a></p>
```

#### 后端反向解析

可以直接用无名分组的情况

也可以规范写：

```python
def get_url(request):
    url = reverse('xxx', kwargs={'year': 13123})
    print(url)
    return HttpResponse('get_url')
```



### 以编辑功能为例，反向解析的应用

```python
# urls.py
url = (r'^edit_user/(\d+)/', views.edit_user, name='edit')

# views.py
def edit_user(request, edit_id):  # edit_id就是用户想要编辑数据主键值
    pass
```



```html
<!--页面-->
{% for user_obj in user_list %}
<a href='/edit_user/{{user_obj.id}}/'>编辑</a>
<a href='{% url 'edit' user_obj.id %}'>编辑</a>
{% endfor %}
```



## 路由分发

前提：

```python
'''
在django中所有的app都可以有自己独立的urls.py \ templates \ static.
正是由于上面的特点 你用django开发项目就能够完全做到多人分组开发 互相不干扰,每个人只开发自己的app.
小组长只需要将所有人开发的app整合到一个空的django项目里面,
然后在settings配置文件注册 再利用路由分发将多个app整合到一起即可完成大项目的拼接
'''
```

路由分发解决的就是 项目的总路由 匹配关系过多的情况。使用路由分发， 会出现：

- 总路由不再做匹配的活 而仅仅是做任务分发
- 请求来了之后 总路由不做对应关系，只询问你要访问哪个app的功能 然后将请求转发给对应的app去处理

### 总路由 (include)

只需要将所有的app的urls.py导入即可

```python
from django.conf.urls import url, include
from app01 import urls as app01_urls
from app02 import urls as app02_urls


urlpatterns = [
    url(r'^app01/', include(app01_urls)),
    url(r'^app02/', include(app02_urls)),
]
# 路由分发
```

### 子路由

```python
# app01 urls.py
from django.conf.urls import url
from app01 import views


urlpatterns = [
    url(r'^reg/', views.reg),

]


# app02 urls.py
from django.conf.urls import url
from app02 import views


urlpatterns = [
    url(r'^reg/', views.reg),

]
```

最省事的写法：

```python
# 连导入都不需要
url(r'^app01/',include('app01.urls')),
url(r'^app02/',include('app02.urls'))
```



## 名称空间 (namespace)

当多个app中出现了起别名冲突的情况 你在做路由分发的时候 可以给每一个app创建一个名称空间。
然后在反向解析的时候 可以选择到底去哪个名称空间中查找别名

在总路由中：

```python
url(r'^app01/',include('app01.urls',namespace='app01')),
url(r'^app02/',include('app02.urls',namespace='app02'))
```

前端：

```html
<a href="{% url 'app01:reg' %}"></a>
<a href="{% url 'app02:reg' %}"></a>
```

后端：

```python
print(reverse('app01:reg'))
print(reverse('app02:reg'))
```

**但是也可以不用，你只要 保证起别名的时候，在整个django项目中不冲突即可**



## django后端获取文件对象

form表达传文件需要注意的事项

- method必须改成post
- enctype改为formdata格式

```python
# urls.py
from django.conf.urls import url
from app02 import views


urlpatterns = [
    url(r'^upload/', views.upload)
]


# views.py
def upload(request):
    if request.method == 'POST':
        print(request.FILES)   # django会将文件类型的数据自动放入request.FILES
        file_obj = request.FILES.get('myfile')  # 文件对象
        # print(file_obj)
        # print(file_obj.name)
        with open(file_obj.name, 'wb') as f:
            for line in file_obj:
                f.write(line)
    return render(request, 'upload.html')
```