# django静态文件

## 静态文件

默认情况下所有的html文件都是放在templates文件夹内

### 什么是静态文件

网站所使用的提前写的css、js 第三方的前端模块、图片都叫做静态资源

默认情况下网站使用的静态资源全部会放到static文件夹下

通常情况下 在static文件夹内部还会再建其他文件夹 这是为了更加方便地管理文件，在django中 需要你自己手动创建静态文件存放的文件夹

```
css   文件夹
js    文件夹
font  文件夹
img   文件夹
Bootstrap
```

注意点：视图函数都必须有返回值，并且返回值都是HttpResponse对象

### 静态文件的配置

django后端如果想要暴露后端资源，必须去urls里面开设对应的资源接口

在项目文件夹下`settings.py`配置：

```python
STATIC_URL = '/static/'  # 访问静态文件资源接口前缀

# 手动开设静态文件访问资源
STATICFILES_DIRS = [     # 静态资源所在的文件所i在文件夹路径
    os.path.join(BASE_DIR, 'static'),  # 将static里面的所有资源暴露给用户
    os.path.join(BASE_DIR, 'static1')   # static找不到会往下找，逐层找
]
```

### 静态文件动态绑定

```html
{% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.7-dist/css/bootstrap.min.css' %}">
    <script src="{% static 'bootstrap-3.3.7-dist/js/bootstrap.min.js' %}"></script>
```

### 注意事项

form表单默认是get请求，get请求也能够携带参数
格式：`http://127.0.0.1:8000/login/?username=cwz&password=123`

特点：

- 携带的数据不安全
- 携带的数据大小有限制
- 通常只会携带一些不是很重要的数据

前期在朝后端提交post请求出现403的情况，需要在配置文件中注释掉一行内容：

```python
'django.middleware.csrf.CsrfViewMiddleware',
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```



## request方法初识

在django中后端的视图函数，无论是发的get请求还是post请求，都会执行视图函数，默认处理的是get请求。

- get请求指向拿到login页面
- post请求想提交数据，然后后端做校验

判断当前请求方式：

- 利用`request.method` 拿到的字符串大写的请求方式

```python
def login(request):
    # print('哈哈哈')
    # print(request.method)
    # print(type(request.method))
    if request.method == 'POST':
        return HttpResponse('收到了')
    return render(request, 'login.html')
```

1. `request.method` 获取请求方式，得到纯大写的字符串 GET POST

2. `request.POST` 获取用户提交的post请求数据

```
request.POST.get('username')    # 默认只取列表最后一个元素
request.POST.getlist('username')   # 获取列表
```

3. `request.GET` 获取用户提交的get请求数据

```python
request.GET.get()  # 默认只会获取列表最后一个元素
request.GET.getlist()  # 如果你想获取列表 用getlist()
```