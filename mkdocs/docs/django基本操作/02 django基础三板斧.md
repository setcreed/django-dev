# django基础必备三板斧

主要操作views.py和urls.py

## HttpResponse

内部传入一个字符串参数，返回给浏览器

```python
from django.shortcuts import render, HttpResponse, redirect
def index(request):
    return HttpResponse('有点意思')
```

## render

render可以返回html文件，可以给html页面传值

```python
def login(request):
    user_dic = {'username': 'neo', 'password': '123'}
    return render(request, 'login.html', {'user_dic':user_dic})
```

## redirect

重定向，接受一个URL参数，表示跳转指定的URL。

可以是后缀名，也可以是全路径

```python
def home(request):
    return redirect('/index')
    # return redirect('https://coding.net/')
```