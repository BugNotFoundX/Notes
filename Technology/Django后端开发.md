---
title: Django后端开发
mathjax: true
date: 2022-11-15 08:11:05
tags:
categories:
cover:
---

# 〇、关于

- 文章基于数据库大作业HuYoCommunity项目编写
  - 项目连接：TODO

- 项目基于django + vue前后端分离架构编写
- 项目基于apifox进行接口的定义与测试
- 项目数据库选用mysql
- 关于版本
  - python3.10
  - django4.1.2
  - mysql5.7.39
  - mysqlclient2.1.1

# 一、创建项目

- 直接在pycharm上点击创建即可

- 文件目录

  ```
  mysite
  ├── manage.py          【项目的管理，启动项目、创建app、数据管理】【不要动】【***常用***】
  └── mysite
      ├── __init__.py
      ├── settings.py    【项目配置】【***常常修改***】
      ├── urls.py        【URL和函数的对应关系】【***常常修改***】
      ├── asgi.py        【接收网络请求】【不要动】
      └── wsgi.py        【接收网络请求】【不要动】
  ```

# 二、创建APP

- 创建命令

	```shell
	// 在终端输入
	python .\manage.py startapp <app name>
	```


- 文件目录

  ```
  ├── <app name>
  │   ├── __init__.py
  │   ├── admin.py         【固定，不用动】django默认提供了admin后台管理。
  │   ├── apps.py          【固定，不用动】app启动类
  │   ├── migrations       【固定，不用动】数据库变更记录
  │   │   └── __init__.py
  │   ├── models.py        【**重要**】，对数据库操作。
  │   ├── tests.py         【固定，不用动】单元测试
  │   └── views.py         【**重要**】，函数。
  ├── manage.py
  └── mysite
      ├── __init__.py
      ├── asgi.py
      ├── settings.py
      ├── urls.py          【URL->函数】
      └── wsgi.py
  ```

# 三、一般开发流程

- 1、注册app【settings.py】

  ```python
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      '<your app name>.apps.<your app name>Config'
  ]
  ```

- 编写URL和视图函数对应关系 【urls.py】

  ```python
  urlpatterns = [
      path('index/', views.index),
  ]
  ```

- 编写视图函数 【views.py】

  ```python
  def index(request):
      return HttpResponse("hello")
  ```

- 启动django项目

  ```
  python manage.py runserver 
  ```

  或直接点击pycharm的运行按钮

# 四、静态文件

- 在app目录下创建static文件夹

  ```
  ├── <app name>
  │   ├── __init__.py
  │   ├── admin.py         【固定，不用动】django默认提供了admin后台管理。
  │   ├── apps.py          【固定，不用动】app启动类
  │   ├── migrations       【固定，不用动】数据库变更记录
  │   │   └── __init__.py
  |   ├── static           【保存静态文件】
  │   ├── models.py        【**重要**】，对数据库操作。
  │   ├── tests.py         【固定，不用动】单元测试
  │   └── views.py         【**重要**】，函数。
  ├── manage.py
  └── mysite
      ├── __init__.py
      ├── asgi.py
      ├── settings.py
      ├── urls.py          【URL->函数】
      └── wsgi.py
  ```

# 五、数据库操作

- 1、MySQL数据库 + pymysql

  ```
  import pymysql
  
  # 1.连接MySQL
  conn = pymysql.connect(host="127.0.0.1", port=3306, user='root', passwd="123456", charset='utf8', db='unicom')
  cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
  
  # 2.发送指令
  cursor.execute("insert into admin(username,password,mobile) values('张三','123456','18512345678')")
  conn.commit()
  
  # 3.关闭
  cursor.close()
  conn.close()
  ```

