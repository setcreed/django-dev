# django之forms组件

## forms组件

先抛出一个需求：

```
1.写一个注册功能，获取用户输入的用户名和密码，提交到后端，后端做校验
2.用户名里面不能含有敏感信息，给出相应的提示
3.如果密码小于三位，就提示用户
```



### 手动书写需求

views.py

```python
def register(request):
    errors = {'username':'', 'password':''}
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')

        if 'zf' in username:
            errors['username'] = '不能使用该字符'
        if len(password) < 4:
            errors['password'] = '密码不能小于三位'

    return render(request, 'register.html', locals())
```

register.html

```html
<form action="" method="post">
    <p>
        username:<input type="text" name="username">
        <span style="color: red">{{ errors.username }}</span>
    </p>
    <p>
        password:<input type="text" name="password">
        <span style="color: red">{{ errors.password }}</span>
    </p>
    <input type="submit">
</form>
```



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305114802.png)

这里实现了三个功能：

- 手写html页面获取用户输入信息
- 将数据传入后端做数据校验
- 如果有错误，展示错误信息

但是这个页面手写麻烦，输入信息写错了，一刷新信息全没了，很不友好！！

## 使用forms组件校验数据

使用forms组件首先要导入forms模块， 写这玩意类似于models

```python
from django import forms

class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3)
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3)
    # email字段  必须是邮箱格式
    email = forms.EmailField()
```

然后对数据进行校验， 可以写一个测试脚本，还可以使用pycharm左下角的Python Console功能，会自动搭建测试脚本

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305115132.png)

使用测试

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305115229.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305115319.png)

校验数据的方法：

1. 给写好的类 传字典数据(待校验的数据)

```python
form_obj = views.MyForm({'username':'cwz','password':'12','email':'123'})
```



2. 如何查看校验的数据是否合法

```python
form_obj.is_valid()    # 只有全部数据符合校验规则才为True
```



3. 如何查看不符合规则的字段及错误的理由

```python
form_obj.errors`
```



4. 如何查看符合校验规则的数据

```python
form_obj.cleaned_data
```



5. forms组件中 定义的字段默认都是必须传值的  不能少传

6. forms组件只会校验forms类中定义的字段  如果你多传了 不会有任何影响

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305120633.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305120700.png)



## forms组件渲染标签

### 1. 方式一

forms组件只会帮你渲染用户输入的标签，不会帮渲染提交按钮标签。

views.py

```python
def index(request):
    # 渲染标签，先生成一个空的form类的对象
    form_obj = MyForm()

    return render(request, 'index.html', locals())
```

前端页面：

```html
<p>forms组件渲染的方式1</p>
{{ form_obj.as_p }}
<br>
{{ form_obj.as_ul }}
<br>
{{ form_obj.as_table }}
```

效果：

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305121844.png)

总结：这种渲染标签的方式封装程度态高    不推荐使用    但是可以用在本地测试

### 2. 方式二

不推荐使用，比较麻烦

```html
<p>forms组件渲染标签方式2：</p>
{{ form_obj.username.label }}{{ form_obj.username }}
{{ form_obj.password.label }}{{ form_obj.password }}
{{ form_obj.email.label }}{{ form_obj.email }}
```

### 3.方式三

```html
<p>forms组件渲染标签方式3：</p>
{% for form in form_obj %}
    <p>{{ form.label }} {{ form }}</p>
{% endfor %}
```



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305121958.png)

若想要label标签显示中文，可以指定label标签：

```python
from django import forms

class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名')
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码')
    # email字段  必须是邮箱格式
    email = forms.EmailField(label='邮箱')
```

## forms组件展示信息

```html
<form action="" method="post">
    {% for form in form_obj %}
        <p>{{ form.label }} {{ form }}</p>
    {% endfor %}
    <input type="submit">
</form>
```



views.py

```python
from django import forms

class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名')
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码')
    # email字段  必须是邮箱格式
    email = forms.EmailField(label='邮箱')

