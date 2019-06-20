# nginx + uWSGI + Django

web服务（nginx）面向外部，可以直接从文件系统提供文件服务（HTML， CSS等）。但是，ningx 不能直接和Django应用通信。它需要某种东西来运行Django应用，可以将客户端的请求丢给这个东西并拿到响应。

WSGI(Web Server Gateway Interface)就是做这个事情的，它是一个Python标准，而uWSGI是WSGI的实现。

最终结构如下：

```
the web client <-> the web server(nginx) <-> the socket <-> uwsgi <-> Django
```

对于大型部署，推荐一台服务器处理静态文件，一台服务器处理django应用。uWSGI支持多种协议：HTTP, FastCGI, SCGI，以及uwsgi，当然，性能最好的是uwsgi协议，而nginx从0.8.40版本后，对此也提供了支持。



### 方案一：http协议 + 端口

#### uWSGI 

```shell
uwsgi --http-socket 127.0.0.1:11112 --chdir /code/yikaoyan/ --wsgi-file yikaoyan/wsgi.py --master --processes=4 --threads=2 --stats 127.0.0.1:9191
```

说明:

* `--http-socket` 使用http协议，适用于前台有服务器（nginx），或者性能要求高的情况，生成环境中建议使用`--http-socket`代替`--http`
* `--stats` 状态监控，curl [http://127.0.0.1:9191](http://127.0.0.1:9191/)    即可查看监控信息

#### nginx

```ini
location / {
        proxy_pass http://127.0.0.1:11112;
    }
```

### 方案二：uwsgi协议 + 端口

#### uWSGI

```SHELL
uwsgi --socket :11112 --chdir /code/yikaoyan/ --wsgi-file yikaoyan/wsgi.py --master --processes=4 --threads=2 --stats 127.0.0.1:9191
```

说明：

* `--socket` 使用uwsgi协议，端口11112

#### nginx

```ini
upstream django {
    server 127.0.0.1:11112; # for a web port socket (we'll use this first)
}

server {
    location / {
        uwsgi_pass django;
        include /etc/nginx/uwsgi_params;  
}
```

说明：

* `uwsgi_params `文件，可以从github上下载： <https://github.com/nginx/nginx/blob/master/conf/uwsgi_params>
* 将所有请求通过uswgi协议，转发到11112端口

### 方案三：uwsgi协议 + unix socket

unix socket 比起TCP/IP host:port，在大负载的情况下具有更好的性能，一般情况下感知不明显：
<https://serverfault.com/questions/195328/unix-socket-vs-tcp-ip-hostport>

不过测试中并没有体现出性能优势。。。

#### sock文件

在项目目录下直接新建即可，比如`touch yikaoyan.sock`

### uWSGI

```shell
uwsgi  --chdir /code/yikaoyan/ --socket yikaoyan.sock  --wsgi-file yikaoyan/wsgi.py --master --processes=2 --threads=4 --stats 127.0.0.1:9191
```

说明：

*  `--socket` 指定使用的socket文件

### nginx

```ini
upstream django {
    server unix:///yikaoyan/yikaoyan-api/yikaoyan/yikaoyan.sock; 
}
```

说明：

* 注意，`unix:`后面没有空格，且3个`/`

### uwsgi监控

使用uwsgitop工具，可以实现对uwsgi类似top命令的监控，github: <https://github.com/xrmx/uwsgitop>

安装：`pip install uwsgitop`

#### 方案一：socket

```shell
uwsgi --module myapp --socket :3030 --stats /tmp/stats.socket
```

用`--status`选项指定socket，然后：

```shell
uwsgitop /tmp/stats.socket
```

不过如果是跑在容器中，就要进入容器中查看了。

#### 方案二：http

```shell
uwsgi  --module yikaoyan.wsgi --processes=10 --threads=2 --buffer-size=65535  --http :11122 --stats :9191 --stats-http
```

查看监控：

```shell
uwsgitop http://127.0.0.1:9191
```







