# django和ajax

## Ajax简介

Ajax（Asynchronous Javascript And XML）翻译成中文就是“异步的Javascript和XML”。即使用Javascript语言与服务器进行异步交互，传输的数据为XML（当然，传输的数据不只是XML）。

**ajax是异步提交的**

Ajax 不是新的编程语言，而是一种使用现有标准的新方法。

**Ajax 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。（这一特点给用户的感受是在不知不觉中完成请求和响应过程）**

举个实例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<input type="text" id="t1"> + <input type="text" id="t2"> = <input type="text" id="t3">
<p>
    <button id="b1">计算</button>
</p>
<script>
    $('#b1').on('click', function () {
        $.ajax({

            url:'',    // 数据提交的地址， 不写就是向当前页面提交，也可以写后缀，也可以写全称，与form表单参数action一样
            type: 'post',   // 提交方式，不写默认是get请求
            data: {'t1': $('#t1').val(), 't2':$('#t2').val()}, // 提交的数据
            success: function (data) {     // 形参data就是异步提交之后后端返回的结果
                $('#t3').val(data)
            }


        })
    })
</script>
</body>
</html>
```

## 数据传输编码格式的解析

前后端交互式一个数据编码格式，针对不同的数据，后端会进行不同的处理

三种数据编码格式：

- urlencoded
- formdata
- application/json

### form表单发送三种数据格式的情况

#### urlencoded

form表单post请求默认的编码格式是urlencoded

在浏览器-->检查-->network可以看到，我们form表单在提交数据的时候，有如下信息：

```
Request Headers：    # 请求头
Content-Type:application/x-www-form-urlencoded; charset=UTF-8   # 数据编码格式-urlencoded

Form Data：# 携带的数据
d1=23&d2=23
```

在我们后端django中针对urlencoded数据，会自动解析并封装到request.POST方法中

#### form表单发送文件

```
Request Headers：    # 请求头
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryhjKCHQHDmcE62iMQ  # 数据编码格式，-form-data

Form Data：#针对form-data格式的数据，在浏览器是无法查看的
```

发送到后端django，文件对象会自动解析到 request.POST 和 request.FILES 中，前者记录文件名，后者记录对象。

form表单无法发送json格式的数据，要想实现，只能借助ajax

### ajax发送数据的编码格式

默认的编码格式是urlencoded

#### ajax传输json格式数据

有个参数，`contentType`，不写默认是urlencoded，

在view.py中：

```python
import json

def home(request):
    if request.method == "POST":
        print(request.body)
        json_bytes = request.body
        print(json.loads(json_bytes), type(json.loads(json_bytes)))
        # 反序列化为python字典格式
    return render(request, 'form_test.html')


# 结果：
'''

b'{"d1":"cwz","d2":"123"}'
{'d1': 'cwz', 'd2': '123'} <class 'dict'>
'''
```

form_test.html文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<form action="" method="post">

    username: <input type="text" name="username" id="d1">
    password: <input type="text" name="password" id="d2">
</form>
<button id="d3">点我发送json格式</button>

<script>
    $('#d3').click(function () {
        $.ajax({
            url:'',
            type:'post',
            contentType:'application/json', //需要指定编码格式为json
            data:JSON.stringify({'d1':$('#d1').val(),'d2':$('#d2').val()}), // 需要前端发送JSON字符串，JSON.stringify序列化即可。
            success:function (data) {
                alert(123)
            }
        })
    })
</script>
</body>
</html>
```

ajax传json格式数据特点：

**django后端针对json格式的数据 不会自动帮你解析 会直接原封不动的给你放到request.body中 你可以手动处理 获取数据。拿到request.body是一个bytes类型数据**

## Ajax传输文件数据

需要借助内置对象，该对象既可以携带文件数据，同样也支持普通的键值对

注意几个参数：

- `data：formdata`对象
- `contentType：false`
- `processData：false`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
username:<input type="text" name="username">
password:<input type="text" name="password">
myfile:<input type="file" name="myfile" id="d1">
<button id="d2">点我发送文件</button>

<script>


    $('#d2').click(function () {
        // 先生成一个内置对象
        var MyFormData = new FormData();
        // 先添加普通的键值
        MyFormData.append('username', 'cwz');
        MyFormData.append('password', '123');
        //添加文件数据
        MyFormData.append('myfile', $('#d1')[0].files[0]);  // 将jquery对象转换成原生的js对象,利用原生js对象的方法 直接获取文件内容

        $.ajax({

            url: '',
            type: 'post',
            data: MyFormData,

            contentType: false,   //不用任何编码,因为formData对象自带编码 django能够识别该对象
            processData: false,   //告诉浏览器不要处理我的数据 直接发就行

            success: function (data) {
                
            }

        })
    })

</script>
</body>
</html>
```



## django内置序列化模块

序列化的目的就是 将数据整合成一个大字典

导入这个模块：`from django.core import serializers`

比自己用json转方便多了

```python
from app01 import models
from django.core import serializers

def yyy(request):
    author_queryset = models.Author.objects.all()
    res = serializers.serialize('json', author_queryset)
    return HttpResponse(res)
