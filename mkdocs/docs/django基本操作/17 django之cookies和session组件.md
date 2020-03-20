# django之cookies和session组件

## cookie与session

### cookie介绍

HTTP协议 是无状态的，每次请求连接都是不保存客户端状态的，cookie就是用来保存客户端状态的。试想一下，如果每次登录一个网站，每次跳转页面都不会记录我的信息，都要求重新输入密码，是不是很不爽？

Cookie具体指的是一段小信息，它是服务器发送出来存储在浏览器上的一组组键值对，下次访问服务器时浏览器会自动携带这些键值对，以便服务器提取有用信息。  简单来说，**就是保存在客户端浏览器的键值对。**

当你登录一个网站后，服务端会把你的用户名、密码等信息发到你的浏览器上保存起来，下次再访问请求的时候，浏览器会自动带上这些cookie，服务器就会认识。

### session介绍

上述的cookie本身保存在客户端，容易被拦截，不是很安全，所以就有了session。 浏览器登录网站成功后，服务器会根据某种算法生成一个随机字符串，发给浏览器，浏览器保存在本地。服务器记录了随机字符串与用户信息的对应关系，之后浏览器 再来访问，就带着字符串就来了，然后服务器识别这个字符串比对 对应关系，如果有这个关系，就知道是你来了。



### token

session虽然相对安全，数据保存在服务端，一个用户保存一份，多个用户保存多份，一旦用户量非常大，会占用服务端大量的资源，占用服务器硬盘空间。

所以就有了token，先提前规定好加密算法。当用户来的时候，比如说username，就会用加密算法对username生成一个随机字符串，然后把username+随机字符串发送给浏览器保存。当这个用户再来的时候，服务器会拿到username再去用加密算法加密，得到的随机字符串与保存在浏览器本地的字符串做比对，判断是否是合法用户。

## django操作cookie

### 设置cookie

```python
obj = HttpResponse()   # 利用obj对象才可以操作cookie
return obj


obj.set_cookie('k1', 'v1')    # 告诉浏览器设置键值对
```

设置cookie过期时间：

```python
obj.set_cookie('k1','v1',max_age=3)
obj.set_cookie('k1','v1',expires=3)    # expires参数 是为IE浏览器准备用的
```

cookie超时时间是以秒  作为单位的

一些参数：

- key, 键
- value='', 值
- max_age=None, 超时时间
- expires=None, 超时时间(IE requires expires, so set it if hasn't been already.)
- path='/', Cookie生效的路径，/ 表示根路径，特殊的：根路径的cookie可以被任何url的页面访问
- domain=None, Cookie生效的域名
- secure=False, https传输
- httponly=False 只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖

### 获取cookie

```python
request.COOKIES.get('k1')   # 获取浏览器携带的cookie值
```

简单的测试：

```python
from django.shortcuts import render, HttpResponse, redirect

# Create your views here.

def login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        if username == 'cwz' and password =='123':
            obj = redirect('/home/')
            obj.set_cookie('whoami', 'cwz')
            return obj
    return render(request, 'login.html')


def home(request):
    if request.COOKIES.get('whoami'):
        return HttpResponse('只有登录成功才能到这儿')

    return redirect('/login/')
```

### 删除cookie

用delete_cookie

```python
def logout(request):
    obj = redirect('/login/')
    obj.delete_cookie('whoami')
    return obj
```



### 基于cookie实现的登录认证

```python
request.get_full_path()   # 获取url以及get携带的参数
request.path_info          # 获取用户输入的url
```



```python
from django.shortcuts import render, HttpResponse, redirect


# Create your views here.

def login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        if username == 'cwz' and password == '123':

            old_path = request.GET.get('next')
            if old_path:
                obj = redirect(old_path)

            else:
                obj = redirect('/home/')
            obj.set_cookie('whoami', 'cwz')
            return obj
    return render(request, 'login.html')


from functools import wraps


def auth_login(func):
    @wraps(func)
    def inner(request, *args, **kwargs):
        res = func(request, *args, **kwargs)
        if request.COOKIES.get('whoami'):
            # print(request.get_full_path())
            # print(request.path_info)
            return res
        else:
            current_path = request.path_info
            return redirect('/login/?next=%s' % current_path)  # 没有登录的用户，跳转到登录页面

    return inner


@auth_login
def home(request):
    if request.COOKIES.get('whoami'):
        return HttpResponse('只有登录成功才能到这儿')

    return redirect('/login/')


@auth_login
def index(request):
    return HttpResponse('index页面，登录之后才能访问')


@auth_login
def reg(request):
    return HttpResponse('reg页面，登录之后才能访问')


@auth_login
def logout(request):
    obj = redirect('/login/')
    obj.delete_cookie('whoami')
    return obj
```

## django操作session

### django中session方法

```python
# 获取、设置、删除Session中数据
request.session['k1']
request.session.get('k1',None)
request.session['k1'] = 123
request.session.setdefault('k1',123) # 存在则不设置
del request.session['k1']


# 所有 键、值、键值对
request.session.keys()
request.session.values()
request.session.items()
request.session.iterkeys()
request.session.itervalues()
request.session.iteritems()

# 会话session的key
request.session.session_key

# 将所有Session失效日期小于当前日期的数据删除
request.session.clear_expired()

# 检查会话session的key在数据库中是否存在
request.session.exists("session_key")

# 删除当前会话的所有Session数据
request.session.delete()
　　
# 删除当前的会话数据并删除会话的Cookie。
request.session.flush() 
    这用于确保前面的会话数据不可以再次被用户的浏览器访问
    例如，django.contrib.auth.logout() 函数中就会调用它。

# 设置会话Session和Cookie的超时时间
request.session.set_expiry(value)
    * 如果value是个整数，session会在些秒数后失效。
    * 如果value是个datatime或timedelta，session就会在这个时间后失效。
    * 如果value是0,用户关闭浏览器session就会失效。
    * 如果value是None,session会依赖全局session失效策略。
```



注意事项：

- **由于session是保存在服务器数据库的，所以django在操作session时要先执行数据库迁移命令， 生成django_session表**

- **django 默认的session失效时间是14天  **

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151156.png)



### 设置session

```python
request.session['k1'] = 'v1'
```

- django内部自动调用算法生成一个随机字符串
- 在django_session添加数据， 数据也是加密处理
- 将产生的随机字符串返回给客户端，让浏览器保存， 字符串的形式为 sessionid : 随机字符串

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151240.png)



### 获取session值

```python
def get_session(request):
    res = request.session.get('k1')
    print(res)
    return HttpResponse('获取成功')
```

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200305151317.png)

django内部处理的事情

- django内部会自动去请求头里面获取cookie
- 拿到sessionid所对应的随机字符串去django_session表中比对
- 如果比对上了，会将随机字符串对应的值取出来，放在request.session中。



### 删除session

```python
request.session.delete()    # 客户端与服务端的session都会删除

request.session.flush()     # 建议使用
```

删除session只会根据浏览器的不同删出对应的数据

### 设置超时时间

```python
request.session.set_expiry(value)
# 如果value是个整数，session会在些秒数后失效。
# 如果value是个datatime或timedelta，session就会在这个时间后失效。
# 如果value是0,用户关闭浏览器session就会失效。
# 如果value是None,session会依赖全局session失效策略。
```



### django中session配置

Django中默认支持Session，其内部提供了5种类型的Session。

```python
1. 数据库Session
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

2. 缓存Session
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

3. 文件Session
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 

4. 缓存+数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

5. 加密Cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

其他公用设置项：
SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）

Django中Session相关设置
```