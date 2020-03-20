# django之视图层

## 视图层函数

在视图层，三个重要的方法：HttpResponse、render、redirect

在视图函数必须要有一个返回值，并且返回值的数据类型必须是HttpResponse对象

原理：利用两个模块

```python
from django.template import Template,Context
def login(request):
    res = Template('<h1>{{ user }}</h1>')
    con = Context({'user':{'username':'cwz','password':'123'}})
    return HttpResponse(res.render(con))
```



## JsonResponse对象

### 前后端数据交互

通常情况下，前后端数据交互采用的都是json的字符串。后端只需要写好对应的url接口，前端访问这个接口。

用来告诉前端工程师，这个接口能够返回哪些数据

### 前后端序列化 反序列化使用的方法

| python后端 | JavaScript前端 |
| ---------- | -------------- |
| json.dumps | JSON.stringify |
| json.loads | JSON.parse     |

### JsonResponse

将一个字典序列化成字符串：

```python
import json

def index(request):
    user_dic = {'name': 'cwz你好啊', 'password': '123'}

    # ensure_ascii 参数控制对中文是否转码, 默认是True
    json_str = json.dumps(user_dic, ensure_ascii=False)
    return HttpResponse(json_str)
```

在django中有JsonResponse

```python
from django.http import JsonResponse


def index(request):
    user_dic = {'name': 'cwz你好啊', 'password': '123'}
    return JsonResponse(user_dic, json_dumps_params={'ensure_ascii': False})
```

JsonResponse默认序列化字典，如果想序列化其他数据类型，可以加safe参数

```python
from django.http import JsonResponse

def index(request):
    lt = [1,2,3,4,5,]
    return JsonResponse(lt, safe=False)
```

## FBV与CBV

### 区别

FBV ：基于函数的视图

CBV：基于类的视图

### CBV简单使用

在`urls.py`中写路由地址

```python
url(r'login/', views.MyLogin.as_view())
```

在视图层`views.py`中写

```python
from django.views import View

class MyLogin(View):
    def get(self, request):
        print('我是MyLogin里面的get方法')
        return render(request, 'login.html')

    def post(self, request):
        print('我是MyLogin的post方法')
        return HttpResponse('post')
```

运行上面的代码，可知CBV能够根据请求方式的不同，自动的执行不同的方法。

### CBV源码分析

首先看路由，`urls.py`中：

```python
url(r'login/', views.MyLogin.as_view())
```

经过分析可知：

- MyLogin是类，类.方法加括号，说明是访问方法
- 方法后面没有加参数，有两种可能，一个是类的绑定方法（classmethod）没有传参数，另一个就是加staticmethod装饰，就是一个普通的函数，没有传形参。
- `as_view()` 方法就是一个函数，函数名加括号执行优先级最高

进入`as_view`源码：

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212163300.png)

这样路由匹配关系就相当于这样：`url(r'login/', views.view)`。这就说明了CBV在路由匹配上，其实本质上还是FBV。



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212163344.png)

然后去 `dispatch`方法中看：

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200212163428.png)

总结CBV能够根据请求方式的不同，自动的执行不同的方法：

- 进入`as_view`源码，是一个类的绑定方法，返回一个view函数名
- 然后看view函数方法，里面给对象添加了一堆属性，最后返回`self.dispatch`, dispatch对象、类中都没有，在父类中有，在dispatch中分析
- 先判断当前请求方式是否在这8个请求方式之内，如果不在就报错
- 以get请求为例，利用反射去我们定义类的对象中查找get属性和方法。最终在自定义的类中找到get方法，即这里的`hander=get方法`，然后调用get方法，执行get函数。

### CBV加装饰器的方式

```python
# 计时任务的装饰器
import time
from functools import wraps

def outter(func):
    @wraps(func)
    def inner(*args, **kwargs):
        start = time.time()
        res = func(*args, **kwargs)
        end = time.time() - start
        print(f'函数执行时间：{end}')
        return res
    return inner

# 法1，直接写

class MyLogin(View):
    @outter
    def get(self, request):
        print('我是MyLogin里面的get方法')
        return render(request, 'login.html')
    
    @outter
    def post(self, request):
        print('我是MyLogin的post方法')
        return HttpResponse('post')
```



推荐使用django提供的模块写 `from django.utils.decorators import method_decorator`

```python
# 法2
from django.views import View
from django.utils.decorators import method_decorator

@method_decorator(outter, name='get')   # 可以指定给谁装
class MyLogin(View):
    def get(self, request):
        print('我是MyLogin里面的get方法')
        return render(request, 'login.html')
    
    
# 法3

class MyLogin(View):
    
    @method_decorator(outter)
    def get(self, request):
        print('我是MyLogin里面的get方法')
        return render(request, 'login.html')
    
    
    
# 法4
@method_decorator(outter, name='dispatch')
class MyLogin(View):
    #@method_decorator(outter)   # 也可以在dispatch方法上加装
    def dispatch(self, request, *args, **kwargs):
        return super().dispatch(request, *args, **kwargs)

    def get(self, request):
        print('我是MyLogin里面的get方法')
        return render(request, 'login.html')

    def post(self, request):
        print('我是MyLogin的post方法')
        return HttpResponse('post')
```