def index(request):
    # 渲染标签，先生成一个空的forms类的对象
    form_obj = MyForm()

    if request.method == 'POST':
        form_obj = MyForm(request.POST)
        if form_obj.is_valid():
            print(form_obj.cleaned_data)
            return HttpResponse('数据正确')
        else:
            print(form_obj.errors)

    return render(request, 'index.html', locals())
```

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305122112.png)

这玩意是前端做的校验

注意：

**数据的校验通常前后端都有，但是前端的校验不堪一击，可有可无；后端的校验必须要有而且必须非常全面**

在前端form表单加上一个参数（**novalidate**），就可以不做校验：`<form action="" method="post" novalidate>`

前端错误信息展示写法：

```html
<form action="" method="post" novalidate>
    {% for form in form_obj %}
        <p>{{ form.label }} {{ form }}
            <span>{{ form.errors.0 }}</span>
        </p>

    {% endfor %}
    <input type="submit">
</form>
```



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305122920.png)



也支持中文显示信息

```python
from django import forms

class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名', error_messages={
        'max_length': '用户名最长八位',
        'min_length': '用户名最短三位',
        'required': '用户名不能为空',
    })
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码', error_messages={
        'max_length': '密码最长八位',
        'min_length': '密码最短三位',
        'required': '密码不能为空',
    })
    # email字段  必须是邮箱格式
    email = forms.EmailField(label='邮箱', error_messages={
        'required': '邮箱不能为空',
        'invalid': '邮箱格式不正确'
    })
```



## form组件自定义校验

### RegexValidator验证器

```python
from django import forms
from django.core.validators import RegexValidator

class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名', error_messages={
        'max_length': '用户名最长八位',
        'min_length': '用户名最短三位',
        'required': '用户名不能为空',
    })
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码', error_messages={
        'max_length': '密码最长八位',
        'min_length': '密码最短三位',
        'required': '密码不能为空',
    }, validators=[
        RegexValidator(r'^[0-9]+$', '请输入数字'),
        RegexValidator(r'^139[0-9]+$', '数字必须要以139开头')   # 可以添加多个正则表达式，从上往下校验
    ]
    )
```



### 钩子函数  (HOOK)

当你觉得上面的所有校验还是不能满足需求，可以考虑钩子函数

#### 全局钩子

我们在Fom类中定义 clean() 方法，就能够实现对字段进行全局校验。

```python
class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名', error_messages={
        'max_length': '用户名最长八位',
        'min_length': '用户名最短三位',
        'required': '用户名不能为空',
    })
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码', error_messages={
        'max_length': '密码最长八位',
        'min_length': '密码最短三位',
        'required': '密码不能为空',
    })

    re_password = forms.CharField(max_length=8, min_length=3, label='确认密码', error_messages={
        'max_length': '确认密码最长八位',
        'min_length': '确认密码最短三位',
        'required': '确认密码不能为空',
    })

    # 校验密码 确认密码是否一致
    def clean(self):
        password = self.cleaned_data.get('password')
        re_password = self.cleaned_data.get('password')
        if not password == re_password:
            self.add_error('re_password', '两次，密码不一致')

        return self.cleaned_data
```

#### 局部钩子

我们在Fom类中定义 clean_字段名() 方法，就能够实现对特定字段进行校验。

```python
class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名', error_messages={
        'max_length': '用户名最长八位',
        'min_length': '用户名最短三位',
        'required': '用户名不能为空',
    })
    # password字段 最多八位， 最少三位
    password = forms.CharField(max_length=8, min_length=3, label='密码', error_messages={
        'max_length': '密码最长八位',
        'min_length': '密码最短三位',
        'required': '密码不能为空',
    })

    re_password = forms.CharField(max_length=8, min_length=3, label='确认密码', error_messages={
        'max_length': '确认密码最长八位',
        'min_length': '确认密码最短三位',
        'required': '确认密码不能为空',
    })

    # 校验用户名中不能有666
    def clean_username(self):
        username = self.cleaned_data.get('username')
        if '666' in username:
            # 给username字段添加错误信息
            self.add_error('username', '666是不存在的')
        # 将username返回出去
        return username
