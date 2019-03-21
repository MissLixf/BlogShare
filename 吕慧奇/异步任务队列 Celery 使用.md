#   异步任务队列 Celery 使用

### 一、异步任务：

* 异步任务是web开发中一个很常见的方法。对于`一些耗时耗资源的操作`，往往`从主应用中隔离`，通过异步的方式执行。
  * 简而言之，做一个注册的功能，在用户使用邮箱注册成功之后，需要给该邮箱发送一封激活邮件。如果直接放在应用中，则调用发邮件的过程会`遇到网络IO的阻塞`，比好优雅的方式则是使用异步任务，应用在业务逻辑中触发一个异步任务。

### 二、Celery简单介绍：

* `Celery是一个异步任务的调度工具，也是处理大量消息的分布式系统`。它是`Python写的库`，但是它实现的通讯协议也可以使用ruby，php，javascript等调用。`异步任务除了消息队列的后台执行的方式，还是一种则是跟进时间的计划任务`。

*  图解：

  ![1551882077439](C:\Users\lh9\Desktop\各种DOM\celery\01.png)

    * `生产者`(生产任务) →  把任务放到`任务队列` → `消费者`监听任务队列是否有可执行的任务。

### 三、Celery 使用场景：

* **异步任务**：将`耗时操作`任务提交给Celery去异步执行，比如发送短信，邮件，消息推送，音视频处理等。
* **定时任务**：类似于Linux系统crontab，比如每天定时的去统计一些数据。

### 四、Celery 安装：

#### 4.1：首先安装Python的环境：

##### 一、使用virtualenv：

* 安装virtualenv： `pip install virtualenv`

* 创建virtualenv环境名称：` virtualenv env_cow[虚拟环境名称]`

* 激活创建的虚拟环境：`source env_cow[新创建虚拟环境名称]/bin/activate`

  ```json
  # 可以看到出现个前缀虚拟环境名称
  (env_cow) [root@localhost /]# 
  ```

* 退出虚拟环境：`deactivate`

##### 二、使用virtualenvwrapper：

* 安装virtualenvwrapper：`pip install virtualenvwrapper`

* 找到virtualenvwrapper.sh 文件所在位置：`whereis virtualenvwrapper.sh`

* **安装zsh**：

  1.输入`cat /etc/shells`命令来查看本地安装的shell。

  ```json
  /bin/sh
  /bin/bash
  /sbin/nologin
  /usr/bin/sh
  /usr/bin/bash
  /usr/sbin/nologin
  # 发现并没有zsh，我们使用yum来安装它
  ```

  2.安装zsh：输入`yum -y install zsh` ，等待下载完成。

  3.再次输入`cat /etc/shells`，发现zsh已经安装好 。

  ```json
  /bin/sh
  /bin/bash
  /sbin/nologin
  /usr/bin/sh
  /usr/bin/bash
  /usr/sbin/nologin
  /bin/zsh
  ```

  4.替换默认的shell：`chsh -s /bin/zsh`切换sehll至zsh。

  ```json
  [root@localhost /]# chsh -s /bin/zsh
  Changing shell for root.
  Shell changed.
  ```

  5.`reboot`重启后，查看当前使用的shell，已经改成了zsh。

     输入`echo $SHELL `：

  ```json
  [root@localhost /]# echo $SHELL
  /bin/bash
  ```

