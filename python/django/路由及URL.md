在写项目的时候，会有多个APP，如果都使用项目文件夹下的urls文件的话，文件会非常赘余，所以常会使用多级路由。

在APP中创建urls.py文件，作为子路由，定义路由

```python
from django.contrib import admin
from django.urls import path,include

from . import views

urlpatterns = [
    path('',views.index),
]
```

这样就创建了一个子路由。这里需要注意，所有的路由只能解析urlpatterns。

在根路由上注册子路由：

```python
from django.contrib import admin
from django.urls import path,include



urlpatterns = [
    path('APP/',include('APP.urls')),
    path('admin/', admin.site.urls),
]

```

这样就将子路由包含进了根路由。访问localhost:8000/APP就可以访问到页面了。

同理可得，若业务过于繁杂，那就使用多级路由。但是等级过多也会引起很多不必要的问题。

# URL解析

在通常情况下，项目都是做了前后分离的，前后分离的主要工具就是request和response。这两种方式的常用方法有GET和POST，GET和POST的功能是相同的，但是在用法上有所不同。

创建GET方法接受类：

```python
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.
def add(request):
    a = request.GET['a']
    b = request.GET['b']
    c = a + b
    return HttpResponse(c)

```

这里使用了request.GET方法接收了request的内容。通过浏览器访问，传入request：


![2.png](.\img\2.png)

还可以用另一种方式传入参数：

在urls中创建request请求格式：
```
urlpatterns = [
    path('test/<int:a>/<int:b>',APP_views.add),
    #path('test/',APP_views.add),
    path('admin/', admin.site.urls),
]
```

在写views的时候，写一个形式参数，和urls中的参数同名。