```

## forms组件补充知识点

### 其他字段参数

label   input对应的提示信息

initial   默认值

required    默认为True  控制字段是否必填

```python
class MyForm(forms.Form):
    # username字段 最多八位， 最少三位
    username = forms.CharField(max_length=8, min_length=3, label='用户名', initial='默认值', 
        error_messages={
        'max_length': '用户名最长八位',
        'min_length': '用户名最短三位',
        'required': '用户名不能为空',
                        }, required=False)
```

widget 给input框设置样式及属性

```python
password = forms.CharField(max_length=8, min_length=3, label='密码', error_messages={
        'max_length': '密码最长八位',
        'min_length': '密码最短三位',
        'required': '密码不能为空',
    }, widget=forms.widgets.PasswordInput()  # 这个password字段是密文的
```

```python
username = forms.CharField(max_length=8, min_length=3, label='用户名', initial='默认值',
                               error_messages={
                                   'max_length': '用户名最长八位',
                                   'min_length': '用户名最短三位',
                                   'required': '用户名不能为空',
                               }, required=False,
                               widget=forms.widgets.TextInput({'class': 'form-control c1 c2', 'username': 'cwz'})
                               )
```

#### error_messages

重写错误信息。

```python
class LoginForm(forms.Form):
    username = forms.CharField(
        min_length=8,
        label="用户名",
        initial="张三",
        error_messages={
            "required": "不能为空",
            "invalid": "格式错误",
            "min_length": "用户名最短8位"
        }
    )
    pwd = forms.CharField(min_length=6, label="密码")
```

#### password

```python
class LoginForm(forms.Form):
    ...
    pwd = forms.CharField(
        min_length=6,
        label="密码",
        widget=forms.widgets.PasswordInput(attrs={'class': 'c1'}, render_value=True)
    )
```

#### radioSelect

单radio值为字符串

```python
class LoginForm(forms.Form):
    username = forms.CharField(
        min_length=8,
        label="用户名",
        initial="张三",
        error_messages={
            "required": "不能为空",
            "invalid": "格式错误",
            "min_length": "用户名最短8位"
        }
    )
    pwd = forms.CharField(min_length=6, label="密码")
    gender = forms.fields.ChoiceField(
        choices=((1, "男"), (2, "女"), (3, "保密")),
        label="性别",
        initial=3,
        widget=forms.widgets.RadioSelect()
    )
```

#### 单选Select

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.ChoiceField(
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
        label="爱好",
        initial=3,
        widget=forms.widgets.Select()
    )
```

#### 多选Select

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.MultipleChoiceField(
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
        label="爱好",
        initial=[1, 3],
        widget=forms.widgets.SelectMultiple()
    )
```

#### 单选checkbox

```python
class LoginForm(forms.Form):
    ...
    keep = forms.ChoiceField(
        label="是否记住密码",
        initial="checked",
        widget=forms.widgets.CheckboxInput()
    )
```

#### 多选checkbox

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.MultipleChoiceField(
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"),),
        label="爱好",
        initial=[1, 3],
        widget=forms.widgets.CheckboxSelectMultiple()
    )
```

#### choice字段注意事项

在使用选择标签时，需要注意choices的选项可以配置从数据库中获取，但是由于是静态字段 获取的值无法实时更新，需要重写构造方法从而实现choice实时更新

方式一

```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields

 
class MyForm(Form):
 
    user = fields.ChoiceField(
        # choices=((1, '上海'), (2, '北京'),),
        initial=2,
        widget=widgets.Select
    )
 
    def __init__(self, *args, **kwargs):
        super(MyForm,self).__init__(*args, **kwargs)
        # self.fields['user'].choices = ((1, '上海'), (2, '北京'),)
        # 或
        self.fields['user'].choices = models.Classes.objects.all().values_list('id','caption')
```

方式二

```python
from django import forms
from django.forms import fields
from django.forms import models as form_model

 
class FInfo(forms.Form):
    authors = form_model.ModelMultipleChoiceField(queryset=models.NNewType.objects.all())  # 多选
    # authors = form_model.ModelChoiceField(queryset=models.NNewType.objects.all())  # 单选
```