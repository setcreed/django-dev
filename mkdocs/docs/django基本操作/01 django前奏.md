# django基本概念

## 前言

### web框架本质

我们可以这样理解：所有的Web应用本质上就是一个socket服务端，而用户的浏览器就是一个socket客户端。 这样我们就可以自己实现Web框架了。

### 服务器程序和应用程序

对于真实开发中的python web程序来说，一般会分为两部分：服务器程序和应用程序。

服务器程序负责对socket服务器进行封装，并在请求到来时，对请求的各种数据进行整理。

应用程序则负责具体的逻辑处理。为了方便应用程序的开发，就出现了众多的Web框架，例如：Django、Flask、web.py 等。不同的框架有不同的开发方式，但是无论如何，开发出的应用程序都要和服务器程序配合，才能为用户提供服务。

这样，服务器程序就需要为不同的框架提供不同的支持。这样混乱的局面无论对于服务器还是框架，都是不好的。对服务器来说，需要支持各种不同框架，对框架来说，只有支持它的服务器才能被开发出的应用使用。

这时候，标准化就变得尤为重要。我们可以设立一个标准，只要服务器程序支持这个标准，框架也支持这个标准，那么他们就可以配合使用。一旦标准确定，双方各自实现。这样，服务器可以支持更多支持标准的框架，框架也可以使用更多支持标准的服务器。

**WSGI**（Web Server Gateway Interface）就是一种规范，它定义了使用Python编写的web应用程序与web服务器程序之间的接口格式，实现web应用程序与web服务器程序间的解耦。

常用的WSGI服务器有uwsgi、Gunicorn。而Python标准库提供的独立WSGI服务器叫wsgiref，Django开发环境用的就是这个模块来做服务器。

### WSGI

wsgi server (比如uWSGI) 要和 wsgi application(比如django )交互,uwsgi需要将过来的请求转给django 处理,那么uWSGI 和 django的交互和调用就需要一个统一的规范,这个规范就是WSGI WSGI(Web Server GatewayInterface)。
WSGI,全称 Web Server Gateway Interface,或者 Python Web Server Gateway Interface ,是为Python 语言定义的 Web 服务器和 Web 应用程序或框架之间的一种简单而通用的接口。
WSGI 的官方定义是,the Python Web Server Gateway Interface。从名字就可以看出来,这东西是一个Gateway,也就是网关。网关的作用就是在协议之间进行转换。
WSGI 是作为 Web 服务器与 Web 应用程序或应用框架之间的一种低级别的接口



## python三大主流web框架

### django

大型框架，自带的组件和功能非常多；不足之处就是写小项目时，可能会很笨重

- socket部分用的别人的wsgiref（django默认的）
- 路由匹配自己写的
- 模板语法是自己写的

### flask

小而精 短小精悍 自带的组件和功能特别特别少 ，基本全部依赖于第三方组件

不足之处: 受限于第三方模块的影响比较大

如果将flask所有第三方模块加起来 能够直接盖过django

- socket部分用的别人的werkzeug
- 路由匹配自己写的
- 模板语法用的别人的jinja2

### torndao

特点：异步非阻塞 这个框架甚至可以用来开发游戏服务器

socket、路由匹配、模板语法都是自己写的

## Django安装配置

### 注意事项

- 计算机名称不能有中文
- python解释器不要使用3.7版本，推荐使用3.4-3.6版本
- 一个pycharm窗口 只能跑一个项目
- django版本使用1.11

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212153648.png)



- 安装django指定版本 两种方法
  - pip安装：`pip install django==1.11.11`
  - pycharm安装指定版本
- 检验django是否安装成功： 命令行执行：`django-admin`，有返回结果说明安装成功

### 命令行创建项目

- 创建django项目 ：`django-admin startproject 项目名(如mysite)`
- 启动django项目：`python manager.py runserver`，或者指定ip和端口：

```python
python manager.py runserver 127.0.0.1:8000   # 默认启动在本地8000端口
```

- 创建app应用（django支持多app开发）：`python manager.py startapp app01`

### app的概念

django是一个以开发app为只要功能的web框架，app就是application应用的意思

一个django项目就相当于一所大学，而一个个app相当于大学里的学院。

而 一个空的django项目本身没有任何作用，仅仅为app提供前期的环境配置，app才是一个个具体的功能，每个app都有自己独立的功能。

**注意：创建好的app需要在django配置文件中注册方可生效：**

```python
# 在配置文件settings.py中配置
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01',  # 简易写法
    'app01.apps.App01Config'   # 完整写法
]
```



**命令行传创建项目注意：**

- **不会自动创建templates文件夹，需要手动创建**
- **templates路径需要手动添加到配置文件中`'DIRS': [os.path.join(BASE_DIR, 'templates')]`**



## pycharm创建项目

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212153749.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212153822.png)



**注意：**

- **一定要确保同一个端口 同一时间只能启一个django项目**
- **templates路径如果没有，需要手动添加到配置文件中**`'DIRS': [os.path.join(BASE_DIR, 'templates')]`

### 目录结构

```python
mysite/
├── manage.py  # django的入口文件
├── templates  # 模板文件夹，存放html文件
├── db.splite3  # django自带的数据库文件
└── mysite  # 项目目录
    ├── __init__.py
    ├── settings.py  # 暴露给用户可以配置的配置文件
    ├── urls.py  # 路由 --> URL和函数的对应关系
    └── wsgi.py  # runserver命令就使用wsgiref模块做简单的web server
├── app01   # 应用文件夹名
    ├── migrations文件夹     # 所有数据库相关的操作记录
    ├── __init__.py
    ├── admin.py     # django admin后台管理
    ├── apps.py      # 注册qpp使用
    ├── models.py     # 放数据库相关的模型类
    ├── tests.py      # 测试文件
    ├── views.py      # 处理业务逻辑的视图函数
```

