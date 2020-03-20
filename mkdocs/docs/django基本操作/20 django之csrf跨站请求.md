# django之csrf跨站请求

## 跨站请求伪造 csrf

### 钓鱼网站

就类似于你搭建了一个跟银行一模一样的web页面 ,  用户在你的网站转账的时候输入用户名 密码 对方账户
银行里面的钱确实少了 但是发现收款人变了

原理实现:

```
你写的form表单中 用户的用户名  密码都会真实的提交给银行后台, 但是收款人的账户却不是用户填的 你暴露给用户的是一个没有name属性的input框,  你自己提前写好了一个隐藏的带有name和value的input框。
```

### 模拟实现

创建两个django项目



```html
<!--钓鱼网站-->
<p>这是伪造的网站</p>
<form action="http://127.0.0.1:8000/transfer/" method="post">
    <p>username:<input type="text" name="username"></p>
    <p>target_account:<input type="text" name="target_user">
        <input type="text" name="target_user" value="jason" style="display: none">
    </p>
    <p>money:<input type="text" name="money"></p>
    <input type="submit">
</form>
```

解决钓鱼网站的思路：

```
只要是用户想要提交post请求的页面，我在返回给用户的时候提前设置好字符串，中间件给前端加上一个隐藏的input框，里面的value就是这个随机字符串。
当用户提交post请求，我会自动先去查找是否有该随机字符串，如果有，正常提交，没有就返回403
```

在django中  中间件csrf  `'django.middleware.csrf.CsrfViewMiddleware',`， 它就是来校验有没有随机字符串。

这个中间件的功能：

```
当你向正规网站发送post请求， 请求来的时候 服务器给你一个随机字符串，响应的时候经过中间件。csrf中间件就会在form表单中找，有没有  <input type="hidden" name="csrfmiddlewaretoken" >， 如果有，就拿着token所对应的值去 给上一次浏览器返回的键去比对，如果比对上了就正常进去，否则403.
```

### 针对form表单

以后再写form表单时 ，只要在表单中加上`{% csrf_token %}`就可以了

```html
<p>这是正经网站</p>
{% csrf_token %}
<form action="" method="post">
    <p>username:<input type="text" name="username"></p>
    <p>target_account:<input type="text" name="target_user"></p>
    <p>money:<input type="text" name="money"></p>
    <input type="submit">
</form>
```

### ajax请求

方式一：

```
先在页面上任意位置写上 {% csrf_token %}
然后在发送ajax请求的时候，通过标签查找获取随机字符串添加到data自定义对象中即可
data:{'username': 'cwz', 'csrfmiddlewaretoken': $('input[name="csrfmiddlewaretoken"]').val()},
```



方式二：

```
data:{'username':'jason','csrfmiddlewaretoken':'{{ csrf_token }}'},
```

方式三：
利用别人写好的脚本文件

只要在static文件夹中先建一个js文件，存放脚本文件， 然后只需要把js脚本文件导过来即可

脚本文件：

```js
function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}
var csrftoken = getCookie('csrftoken');

function csrfSafeMethod(method) {
  // these HTTP methods do not require CSRF protection
  return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}

$.ajaxSetup({
  beforeSend: function (xhr, settings) {
    if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader("X-CSRFToken", csrftoken);
    }
  }
});
```

## csrf相关的两个装饰器

首先要导入模块

```
from django.views.decorators.csrf import csrf_exempt, csrf_protect
```



```python
@csrf_exempt   # 装饰了这个装饰器之后，就可以绕过校验了。
def exem(request):
    return HttpResponse('exempt')

@csrf_protect   # 装饰了这个装饰器就会被校验了。
def pro(request):
    return HttpResponse('pro')
```



两个装饰器在CBV上有何不同

csrf_exempt

这个装饰器只能给dispatch装

```python
from django.views.decorators.csrf import csrf_exempt, csrf_protect
from django.utils.decorators import method_decorator
# 第一种
# @method_decorator(csrf_exempt,name='dispatch')
class MyCsrf(View):
    # 第二种
    @method_decorator(csrf_exempt)
    def dispatch(self, request, *args, **kwargs):
        return super().dispatch(request,*args,**kwargs)
    
    def get(self,request):
        return render(request, 'transfer.html')
    
    def post(self, request):
        return HttpResponse('OK')
```



csrf_protect

怎么装饰都可以，与普通的装饰器装饰CBV一致

```python
@method_decorator(csrf_protect,name='post')   # 第一种
class MyCsrf(View):
    @method_decorator(csrf_protect)   #  第二种
    def dispatch(self, request, *args, **kwargs):
        return super().dispatch(request,*args,**kwargs)
    
    def get(self,request):
        return HttpResponse('hahaha')

    @method_decorator(csrf_protect)   # 第三种
    def post(self,request):
        return HttpResponse('post')
```

