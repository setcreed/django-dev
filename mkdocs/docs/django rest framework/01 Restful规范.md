# Restful规范

## Web API接口

### 什么是接口

规定了提交参数的请求方式，访问  其可以获取响应的反馈数据的url链接

包含了四部分： url链接 + 请求方式 + 请求参数 + 响应数据

- url: 长得像返回数据的url链接      `https://api.map.baidu.com/place/v2/search`

- 请求方式:   get   post    put   patch    delete

- 请求参数: json或xml格式的key-value类型数据

  - ak: 124524542EJbhbuH899
  - regin: 上海
  - query: 大本营
  - output: json

- 响应结果: json或xml格式的数据

  - 上方请求参数的output参数值决定了响应数据的格式


```json
{
    "status": 0,
    "message": "ok",
    "results": [
        {
            "name": "大本营",
            "location": {
                "lat": 31.45346,
                "lng": 146.23423
            },
            "address": "本环路123号",
            "province": "上海市",
        }
        ......
    ]
}
```



### 接口文档的编写:YApi

[YApi](http://yapi.demo.qunar.com/)是去哪儿网前段计数中心的一个开源可视化  接口管理平台. 

详情见[官方文档](https://hellosean1025.github.io/yapi/)

### 接口测试工具:  Postman

Postman是一款接口调试工具,   支持多操作系统平台,  是测试接口的首选工具

可以在[官网](https://www.getpostman.com/)下载使用

推荐Postman的开源代替品:    [Postwoman](https://github.com/liyasthomas/postwoman)

## Restful接口规范

[RESTful](http://www.ruanyifeng.com/blog/2011/09/restful.html) 是目前最流行的 API 设计规范，用于 Web 数据接口的设计。

### url设计

#### 1、保障数据安全

- 接口都是操作前后端数据的,   为了保证数据的安全,  采用https协议

#### 2、接口特征表现

- 接口用来操作数据,  与网址有区别,   所有用特定的关键字表示接口


```
https://api.baidu.com
https://www.baidu.com/api
```



#### 3、多版本资源共存

- 如果一个资源存在多版本结果,   在url链接中要用特定符号来兼容多版本共存

```
https://api.baidu.com/v1/books/
https://api.baidu.com/v2/books/
```



#### 4、数据就是资源

接口操作的数据称之为资源,   在url中体现 资源的名称,   不能体现操作资源的动词,  错误示范:`https://api.baidu.com/get_books`

- 常规资源接口


```
https://api.baidu.com/books/
https://api.baidu.com/books/(pk)/
```



- 非常规接口     和某资源不是特别密切或是不止一种资源

```
https://api.baidu.com/login/
https://api.baidu.com/place/search/
```



#### 5、群资源操作

一般还有额外的限制条件,   如排序、限制调试、分页等

```
https://api.baidu.com/v1/books/?ordering=-price&limit=3

这让人就知道  搜索价格最贵的前三本书
```



#### 6、资源操作由请求方式决定

- get  获取单个或多个资源

```
https://api.baidu.com/books/        群查，返回多个结果对象
https://api.baidu.com/books/(pk)/   单查，返回单个结果对象
```



- post    新增单个或多个资源


```
https://api.baidu.com/books/

单增  提交单个数据字典, 完成单增,  返回单个结果对象
群增   提供多个数据字典的数组, 完成群增, 返回多个结果对象
```



- put   整体修改单个或多个资源

```
https://api.baidu.com/books/
整体修改多个, 提供多个数据字典的数组(数据字典中包含主键), 完成整体多个修改,返回对个结果对象
```



```
https://api.baidu.com/books/(pk)/
整体修改单个, 提供单个数据字典(主键在url中体现),  完成整体单个修改, 返回单个结果对象s
```




- patch  局部修改单个或多个资源
  
  方式与put相同, 不同的是  操作的资源如果有5个key-value键值对, put请求提供的字典必须全包含,  但是patch提供的字典包含的键值对0-5个都可以    ，一般用patch
  
- delete   删除单个或多个资源

```
https://api.baidu.com/books/
多删, 提供多个资源主键数据,  完成群删, 不做任何资源的返回,   一般返回的结果就是: 成功或失败
```



```
https://api.baidu.com/books/(pk)/

单删, 不需要提供额外数据,  完成单删,  不做资源的返回
```





### 响应结果

#### 1、响应对象中要包含网络状态码(网络状态信息和网络状态码捆绑出现, 不要额外设置)

- 1xx:  基本信息
- 2xx  成功  
  - 200   常规请求 成功
  - 201  创建成功
- 3xx   重定向
- 4xx  客户端 错误
  - 400 错误请求
  - 403 请求无权限
  - 404 请求资源不存在
- 5xx  服务器错误   500

#### 2、数据状态码  (一般是前后端约定规则)

如: 

```
0:  成功
1:  失败    1xx 具体失败信息 要在接口文档中明确写出
2:  无数据   2xx  具体无数据信息
```

#### 3、数据状态信息

一般不仅仅是对数据状态码的解释,  更多的是对结果的描述,  给前端开发者阅读

```json
{
    "status": 0,
    "message": "ok",
    "results": [
        {
            "name": "大本营",
            "location": {
                "lat": 31.45346,
                "lng": 146.23423
            },
            "address": "本环路123号",
            "province": "上海市",
        }
        ......
    ]
}
```

#### 4、数据结果

一般是数组  \ 字典形式,  如果有子资源(图片  视频  音频),  返回资源的url链接

```json
{
    "status": 0,
    "msg": "ok",
    "results": [
        {
            "name": "小王子",
            "img": "https://api.baidu.com/media/books/1.jpg"
        }
    ]
}
```

