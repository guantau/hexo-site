---
title: Django Rest Framework指南
date: 2017-09-17 15:41:55
categories:
tags:
---

# 前言
Web开发采用`RESTFUL`方式越来越流行，且体现出很好的灵活性。 [理解RESTFUL架构](http://www.ruanyifeng.com/blog/2011/09/restful)对`RESTFUL`的原理做了很详细的介绍。

[Django REST Framework](http://www.django-rest-framework.org/)，简称DRF，是Django上用来实现REST API的利器。

在熟悉DRF的基础上，[Django REST Framework的各种技巧](https://segmentfault.com/a/1190000004401112)给出了一些使用技巧，总结的比较到位。

[Django REST Swagger](https://github.com/marcgibbons/django-rest-swagger)用来对django-restframework的接口提供文档说明

# 基本使用

## 安装
通过pip进行安装：

```bash
$ pip install djangorestframework
$ pip install markdown      # 用来书写可浏览API文档
$ pip install django-filter # 用来过滤
```

## 设置
在设置`INSTALLED_APPS`中添加`rest_framework`后就可以使用了：

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```

全局设置都是通过`settings.py`中`REST_FRAMEWORK`来完成的。

控制用户权限控制：

```python
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}
```

在开发的时候，通常采用`BrowsableAPI`渲染方式：

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    )
}
```

采用`BrowsableAPI`渲染时，通常会需要用户登录，那么需要在`urls.py`中添加：

```python
urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

# 入门

使用DRF时，需要考虑以下几个问题：
1. 输入输出接口：对应于`Serializer`的设计和实现
2. 统一参数校验：对接口参数做统一的校验
3. 统一认证方式：不同的用户具有不同的API使用权限
4. 异常处理方式：当接口流程中出现异常时的处理方式

## 请求与响应

请求与响应都需要进行序列化进行传输，序列化用来完成数据的接口，包括几类数据：
1. 模型数据：数据表中包含的字段
2. 外键数据：数据表中通过外键关联的数据
3. 其它数据：需要额外构造的数据

### 序列化
序列化完成的是复杂数据结构和传输数据结构的相互转化。其中，复杂数据结构方便在程序中进行使用，比如数据模型等；传输数据结构方便进行网络传输，比如json、xml。通常，序列化指的是将复杂数据结构转化为传输数据结构的过程（读过程）；而反序列化指的是将传输数据结构转化为复杂数据结构的过程（写过程），同时也包括数据校验。在DRF中，序列化通过类`Serializer`来实现，还有专门针对数据模型的类`ModelSerializer`。

详细的文档可以参考[Serializer](http://www.django-rest-framework.org/api-guide/serializers/)

#### 关于参数检查
对单个字段分别进行检查和校验，只需要添加`validate_<field_name>`方法，并返回检查后的数据或者抛出异常`serializers.ValidationError`，比如：

```python
from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
```

为了方便，在对单个字段进行检查时，使用`Validator`，比如：

```python
def multiple_of_ten(value):
    if value % 10 != 0:
        raise serializers.ValidationError('Not a multiple of ten')

class GameRecord(serializers.Serializer):
    score = IntegerField(validators=[multiple_of_ten])
    ...
```


如果需要对多个字段进行检查时，只需要修改`validate`方法即可，比如：

```python
from rest_framework import serializers

class EventSerializer(serializers.Serializer):
    description = serializers.CharField(max_length=100)
    start = serializers.DateTimeField()
    finish = serializers.DateTimeField()

    def validate(self, data):
        """
        Check that the start is before the stop.
        """
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data
```

#### 关于外键数据
可以将`Serializer`内嵌为`Field`，可设置的属性包括`required=False`（允许为`None`）以及`Many=True`（为`list`类型），比如：

```python
class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    username = serializers.CharField(max_length=100)

class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)
    edits = EditItemSerializer(many=True)  # A nested list of 'edit' items.
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

外键直接可以引用其他的serializer,例如group，可以看到：

* response中group是嵌套的
* 外键的属性可以使用source，例如phone
* 不在原来model上的东西使用SerializerMethodField（或者在model上但是你要对这个值做一些特殊处理)

```python
class GroupSerializer(serializers.ModelSerializer):

    class Meta:
        model = Group
        fields = ('id', 'name')

class UserSerializer(serializers.ModelSerializer):
    groups = GroupSerializer(many=True)
    phone = serializers.CharField(source='profile.phone', read_only=True)
    name = serializers.CharField(source='profile.name', read_only=True)
    menus = serializers.SerializerMethodField()
    is_active = serializers.BooleanField(source='profile.is_cms_active')

    def get_menus(self, user):
        return get_menus(user)

    class Meta:
        model = User
        fields = ('id', 'username', 'name', 'email', 'phone', 'groups', 'menus', 'is_active')
```

#### 关于ModelSerializer
相比于`Serializer`，`ModelSerializer`与某个数据模型对应，并做了如下工作：

1. 根据数据模型创建相应的`Field`
2. 创建`validator`，包括`unique_together`的检查
3. 实现了简单的`.create()`和`.update()`

外键可以采用三种方式进行序列化：

1. 采用超链接（hyperlink）
2. 采用内联（nested representation）
3. 采用自定义（custom representation）

#### 关于HyperlinkedModelSerializer
相比于`ModelSerializer`，`HyperlinkedModelSerializer`是采用超链接的方式来处理关联的。


### 请求
DRF的`Request`对象扩展了Django中的`HttpRequest`类，添加了灵活的解析和认证方法。

### 响应


## Views
### Generic Views
用于快速构建接口，在`APIView`的基础上提供了许多封装好的View和Mixin。

### ViewSet
通过在router中注册Viewset，会相应的生成URL地址

### Router
SimpleRouter仅仅对list, create, retrieve, update, partial_update, destroy这几类操作进行了路由，其它的则需要通过加修饰符`@detail_route`或者`@list_route`来进行路由。