- 2、Django开发操作数据库更简单，内部提供了ORM框架

  ![](https://s3.bmp.ovh/imgs/2022/11/15/08bf99ad28e7a18d.png)



## (一)安装第三方模块

- 安装ORM依赖的mysql库

  ```
  pip install mysqlclient
  ```

## (二)ORM

- ORM可以帮助我们做两件事：
  - 创建、修改、删除数据库中的表（不用你写SQL语句）。 【无法创建数据库】
  - 操作表中的数据（不用写SQL语句）。

### 1、创建数据库

- 启动MySQL服务

  ```
  net start mysql
  mysql -uroot -p
  ```

  ```
  PS C:\Users\<usr>\Desktop> mysql -uroot -p
  Enter password: ********
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 7
  Server version: 5.7.39 MySQL Community Server (GPL)
  
  Copyright (c) 2000, 2022, Oracle and/or its affiliates.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql>
  ```

- 创建数据库

  ```
  create database <name> DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
  ```

### 2、django连接数据库

- 在settings.py文件中进行配置和修改。

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': '<database name>',  # 数据库名字
          'USER': 'root',
          'PASSWORD': '123456',
          'HOST': '127.0.0.1',  # 那台机器安装了MySQL
          'PORT': 3306,
      }
  }
  ```

### 3、django操作数据库

#### (1)创建表

- 在models.py文件中

  ```python
  class UsrInfo(models.Model):
      usrname = models.CharField(max_length=32)
      passwd = models.CharField(max_length=16)
      age = models.IntegerField()
      # 还有外键等奇奇怪怪的类，自行探索
  ```

- 执行命令

  ```powershell
  python manage.py makemigrations
  python manage.py migrate
  ```

  注意：app需要提前注册

- 在表中新增列时，由于已存在列中可能已有数据，所以新增列必须要指定新增列对应的数据：

  - 1、手动输入一个值。

  - 2、设置默认值

    ```
    age = models.IntegerField(default=2)
    ```

  - 3、允许为空

    ```
    data = models.IntegerField(null=True, blank=True)
    ```

- 以后在开发中如果想要对表结构进行调整：

  - 在models.py文件中操作类

  - 执行命令

    ```powershell
    python manage.py makemigrations
    python manage.py migrate
    ```


#### (2)操作表

```python
#### 1.新建 ####
Department.objects.create(title="销售部")
Department.objects.create(title="IT部")
Department.objects.create(title="运营部")
UserInfo.objects.create(name="张三", password="123", age=19)
UserInfo.objects.create(name="李四", password="456", age=29)
UserInfo.objects.create(name="王五", password="789")


#### 2.删除 ####
UserInfo.objects.filter(id=3).delete()
Department.objects.all().delete()


#### 3.获取数据 ####
# 3.1 获取符合条件的所有数据
# data_list = [对象,对象,对象]  QuerySet类型
data_list = UserInfo.objects.all()
for obj in data_list:
    print(obj.id, obj.name, obj.password, obj.age)

data_list = UserInfo.objects.filter(id=1)
print(data_list)
# 3.2 获取第一条数据【对象】
row_obj = UserInfo.objects.filter(id=1).first()
print(row_obj.id, row_obj.name, row_obj.password, row_obj.age)


