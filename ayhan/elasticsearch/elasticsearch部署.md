推荐到官网下载ES源码的打包，个人觉得比起通过yum等包管理工具安装更灵活，且方便管理，比如

* 目录自主可控，便于配置
* 安装插件更方便
* 同义词等词库词库维护

##ES长期运行

推荐使用进程管理工具来运行ES，这里使用supervisor，将es作为supervisor的子进程运行。

### 配置supervisord

```ini
[supervisord]
; ....
nodaemon=false               ; (start in foreground if true;default false)

[program:elk_01]
directory=/yikaoyan/elasticsearch-7.2.1
command=/usr/bin/bash bin/elasticsearch
user=elk  ; 注意，ES不能以root身份运行
```

注意，上面的`nodaemon=false`，如果为true，通过systemctl命令将无法启动supervisord服务。

安装并设置`supervisord`服务开机自启

```bash
systemctl enable supervisord  // 允许开机启动
```

允许开机启动后，将在`/lib/systemd/system`目录下生成`supervisord.service`文件，编辑该文件，可以编辑supervisord的启动命令，这里重新制定supervisor的配置文件目录：

```ini
[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
```

注意，每次修改完配置文件后（包括修改以上文件和supervisord.conf），都需要通过如下命令重载后再启动服务：

```bash
systemctl daemon-reload
systemctl start supervisord
```

这样每次启动supervisord，就可以自动运行ES

### ES非root用户运行问题

es默认不允许以root身份运行，可以通过添加用户并授权该用户对es目录的操作权限来解决：

```bash
# 添加用户
useadd elk
# 授权该用户和用户组对es目录的操作权限
chown -R elk:elk elasticsearch-folder
```



##ES认证和NGINX

es默认没有认证，有未授权访问漏洞的危险。商业套件x-pack中提供认证功能，但是需要授权，试用只有一个月。另外ES也有第三方插件search-guard提供安全方面的功能，但是生产环境中配置比较复杂。这里我采用nginx的`Http Basic Authentication`来做认证功能（官网文档[参考](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)），并利用nginx将搜索请求通过端口转发给es。下面是我的nginx配置参考：

```nginx
upstream elk {
    server 127.0.0.1:9200;
}

server {
    listen 8080;  # 监听8080端口，并将请求转发到本机9200端口
    access_log  /var/log/nginx/statistics_access.log;

    client_max_body_size 4G;
    fastcgi_buffers 64 8K;
    client_body_buffer_size 1024k;
    keepalive_timeout 5;
    client_body_timeout 15;
    client_header_timeout 15;

    send_timeout 15;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    location / {
        proxy_pass http://elk;  # 重要，http://不可少
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffers 8 32k;
        proxy_buffer_size 64k;

        # 配置访问账号密码
        auth_basic "auth required";
        auth_basic_user_file /etc/nginx/conf.d/htpasswd/es;
    }
}

```

出于安全考虑，再采取如下两个措施：

* 编辑ES配置文件`.../elasticsearch-7.2.1/config/elasticsearch.yml`，绑定ES仅限本机访问：

  ```yaml
  
  # ---------------------------------- Network -----------------------------------
  #
  # Set the bind address to a specific IP (IPv4 or IPv6):
  #
  network.host: 127.0.0.1
  #
  # Set a custom port for HTTP:
  #
  # http.port: 9200
  ```

* 在安全组中关闭其他端口的访问，只开放上面8080端口的访问。

### 连接ES

Kibana修改连接配置：

```yaml
# elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.hosts: ["http://xx.xx.xx.xxx:8080"]

# ...

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
elasticsearch.username: "user"
elasticsearch.password: "pwd"
```

Python客户端连接：

```python
ES_CLIENT = Elasticsearch(['http://user:pwd@xx.xx.xx.xxx:8080'])
```

注意，ES貌似不支持通过域名的方式访问，必须指定IP和端口。