```

效果：

```json
[{
	"model": "app01.author",
	"pk": 1,
	"fields": {
		"name": "\u90b6\u98ce",
		"email": "123@qq.com",
		"author_detail": 1
	}
}, {
	"model": "app01.author",
	"pk": 2,
	"fields": {
		"name": "\u5f0f\u5fae",
		"email": "111@sin.com",
		"author_detail": 2
	}
}, {
	"model": "app01.author",
	"pk": 3,
	"fields": {
		"name": "\u65e0\u540d",
		"email": "100@qq.com",
		"author_detail": 3
	}
}]
```



## ajax结合sweetalert使用

点击下载[Bootstrap-sweetalert](https://github.com/lipis/bootstrap-sweetalert)

一通CV大法：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>

    {% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.7-dist/css/bootstrap.min.css' %}">
    <link rel="stylesheet" href="{% static 'dist/sweetalert.css' %}">
    <script src="{% static 'bootstrap-3.3.7-dist/js/bootstrap.min.js' %}"></script>
    <script src="{% static 'dist/sweetalert.min.js' %}"></script>

</head>
<body>

<div class="container-fluid">
    <div class="row">
        <div class="col-md-8 col-md-offset-2">
            <h2 class="text-center">数据展示</h2>
            <br>
            <table class="table-bordered table table-striped table-hover">
                <thead>
                <tr>
                    <th>序号</th>
                    <th>用户名</th>
                    <th>年龄</th>
                    <th>性别</th>
                    <th class="text-center">操作</th>
                </tr>
                </thead>
                <tbody>
                {% for user in user_queryset %}
                    <tr>
                        <td>{{ forloop.counter }}</td>
                        <td>{{ user.username }}</td>
                        <td>{{ user.age }}</td>
                        <td>{{ user.get_gender_display }}</td>
                        <td class="text-center">
                            <a href="#" class="btn btn-primary btn-sm">编辑</a>
                            <a href="#" class="btn btn-danger btn-sm cancel">删除</a>
                        </td>
                    </tr>
                {% endfor %}

                </tbody>
            </table>
        </div>
    </div>
</div>

<script>
    $('.cancel').click(function () {
        swal({
                title: "你确定删除吗?",
                text: "如果删了，你就跑路吧！",
                type: "warning",
                showCancelButton: true,
                confirmButtonClass: "btn-danger",
                confirmButtonText: "是的，我就要删！",
                cancelButtonText: "不删了",
                closeOnConfirm: false,
                closeOnCancel: false
            },
            function (isConfirm) {
                if (isConfirm) {
                    swal("准备跑路吧！", "跑不了了。。。", "success");
                } else {
                    swal("取消删除", "数据还在", "error");
                }
            });
    })
</script>

</body>
</html>
```

这里有个问题，发现汉字被挡住了。。。

![img](https://i.loli.net/2019/12/02/rfMyK4RJBYDFksW.png)

通过谷歌浏览器的检查，查看html元素修改，加上样式即可：

```css
<style>
    div.sweet-alert h2 {
    padding: 10px;
    }
</style
```

最终的实例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>

    {% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.7-dist/css/bootstrap.min.css' %}">
    <link rel="stylesheet" href="{% static 'dist/sweetalert.css' %}">
    <script src="{% static 'bootstrap-3.3.7-dist/js/bootstrap.min.js' %}"></script>
    <script src="{% static 'dist/sweetalert.min.js' %}"></script>
    <style>
        div.sweet-alert h2 {
            padding: 10px;
        }
    </style>
</head>
<body>

<div class="container-fluid">
    <div class="row">
        <div class="col-md-8 col-md-offset-2">
            <h2 class="text-center">数据展示</h2>
            <br>
            <table class="table-bordered table table-striped table-hover">
                <thead>
                <tr>
                    <th>序号</th>
                    <th>用户名</th>
                    <th>年龄</th>
                    <th>性别</th>
                    <th class="text-center">操作</th>
                </tr>
                </thead>
                <tbody>
                {% for user in user_queryset %}
                    <tr>
                        <td>{{ forloop.counter }}</td>
                        <td>{{ user.username }}</td>
                        <td>{{ user.age }}</td>
                        <td>{{ user.get_gender_display }}</td>
                        <td class="text-center">
                            <a href="#" class="btn btn-primary btn-sm">编辑</a>
                            <a href="#" class="btn btn-danger btn-sm cancel" userId = {{ user.pk }}>删除</a>
                        </td>
                    </tr>
                {% endfor %}

                </tbody>
            </table>
        </div>
    </div>
</div>

<script>
    $('.cancel').click(function () {
        var $btn = $(this);
        swal({
                title: "你确定删除吗?",
                text: "如果删了，你就跑路吧！",
                type: "warning",
                showCancelButton: true,
                confirmButtonClass: "btn-danger",
                confirmButtonText: "是的，我就要删！",
                cancelButtonText: "不删了",
                closeOnConfirm: false,
                closeOnCancel: false,
                showLoaderOnConfirm: true
            },
            function (isConfirm) {
                if (isConfirm) {

                    // 朝后端发送ajax请求
                    $.ajax({
                        url: '',
                        type: 'post',
                        data: {'delete_id': $btn.attr('userId')},
                        success: function (data) {
                            if(data.code==1000){
                                swal("准备跑路吧！", data.msg, "success");

                                // 通过DOM操作直接操作标签
                                $btn.parent().parent().remove()

                            }else {
                                swal("有bug", "发生了未知错误", "warning")
                            }
                        }
                    });

                } else {
                    swal("取消删除", "数据还在", "error");
                }
            });
    })
</script>

</body>
</html>
```

后端views.py

```python
def home(request):

    if request.is_ajax():
        back_dic = {'code': 1000, 'msg': ''}
        delete_id = request.POST.get('delete_id')
        time.sleep(3)
        models.User.objects.filter(pk=delete_id).delete()
        back_dic['msg'] = '数据已经被我删掉了'
        return JsonResponse(back_dic)

    user_queryset = models.User.objects.all()
    return render(request, 'home.html', locals())
```