* **安装oh-my-zsh**：

  一、手动安装：

  ​	1.安装git：`yum -y install git `

  ​	2.克隆oh-my-zsh：git clone [git://github.com/robbyrussell/oh-my-zsh.git](https://link.jianshu.com/?t=git://github.com/robbyrussell/oh-my-zsh.git) ~/.oh-my-zsh 

  ​	3.复制zshrc：`cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc`

  二、自动安装：

  ​	1.curl -L [https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh](https://link.jianshu.com/?t=https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh) | sh 

  	使用手动或者自动安装，完成后重启。oh-my-zsh就生效了。

* 下载完oh-my-zsh后，发现多了`~/.zshrc`文件。

* 编辑`~/.zshrc`，把`virtualenvwrapper.sh绝对路径`添加该文件中：

  ```json
  # If you come from bash you might have to change your $PATH.
  # export PATH=$HOME/bin:/usr/local/bin:$PATH
  
  # Path to your oh-my-zsh installation.
  export ZSH=$HOME/.oh-my-zsh
  source /usr/bin/virtualenvwrapper.sh  # 添加进来
  
  # Set name of the theme to load --- if set to "random", it will
  # load a random theme each time oh-my-zsh is loaded, in which case,
  # to know which specific one was loaded, run: echo $RANDOM_THEME
  # See https://github.com/robbyrussell/oh-my-zsh/wiki/Themes
  ZSH_THEME="robbyrussell"
  ```

* 启动.zshrc文件：`source ~/.zshrc`

  ```json
  ➜  / source ~/.zshrc 
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/initialize
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/premkvirtualenv
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/postmkvirtualenv
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/prermvirtualenv
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/postrmvirtualenv
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/predeactivate
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/postdeactivate
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/preactivate
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/postactivate
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/get_env_details
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/premkproject
  virtualenvwrapper.user_scripts creating /root/.virtualenvs/postmkproject
  ```

* 使用virtualenvwrapper创建虚拟环境：`mkvirtualenv env_cow[虚拟环境名称]`

  ```json
  (env_cow) ➜  /   # 可以看到前缀名的变化，代表已经进入了虚拟环境
  ```

* 退出虚拟环境：`deactivate`

* 激活虚拟环境：`workon env_cow[虚拟环境名称]`

* 列出所有的虚拟环境：`workon`

* 查看虚拟环境安装哪些包(进入虚拟环境)：`pip list`

  ```json
  Package    Version
  ---------- -------
  amqp       2.4.2  
  billiard   3.5.0.5
  celery     4.2.1  
  kombu      4.4.0  
  pip        19.0.3 
  pytz       2018.9 
  redis      3.2.0  
  setuptools 40.8.0 
  vine       1.2.0  
  wheel      0.33.1 
  ```

##### 三、使用pyenv(Python全局版本)：

* 下载安装：`curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash`

* 下载完，打开`vim ~/.zshrc`：

  ```json
  export PATH=$HOME/.pyenv/bin:$PATH
  eval "$(pyenv init -)"
  eval "$(pyenv virtualenv-init -)"
  ```

* 启动.zshrc文件：`source ~/.zshrc`

* 查看本机安装Pyenv的版本：`pyenv versions`

* 查看可安装的版本：`pyenv   install -l`

* 安装指定版本：`pyenv  install  3.6.4`

* 卸载指定版本：`pyenv uninstall  3.6.4`

* 通过shell切换Python 版本：`pyenv shell  3.6.4`

* 查看安装的pyenv所在的插件：`ls -la  ~/.pyenv/plugins`

* 列出当前的虚拟环境：`pyenv virtualenvs`

* 激活一个虚拟环境：`pyenv  activate  env_cow[虚拟环境]`

* 创建一个虚拟环境并指定该Python版本：`pyenv  virtualenv  2.7.4[指定的python版本,已经安装过]  env_cow[虚拟环境名]`

#### 4.2：安装celery/Redis：

* 安装celery：`pip install celery[redis]`

* 安装Redis：

  ```json
  $ wget http://download.redis.io/releases/redis-5.0.3.tar.gz
  $ tar xzf redis-5.0.3.tar.gz
  $ cd redis-5.0.3
  $ make
  ```

### 五、Celery异步任务：

* 写一个**不使用异步**的文件(文件名app.py)。

  ```json
  # -*- coding: utf-8 -*-
  
  import time
  
  def add(x, y):
      print 'enter call func...'
      time.sleep(6)   #  在这里停留了6s才会往下执行
      return x + y
  
  if __name__ == '__main__':
      print 'start task...'
      result = add(2, 8)
      print 'end task...'
      print result
  ```

  ```json
  (env_cow) ➜  test_celery python app.py   # 执行命令查看执行结果及顺序
  start task...
  enter call func...
  # 在这里停留了10s 才会继续往下执行 
  end task...
  10 
  ```

* 写一个**使用异步**的文件(文件名tasks.py)。

  ```json
  # -*- coding: utf-8 -*-
  
  import time 
  from celery import Celery
  
  # 消息中间件, 使用的redis
  broker = 'redis://localhost:6379/1'
  
  # 存储任务的执行结果
  backend = 'redis://localhost:6379/2'
  
  # Celery 使用时区, 不指定默认就是UTC
  CELERY_TIMEZONE = 'Asia/Shanghai'
  
  # 实例化Celery
  app = Celery('my_task', broker=broker, backend=backend)
  
  # 添加装饰器@app.task 这样就把add这个函数变成异步执行
  @app.task
  def add(x, y):
      print 'enter call func...'
      time.sleep(4)
      return x + y
  
  if __name__ == '__main__':
      print 'start task...'
      # 调用该函数时, 使用delay该方法执行异步的调用，也可以使用apply_async()
      result = add.delay(2, 8)
      print 'end task...'
      print result
  ```

  ```json
  # 执行该文件, 会发现 执行函数没有等待10s，而是异步的处理
  (env_cow) ➜  test_celery python tasks.py 
  start task...
  end task...
  35a60e25-82f5-4c5b-a90e-799ef3218d77  # 发现结果并不是我们想要的，因为我们没有启动worker，只是把异步任务发过去了
  ```

* 启动worker：`celery worker  -A  tasks  -l  INFO`

  * -A：指定celery 实例的位置。
  * -l：指定日志的级别。

  ```json
  (env_cow) ➜  test_celery celery worker -A tasks -l INFO
  /root/.virtualenvs/env_cow/lib/python2.7/site-packages/celery/platforms.py:796: RuntimeWarning: You're running the worker with superuser privileges: this is
  absolutely not recommended!
  
  Please specify a different user using the --uid option.
  
  User information: uid=0 euid=0 gid=0 egid=0
  
    uid=uid, euid=euid, gid=gid, egid=egid,
   
   -------------- celery@localhost.localdomain v4.2.1 (windowlicker)
  ---- **** ----- 
  --- * ***  * -- Linux-3.10.0-862.el7.x86_64-x86_64-with-centos-7.5.1804-Core 2019-03-10 11:08:37
  -- * - **** --- 
  - ** ---------- [config]
  - ** ---------- .> app:         tasks:0x7f7f80269a10  # app实例
  - ** ---------- .> transport:   redis://localhost:6379/1  # 中间人
  - ** ---------- .> results:     redis://localhost:6379/2  # 存储任务执行接口
  - *** --- * --- .> concurrency: 1 (prefork) # 并发量：1
  -- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
  --- ***** ----- 
   -------------- [queues]  # 消息队列
                  .> celery           exchange=celery(direct) key=celery
                  
  
  [tasks]  #  执行任务
    . tasks.add
  
  [2019-03-10 11:08:37,884: INFO/MainProcess] Connected to redis://localhost:6379/1  
  [2019-03-10 11:08:37,889: INFO/MainProcess] mingle: searching for neighbors
  [2019-03-10 11:08:38,914: INFO/MainProcess] mingle: all alone
  [2019-03-10 11:08:38,940: INFO/MainProcess] celery@localhost.localdomain ready.
  [2019-03-10 11:08:39,090: INFO/MainProcess] Received task: tasks.add[e49a6196-c254-494c-8c38-df49d952a96d]  
  [2019-03-10 11:08:39,092: WARNING/ForkPoolWorker-1] enter call func...
  [2019-03-10 11:08:43,103: INFO/ForkPoolWorker-1] Task tasks.add[e49a6196-c254-494c-8c38-df49d952a96d] succeeded in 4.01190678502s: 10 # 执行结果
  [2019-03-10 11:08:43,774: INFO/MainProcess] Received task: tasks.add[2f9e9f43-7adb-4502-a87f-e585bf7e3cdb]  
  [2019-03-10 11:08:43,775: WARNING/ForkPoolWorker-1] enter call func...
  [2019-03-10 11:08:47,778: INFO/ForkPoolWorker-1] Task tasks.add[2f9e9f43-7adb-4502-a87f-e585bf7e3cdb] succeeded in 4.002815485s: 10
  ```

* 如果运行celery worker 命令的时候报错：

  ```json
  AttributeError: ‘str’ object has no attribute 'iteritems’
  ```

  * 解决办法：
    * 将redis 的版本进行更换，我更换了redis ==2.10.6 就好勒。

### 六、Celery定时任务：

* 首选查看目录结构：

  ```json
  celery_app [目录]
  	__init__.py
  	celeryconfig.py
  	task1.py
  	task2.py
  ```

* 每个文件中执行的代码，以及作用：

  * `__ init__.py`：初始化Celery，通过celery实例加载配置模块

    ```json
    # -*- coding: utf-8 -*-
    from celery import Celery
    
    app = Celery('demo')
    
    # 使用config_from_object方法
    # 'celery_app.celeryconfig'  通过Celery实例加载配置模块
    app.config_from_object('celery_app.celeryconfig')
    ```

  * `celeryconfig.py`：celery 配置文件：

    ```json
    # -*- coding: utf-8 -*-
    from datetime import timedelta
    from celery.schedules import crontab
    # 消息中间件
    BROKER_URL = 'redis://localhost:6379/1'
    
    # 存储执行任务结果
    CELERY_RESULT_BACKEND = 'redis://localhost:6379/2'
    
    # Celery 使用时区, 不指定默认就是UTC
    CELERY_TIMEZONE = 'Asia/Shanghai'
    
    # 导入指定的任务模块
    CELERY_IMPORTS = (
            'celery_app.task1',
            'celery_app.task2'
    )
    
    # celery 定时任务
    CELERYBEAT_SCHEDULE = {
            'task1': {
                    'task': 'celery_app.tasks.add',
                    'schedule': timedelta(seconds=10),  # 每十秒执行一次
                    'args': (2, 8)
            },
            'task2': {
                    'task': 'celery_app.tasks.multiply',
                    'schedule': crontab(hour=13, minute=24),  # 每天的13点24分执行一次
                    'args': (4, 5)
            }
    
    }
    ```

  * `task1.py`：执行函数：

    ```json
    # -*- coding: utf-8 -*-
    
    import time
    # 从__init__.py 把app实例导入进来, 进行加载
    from celery_app import app
    
    @app.task
    def add(x, y):
        time.sleep(3)
        return x + y
    ```

  *  `task2.py`：执行函数：

    ```json
    # -*- coding: utf-8 -*-
    
    import time
    
    from celery_app import app
    
    @app.task
    def multiply(x, y):
        time.sleep(5)
        return x * y
    ```

* **开启celery beat 进程**：`celery beat  -A  celery_app  -l  INFO`

  ```json
  celery beat v4.2.1 (windowlicker) is starting.
  __    -    ... __   -        _
  LocalTime -> 2019-03-10 13:38:36
  Configuration ->
      . broker -> redis://localhost:6379/1
      . loader -> celery.loaders.app.AppLoader
      . scheduler -> celery.beat.PersistentScheduler
      . db -> celerybeat-schedule
      . logfile -> [stderr]@%INFO
      . maxinterval -> 5.00 minutes (300s)
  [2019-03-10 13:38:36,403: INFO/MainProcess] beat: Starting...
  [2019-03-10 13:38:36,437: INFO/MainProcess] Scheduler: Sending due task task1 (celery_app.tasks.add)   #  可以看到task1 每10秒执行一次
  ```

### 七、在Django中使用Celery：

* 首先安装django-celery依赖：`pip install django-celery `

* 安装django：`pip install django==1.8`

  * 创建django项目：`django-admin.py startproject  my_celery[项目名]`
  * 进入my_celery中：`cd  my_celery`
  * 创建django项目应用(app)：`python manage.py  startapp  course[app名字]`
  * 在django中启动celery-worker的命令：`python  manage.py  celery  worker -Q  queue`
  * 在django中启动celery-beat的命令：`python manage.py  celery beat -l INFO`
  * 启动django服务：`python manage.py  runserver`

* 项目目录结构：

  ```json
  my_celery
  	course   # app 应用
  		admin.py
  		__init__.py
  		migrations
  		models.py
  		tests.py
  		views.py  # 处理url请求类视图
  		tasks.py  # 执行任务模块
  	manage.py 
  	my_celery
  		__init__.py
  		celeryconfig.py  # celery配置文件
  		settings.py   # django 的配置文件
  		urls.py   # url请求地址
  		wsgi.py
  ```

  1.celeryconfi.py：

  ```json
  # -*- coding: utf-8 -*-
  
  import djcelery
  from celery import Celery, platforms
  from datetime import timedelta
  
  djcelery.setup_loader()
  
  # 设置队列
  CELERY_QUEUES = {
          'beat_tasks': {
                  'exchage': 'beat_tasks',
                  'exchage_type': 'direct',
                  'binding_key': 'beat_tasks'
          },
          'work_queue': {
                  'exchage': 'work_queue',
                  'exchage_type': 'direct',
                  'binding_key': 'work_queue'
          },
  }
  
  # 解决celery不能用root用户启动问题
  platforms.C_FORCE_ROOT = True
  
  # 设置默认的队列
  CELERY_DEFAULT_QUEUE = 'work_queue'
  
  # 导入指定的任务模块
  CELERY_IMPORTS = (
          'course.tasks',
  )
  
  # 有些情况可以防止死锁
  CELERYD_FORCE_EXECV = True
  
  # 设置并发的worker数量(一般情况下根据CUP核数设置)
  CELERYD_CONCURRENCY = 4
  
  # 任务失败了, 允许重试
  CELERY_ACKS_LATE = True
  
  # 每个worker最多执行100个任务被销毁，可以防止内存泄露
  CELERYD_MAX_TASKS_PER_CHILD = 100
  
  # 单个任务的最大运行时间
  CELERD_TASK_TIME_LIMIT = 12 * 30
  
  # 设置定时任务
  CELERYBEAT_SCHEDULE = {
          'task1': {
                  'task': 'course-task',
                  'schedule': timedelta(seconds=5),
                  'options': {  # 指定任务队列
                          'queue': 'beat_tasks'
                  }
          }
  }
  ```

  2.settings.py：

  ```json
  INSTALLED_APPS = (
   	.........
      'djcelery',  # 将djcelery 添加应用中
      'course'  # 新创建的course应用
  )
  
  # Celery  连接Redis
  from .celeryconfig import *
  BROKER_BACKEND = 'redis'
  BROKER_URL = 'redis://localhost:6379/1'
  CELERY_RESULT_BACKEND = 'redis://localhost:6379/2'
  ```

  3.urls.py:

  ```json
  from django.conf.urls import include, url
  from django.contrib import admin
  
  urlpatterns = [
      # Examples:
      # url(r'^$', 'my_celery.views.home', name='home'),
      # url(r'^blog/', include('blog.urls')),
  
      url(r'^admin/', include(admin.site.urls)),
      url(r'^do/$', 'course.views.do', name='do')  # 配置主url
  
  ```

  4.views.py:

  ```json
  # -*- coding: utf-8 -*-
  from django.http import JsonResponse
  
  from course.tasks import CourseTask
  
  def do(request):
      print 'start do request'
      CourseTask.delay()
      # 如果指定参数的话, 建议使用apply_async
      # CourseTask.apply_async(args=('hello',), queue='work_queue')
      print 'end do request'
      return JsonResponse({'result': 'ok'})
  ```

  5.tasks.py:

  ```json
  # -*- coding: utf-8 -*-
  
  import time
  from celery.task import Task
  
  # 执行任务
  class CourseTask(Task):
      def run(self, *args, **kwargs):
          print 'start course task'
          time.sleep(5)
          print 'end course task'
  ```

  * 在manage.py 当前模块执行启动django服务器命令：`python manage.py runserver`

  * 在mange.py  当前模块启动celery worker 命令：`python manage.py  celery worker -l  INFO`

    * 如果遇到以下错误：

      ```json
      Running a worker with superuser privileges when the
      worker accepts messages serialized with pickle is a very bad idea!
      If you really want to continue then you have to set the C_FORCE_ROOT
      environment variable (but please think about this before you do).
      ```

    * 解决办法：

      ```json
      from celery import Celery, platforms
      # 解决celery不能用root用户启动问题
      platforms.C_FORCE_ROOT = True
      ```

  * 在mange.py  当前模块启动celery  beat  命令：`python manage.py  celery  beat  -l  INFO`

  * 访问：127.0.0.1:8000， 会显示：`{'result': 'ok'}`

### 八、监控工具Flower的使用：

* 用来监控celery任务执行是否成功。

* 安装：`pip install flower`

* 启动命令：`celery  flower  --address==0.0.0.0 --port=5555 --broker=xxxx --basic_auth=xxx:xxx[账号：密码]`

* 在django 中的mange.py 目录中执行命令：`python manage.py celery  flower`

  ```json
  (env_cow) ➜  my_celery python manage.py celery flower
  [I 190314 01:28:32 command:139] Visit me at http://localhost:5555
  [I 190314 01:28:32 command:144] Broker: redis://localhost:6379/1  # 任务队列,redis  db:1
  [I 190314 01:28:32 command:147] Registered tasks: 
      ['celery.backend_cleanup',
       'celery.chain',
       'celery.chord',
       'celery.chord_unlock',
       'celery.chunks',
       'celery.group',
       'celery.map',
       'celery.starmap',
       'course.tasks.CourseTask']  # 项目中执行的任务
  [I 190314 01:28:32 mixins:231] Connected to redis://localhost:6379/1
  ```

  **注意** ：打开`http://localhost:5555  `

  ```json
  [I 190314 01:28:32 command:139] Visit me at http://localhost:5555  
  ```

  打开后会发现，进入这样的一个页面：

![1552527202887](C:\Users\lh9\Desktop\各种DOM\celery\02.png)

### 九、Supervisor部署Celery:

* 安装：`pip install supervisor`

* 设置配置文件：`supervisord -c /etc/supervisord.conf`

* 在mange.py 同级目录创建配置目录：`mkdir conf`

* 将supervisord配置重定向到新创建的conf目录中：`echo_supervisord_conf > conf/supervisord.conf`

* 进程的配置：

  ```json
  [program:my_celery_worker]  # 名字
  command = # 执行命令
  directory = # 相对于执行命令的绝对目录
  environment =PATH= # '环境变量'
  stdout_logfile = # 日志文件  注意日志文件夹必须提前创建
  stderr_logfile = # 出错时的日志文件
  autostart=true  # 自动启动
  autorestart=true # 自动重启
  priority=100  # 优先级
  ```

  



