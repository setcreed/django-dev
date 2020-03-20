# django之auth模块

我们在开发一个网站的时候，无可避免的需要设计实现网站的用户系统。此时我们需要实现包括用户注册、用户登录、用户认证、注销、修改密码等功能，这还真是个麻烦的事情呢。

Django作为一个完美主义者的终极框架，当然也会想到用户的这些痛点。它内置了强大的用户认证系统--auth，它默认使用 auth_user 表来存储用户数据。



先执行数据库迁移命令，django会自动产生几张表，里面就有auth_user表， 里面的字段还挺多的

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305154506.png)



创建超级用户

执行`python manage.py createsuperuser`

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305154529.png)



![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305154556.png)

然后你就可以进入django后台管理 进行操作

## auth模块常用方法

### 创建用户

```python
from django.contrib.auth.models import User

def register(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')

        User.objects.create_superuser(username=username, password=password, email='123@qq.com')   # 创建超级用户， 需要邮箱数据， 如果使用命令行创建就不需要
        return HttpResponse('ok')
    return render(request, 'register.html')
```



```python
User.objects.create(username=username,password=password)  # 这种创建的方式  密码不是加密的

User.objects.create_user(username=username,password=password)  # 创建普通用户

User.objects.create_superuser(username=username, password=password, email='123@qq.com')   # 创建超级用户， 需要邮箱数据， 如果使用命令行创建就不需要
```



### 校验用户名与密码是否正确

```python
from django.contrib import auth
user_obj = auth.authenticate(request, username=username, password=password)  # 自动给你加密密码，然后去数据库里校验
print(user_obj)  # 数据对象
print(user_obj.username)
print(user_obj.password)   # 密码是密文
```



### 保存用户登录状态

```python
from django.contrib import auth

auth.login(request, user_obj)
# 只要执行了这一句话，之后你可以在任意位置，可以通过request.user获取到当前登录的用户对象
# 如果没有登录，获取  AnonymousUser
```

### 判断当前用户是否登录

```python
request.user.is_authenticated()
```

### 校验原密码是否正确

```python
is_right = request.user.check_password(old_password)
```

### 修改密码

```python
request.user.set_password(new_password)
request.user.save()  # 修改密码一定要保存
```

### 注销

```python
auth.logout(request)
```



### 校验用户是否登录装饰器

```python
from django.contrib.auth.decorators import login_required

# 全局配置

@login_required(login_url='/login')  # 后面加自己跳转的页面，如果不加，默认是跳转到它自己的页面，会报错
def change_password(request):
    return render(request, 'change_password.html')


# 局部配置
# 在settings配置文件中，直接配置  LOGIN_URL = '/login/'  


# 如果全局和局部都配置，以局部配置为准
```



## 扩展auth_user表字段

### 利用一对一外键字段关系

```python
from django.contrib.auth.models import User, AbstractUser

class UserDetail(models.Model):
    phone = models.BigIntegerField()
    user = models.OneToOneField(to='User')
```



### 利用继承关系

```python
from django.contrib.auth.models import User, AbstractUser

class UserInfo(AbstractUser):
    phone = models.BigIntegerField()
    register_time = models.DateField(auto_now_add=True)
    
    
# 需注意， 还要在配置文件中配置
AUTH_USER_MODEL = 'app01.UserInfo'    # 应用名.表名
```