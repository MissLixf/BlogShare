## Django分库

### 自动配置

配置多个MySQL数据库连接

```python
DATABASES = {
    'default': {},
    'auth_db': {
        'NAME': 'auth_db',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'swordfish',
    },
    'primary': {
        'NAME': 'primary',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'spam',
    },
    'replica1': {
        'NAME': 'replica1',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'eggs',
    },
    'replica2': {
        'NAME': 'replica2',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'bacon',
    },
}
```

为了选择数据库，需要重写Router

将`auth`应用程序的查询发送到的路由器`auth_db`

```python
class AuthRouter:

    def db_for_read(self, model, **hints):

        if model._meta.app_label == 'auth':
            return 'auth_db'
        return None

    def db_for_write(self, model, **hints):

        if model._meta.app_label == 'auth':
            return 'auth_db'
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """
        如果涉及auth应用程序中的模型，则允许关连。
        """
        if obj1._meta.app_label == 'auth' or \
           obj2._meta.app_label == 'auth':
           return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        Make sure the auth app only appears in the 'auth_db'
        database.
        """
        if app_label == 'auth':
            return db == 'auth_db'
        return None
```

一个路由器将所有其他应用程序发送到主/副本配置，并随机选择一个副本来读取

```python
import random

class PrimaryReplicaRouter:
    def db_for_read(self, model, **hints):

        return random.choice(['replica1', 'replica2'])

    def db_for_write(self, model, **hints):

        return 'primary'

    def allow_relation(self, obj1, obj2, **hints):

        db_list = ('primary', 'replica1', 'replica2')
        if obj1._state.db in db_list and obj2._state.db in db_list:
            return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        All non-auth models end up in this pool.
        """
        return True
```

在设置文件中，我们添加以下内容

```python
DATABASE_ROUTERS = ['AuthRouter', 'PrimaryReplicaRouter']
```

处理路由器的顺序非常重要。将按照DATABASE_ROUTERS设置中列出的顺序查询路由器 。

###  手动选择

在链式queryset中使用using()

```python
Author.objects.using('default').all()
Author.objects.using('other').all()

p = Person(name='Fred')
p.save(using='first')
p.save(using='second')
```



## 分表

数据库中的表是和model中的tb_name对应的。也就是说正常情况下django是一个表和一个model对应的。现在想要让一个model对应所有的表是一件困难的事。让代码层来接管class中的tb_name属性。于是我们要用到的就是python中有关元类的知识。

```python
class NewDynamicModel(object):
    _instance = dict()

    def __new__(cls, base_cls, tb_name):
        """
        创建类根据表名
        :param base_cls: 类名(这个类要models基类)
        :param tb_name: 表名
        :return: base_cls类的实例
        """
        new_cls_name = "%s_To_%s" % (base_cls.__name__, '_'.join(map(lambda x: x.capitalize(), tb_name.split('_'))))

        if new_cls_name not in cls._instance:
            new_meta_cls = base_cls.Meta
            new_meta_cls.db_table = tb_name
            model_cls = type(str(new_cls_name), (base_cls,),
                             {'__tablename__': tb_name, 'Meta': new_meta_cls, '__module__': cls.__module__})
            cls._instance[new_cls_name] = model_cls
        return cls._instance[new_cls_name]
```

`new_meta_cls.db_table = tb_name`

这句代码中控制了我们具体的model中的tb_table从而控制新生成的class对应那张表的问题