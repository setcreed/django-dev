# jwt认证

## admin后台关联自定义用户表

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as AuthUserAdmin
from . import models


class UserAdmin(AuthUserAdmin):
    # 添加用户页面可控制字段
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('username', 'password1', 'password2', 'phone', 'is_staff', 'is_active'),
        }),
    )

    # 用户列表展示页面显示字段
    list_display = ('username', 'email', 'mobile', 'is_staff', 'is_active')

# 注册自定义User表，用admin管理，配置UserAdmin，定制化管理页面
admin.site.register(models.User, AuthUserAdmin)
```





## 用户权限关系  RBAC（Role-BasedAccessControl）	

```
表： User、Group、Permission、UG关系表、UP关系表、GP关系表
传统的RBAC 有两种：权限三表 => 权限五表(没有UP关系表)
Django中Auth组件采用的是  权限六表 （在传统RBAC基础上增加UP关系表）
```
### 权限三表
![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200101155427.png)

### 权限六表

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20191231105924.png)



## 基于后台seesion的token认证

- 未登录状态发送登录请求，提交账号密码数据，后端 对账号密码进行校验
- 后端为当前账号以及当前客户端创建session表，存到session表中
- 服务端做出响应，将session中的认证字符串token传给前端，存到cookie中
- 浏览器 cookie存储服务端返回的token，下一次请求携带着token。服务端接收前台token，并拿着session与user表进行校验。

需要优化的地方：

- 客户端服务端都会存储session相关的token
- 服务端要存token，一定会进行IO操作，存token是IO写操作，服务器并发压力很大

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200101163256.png)



## 基于session的token认证服务器集群

- 客户端访问服务端资源，需要通过Nginx进行分发资源的访问，Nginx能做到负载均衡，将多个客户端访问的请求压力让多台服务器承受，并且Nginx存储静态资源，客户端访问的静态资源不需要从服务端拿，直接从Nginx服务器上拿静态资源，大大降低了服务器的压力。
- 部署多台服务器，一个用来产生token，一个校验token，它们需要数据同步

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200101163419.png)




## jwt认证规则    json  web  token

优点：

- 数据库不需要存储token，所有服务器的IO操作会减少（没有IO写操作）
- 客户端存token，服务器只存储签发与校验算法，执行效率高
- 签发与校验算法在多个服务器可以直接统一，在jwt认证规则下，服务器做集群非常便捷

突破点：

- token必须要有多个部分组成，有能反解的部分，也有不能反解的部分。jwt一般采用三段式
- token中必须包含过期时间，保证token的安全性与时效性



### jwt原理

- jwt由 头.载荷.签名    三部分组成
- 每一个部分都是一个json字典， 头和载荷采用 base64 可逆加密算法加密，签名采用HS256不可逆加密

内容：

- 头（基本信息）：可逆不可逆采用的加密算法、公司名称、项目组信息、开发者信息……
- 载荷（核心信息）：用户主键、用户账号、客户端设备信息、过期时间……
- 签名（安全信息）：头的加密结果、载荷的加密结果、服务器的安全码（盐）……

### 基于jwt的token认证集群

![](https://cdn.jsdelivr.net/gh/setcreed/pic_img/cdn_img/20200101163559.png)



### 签发算法：

- 头内容写死（可以为空{}）：公司、项目组信息都是固定不变的
  - 将数据字典转化为json字符串，再将json字符串加密成base64字符串
- 载荷的内容，用户账号、客户端设备信息是由客户端提供，用户主键是客户端提供账号密码校验User表通过后才能确定，过期时间根据当前时间与配置的过期时长相结合产生。
  - 将数据字典转化为json字符串，再将json字符串加密成base64字符串
- 签名的内容，先将头的加密结果、载荷的加密结果作为成员，再从服务器上拿安全码（不能让任何客户端知道），也可以额外包含载荷的部分（用户信息，设备信息）   
  - 将数据字典转化为json字符串，再将json字符串不可逆加密成SH256字符串
- 将三个字符串用`.` 连接产生三段式token  



### 校验算法：

- 从客户端提交的请求中拿到token，用`.` 分割成三段 （如果不是三段，非法）
- 头（第一段）可以不用解密
- 载荷（第二段）一定要解密，先base64解密成json字符串，再转化成json字典数据
  - 用户主键与用户账号查询User表确定用户是否存在
  - 设备信息用本次请求提交的设备信息比对，确定前后是否是统一设备，决定是否对用户做安全提示，比如短信邮箱提示异地登录
  - 过期时间与当前时间比对，该token是否在有效期内
- 签名（第三段）采用加密碰撞校验，
  - 同样将头、载荷加密字符串和数据安全码形成json字典，转换成json字符串
  - 采用不可逆HS256加密形成加密字符串
  - 新的加密字符串与第三段签名碰撞比对，一致才能确保token是合法的
- 前面的算法都通过后，载荷校验得到的User对象，就是该token代表的登录用户，django项目一般把登录用户存放在`request.user`中



### 刷新算法：

- 要在签发token的载荷中，额外添加两个时间信息：第一次签发token的时间、最多往后刷新的有效时间。
- 每一次请求携带token，不仅走校验算法验证token是否合法，还要额外请求刷新token的接口，完成token的刷新：校验规则与校验算法差不多，但是要将过期时间后移（没有超过有效时间，产生新token给客户端，如果超过了，刷新失败）
- 所以服务器不仅要配置过期时间，还要配置最长刷新时间



## drf-jwt  插件简单使用

安装插件`pip3 install djangorestframework-jwt`

在settings文件中自定义配置jwt

```python
# drf-jwt自定义配置
import datetime
JWT_AUTH = {
    # 过期时间
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),

    # 是否允许刷新
    'JWT_ALLOW_REFRESH': False,

    # 最大刷新的过期时间
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),
}
```



在路由中设置

```python
# urls.py
from django.conf.urls import url, include
from .router import router
# router.register('users', UserModelViewSet, basename='user')
from rest_framework_jwt.views import ObtainJSONWebToken, RefreshJSONWebToken, VerifyJSONWebToken
urlpatterns = [
    # 登录接口，签发token
    url(r'^login/$', ObtainJSONWebToken.as_view()),
    # 刷新token
    url(r'^refresh/$', RefreshJSONWebToken.as_view()),
    # 验证token
    url(r'^verify/$', VerifyJSONWebToken.as_view()),

    url('', include(router.urls))
]

```