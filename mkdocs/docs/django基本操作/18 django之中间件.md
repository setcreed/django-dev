# django之中间件

## 中间件简介

django中间件就类似于django的门户，所有的请求和响应都必须经过中间件才能正常通过，可以用来处理Django的请求和响应的数据。django中间件在设计到一些全局方面的功能时，作用非常大。每个中间件组件都负责做一些特定的功能。django默认有七个中间件。

django   settings.py中  MIDLLEWARE配置项：

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```



## 自定义中间件

django默认有七个中间件， 并且支持自定义中间件，给用户暴露5个自定义的方法

只要你想要做一些网站的全局性功能，你都应该考虑使用django的中间件

1. 全局的用户登录校验
2. 全局的用户访问频率限制
3. 全局的用户权限校验

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151647.png)

### process_request（重点）

```python
from django.utils.deprecation import MiddlewareMixin

class MyMdd1(MiddlewareMixin):
    def process_request(self, request):
        print('我是第一个中间件里的process_request方法')


class MyMdd2(MiddlewareMixin):
    def process_request(self, request):
        print('我是第二个中间件里的process_request方法')
```

![](https://setcreed-1258791144.cos.ap-shanghai.myqcloud.com/picgo-img/20191204210025.png)

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151856.png)

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151931.png)

总结：

- 请求来的时候会按照settings配置文件中从上往下的顺序，依次执行每一个中间件内部定义的process_request方法
- 如果中间件内部没有该方法，直接跳过执行下一个中间件
- 该方法一旦返回了HttpResponse对象，那么请求会立刻停止往下走，经过同级别的process_response方法原路返回。

### process_response（重点）

```python
def process_response(self, request, response):
    """
    :param request:
    :param response: 就是后端返回给前端的数据
    :return:
    """
    
    print('我是第一个中间件里的process_response方法')
    return response   # 必须要返回response
```

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305152258.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305152351.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305152423.png)

总结：

- 响应走的时候会按照settings配置文件中从下往上的顺序， 依次执行每一个中间件中的process_response方法
- 该方法必须要有两个形参，并且必须返回response形参，不返回直接报错
- 该方法自定义的HttpResponse对象，前端就会得到什么

### process_view

```python
def process_view(self, request, view_func, *args, **kwargs):
    print('我是第一个中间件里的process_view方法')
```

process_view(self, request, view_func, view_args, view_kwargs)， 该方法有四个参数

- 路由匹配成功之后，执行视图函数之后触发
- 如果该方法返回了HttpResponse对象， 那么会从下往上一次经过每一个中间件里的process_response方法

### process_templates_response

process_template_response(self, request, response)

视图函数要这样写，才能触发：

```python
def func(request):
    print('我是func函数')

    def render():
        return HttpResponse('你好吗')

    obj = HttpResponse('我不好')
    obj.render = render

    return obj
```



当你返回的对象中含有render属性指向的是一个render方法的时候才能触发， 是从下往上的顺序



### process_exception

```python
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse, render, redirect

class MyMdd1(MiddlewareMixin):
    def process_request(self, request):

        print('我是第一个中间件里的process_request方法')
        # return HttpResponse('我是中间件1')

    def process_response(self, request, response):
        """

        :param request:
        :param response: 就是后端返回给前端的数据
        :return:
        """
        print('我是第一个中间件里的process_response方法')
        return response
        # return HttpResponse('我是中间件1')


    def process_view(self, request, view_func, *args, **kwargs):
        print('我是第一个中间件里的process_view方法')


    def process_exception(self, request, exception):
        print('我是第一个中间件里的process_exception方法')



class MyMdd2(MiddlewareMixin):
    def process_request(self, request):
        print('我是第二个中间件里的process_request方法')



    def process_response(self, request, response):
        print('我是第二个中间件里的process_response方法')
        return response
        # return HttpResponse('我是中间件2')


    def process_view(self, request, view_func, *args, **kwargs):
        print('我是第二个中间件里的process_view方法')


    def process_exception(self, request, exception):
        print('我是第二个中间件里的process_exception方法')
```



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305152637.png)

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305152840.png)


- 当视图函数中出现错误时，会自动触发，顺序是settings配置文件从下往上

