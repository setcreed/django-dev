# Django实现读写分离

## 手动实现

**settings.py中配置数据库**

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },
    'db1': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

在视图层views.py

```python
def home(request):
    # 写数据，在主库中写
    user_obj = models.User.objects.create(name="reese").using("default")
    # 查数据，去从库中查
    user_query = models.User.objects.all().using("db1")
```



## 自动实现

**项目路径下创建db_router.py**

```python
# db_router.py
class Router1:
    def db_for_read(self, model, **hints):
                return 'db1'

    def db_for_write(self, model, **hints):
                return 'default'
```

**settings.py中配置数据库**

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },
    'db1': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DATABASE_ROUTERS = ['db_router.Router1',]
```

这样配置后，只要是写的操作，都到default上，只要是读的操作，都到db1上了