#### 4.更新数据 ####
UserInfo.objects.all().update(password=999)
UserInfo.objects.filter(id=2).update(age=999)
UserInfo.objects.filter(name="张三").update(age=999)
```

#### (3)删除表

- 1、找到models.py文件，直接删除想要删除的数据表的生成代码

- 2、然后将所有调用到该数据表的地方注释或删除，重新运行命令

  ```powershell
  python manage.py makemigrations
  python manage.py migrate
  ```

### 4、高级操作

#### (1)Q查询

- TODO

# 六、用户登录

## (一)cookie和session

### 1、cookie

- Cookie意为“甜饼”，是**由W3C组织提出**，最早由Netscape社区发展的一种机制。目前Cookie已经成为标准，所有的主流浏览器如IE、Netscape、Firefox、Opera等都支持Cookie。
- 由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。怎么办呢？就**给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理**。
- Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

### 2、session

- Session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。
- 如果说**Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。**

### 3、工作原理示意

![](https://s3.bmp.ovh/imgs/2022/11/15/f7c64e6f56ddcbcc.png)

## (二)用户登录

- 返回session_id也就是前端的cookie

  ```python
  def login(request):
      # 必须使用post请求
      if request.method == 'POST':
          usrname = request.POST.get('usrname')
          passwd = request.POST.get('passwd')
          # 去数据库校验用户名和密码是否正确，获取用户对象、None
          usr = models.Usr.objects.filter(usrname=usrname, passwd=passwd).first()
          if not usr:
              return JsonResponse({'valid': False, 'token': ''})
          # 用户名和密码正确
          # 生成session
          # 当执行下面一行代码时,django会自动在django_session表中创建对应的表项
          request.session["info"] = {'id': usr.id, 'name': usr.passwd}
          # session可以保存1天
          request.session.set_expiry(60 * 60 * 24 * 1)
          # 返回session_id作为cookie
          return JsonResponse({'valid': True, 'token': request.session.session_key})
      else:
          return JsonResponse({'valid': False, 'token': ''})
  ```

  ![](https://s3.bmp.ovh/imgs/2022/11/17/7a485e245337da0d.jpg)

## (三)登录校验

- 在其他需要登录才能访问的页面中，都需要加入：

  ```python
  def index(request):
      # 这里事实上是django帮我们进行了一次数据库查询，从django_session中查询登录信息
      info = request.session.get("info")
      if not info:
          return redirect('/login/')
      
      ...
  ```

- 也可以使用中间件实现，见[中间件](#中间件)

## (四)注销

- 事实上消除session即可

  ```python
  def logout(request):
      """ 注销 """
      request.session.clear()
      return ...
  ```
  


# 七、django图片存储

- 1、设置settings.py，在底部添加以下两行。MEDIA_ROOT是项目中保存上传的文件的根目录，MEDIA_URL是要访问存放文件目录时使用的地址。

  ```python
  MEDIA_ROOT = os.path.join(BASE_DIR, 'img')
  MEDIA_URL = '/media/'
  ```

- 2、设置根urls.py

  ```python
  from django.contrib import admin
  from django.urls import path, include
  from django.conf.urls.static import static # 添加本行
  from django.conf import settings # 添加本行
  
  urlpatterns = [
      path('admin/', admin.site.urls),
      ...
  ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)  # 将文件路径添加进来
  ```

- 3、models

  在django的模型中支持ImageField的存储。一定要记得设置upload_to，这里表示图片将被存放在根目录下的哪一个文件夹中，若没有该文件夹则会自动创建。例如，现在代码中设置的图片存储路径是img/avatar

  ```python
  class Usr(models.Model):
      usrname = models.CharField(max_length=32)
      passwd = models.CharField(max_length=16)
      avatar = models.ImageField('头像', upload_to="avatar", null=True) # 使用ImageField upload_to必不可少
  ```

- 4、通过url访问服务器图片文件

  ```
  http://<服务器地址>/media/<图片在服务器中地址>
  ```

# 八、<span id = "中间件">中间件</span>

![](https://s3.uuu.ovh/imgs/2022/11/29/523164700891d9f1.png)

## (一)Django 中间件作用

- 修改请求，即传送到 view 中的 HttpRequest 对象。
- 修改响应，即 view 返回的 HttpResponse 对象。

## (二)定义中间件

- 中间件可以定义四个方法，分别是：

	```python
	process_request(self,request)
	process_view(self, request, view_func, view_args, view_kwargs)
	process_exception(self, request, exception)
	process_response(self, request, response)
	```

- 自定义中间的步骤：

	- 1、在 app 目录下新建一个 py 文件，名字自定义，并在该 py 文件中导入 MiddlewareMixin：

		```python
		from django.utils.deprecation import MiddlewareMixin
		```

	- 2、自定义的中间件类，要继承父类 MiddlewareMixin:

		```python
		from django.utils.deprecation import MiddlewareMixin
		from django.shortcuts import HttpResponse
		
		class M1(MiddlewareMixin):
		    """ 中间件1 """
		    def process_request(self, request):
		        # 如果方法中没有返回值（返回None），继续向后走
		        # 如果有返回值 HttpResponse、render 、redirect
		        print("M1.process_request")
		        return HttpResponse("无权访问")
		
		    def process_response(self, request, response):
		        print("M1.process_response")
		        return response
		
		
		class M2(MiddlewareMixin):
		    """ 中间件2 """
		    def process_request(self, request):
		        print("M2.process_request")
		
		    def process_response(self, request, response):
		        print("M2.process_response")
		        return response
		```

	- 3、在 settings.py 中的 MIDDLEWARE 里注册自定义的中间件类:

		```python
		MIDDLEWARE = [
		    'django.middleware.security.SecurityMiddleware',
		    'django.contrib.sessions.middleware.SessionMiddleware',
		    'django.middleware.common.CommonMiddleware',
		    'django.middleware.csrf.CsrfViewMiddleware',
		    'django.contrib.auth.middleware.AuthenticationMiddleware',
		    'django.contrib.messages.middleware.MessageMiddleware',
		    'django.middleware.clickjacking.XFrameOptionsMiddleware',
		    'app01.middleware.auth.M1',
		    'app01.middleware.auth.M2',
		]
		```

	- 4、在中间件的process_request方法
		- 如果方法中没有返回值（返回None），继续向后走
		- 如果有返回值 HttpResponse、render 、redirect…，则不再继续向后执行。

## (三)中间件实现登录校验

- 仅为一个例子，具体问题具体实现

- 编写中间件

	```python
	from django.utils.deprecation import MiddlewareMixin
	from django.shortcuts import HttpResponse, redirect
	
	
	class AuthMiddleware(MiddlewareMixin):
	
	    def process_request(self, request):
	        # 0.排除那些不需要登录就能访问的页面
	        #   request.path_info 获取当前用户请求的URL /login/
	        if request.path_info == "/login/":
	            return
	
	        # 1.读取当前访问的用户的session信息，如果能读到，说明已登陆过，就可以继续向后走。
	        info_dict = request.session.get("info")
	        print(info_dict)
	        if info_dict:
	            return
	
	        # 2.没有登录过，重新回到登录页面
	        return redirect('/login/')
	```

- 应用中间件

	```python
	MIDDLEWARE = [
	    'django.middleware.security.SecurityMiddleware',
	    'django.contrib.sessions.middleware.SessionMiddleware',
	    'django.middleware.common.CommonMiddleware',
	    'django.middleware.csrf.CsrfViewMiddleware',
	    'django.contrib.auth.middleware.AuthenticationMiddleware',
	    'django.contrib.messages.middleware.MessageMiddleware',
	    'django.middleware.clickjacking.XFrameOptionsMiddleware',
	    'app01.middleware.auth.AuthMiddleware', # 应用中间件
	]
	```

# 九、文件的上传

