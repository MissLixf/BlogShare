# 开启docker中MongoDB的认证授权

## 思路

开启MongoDB服务后，默认是没有权限验证的。直接通过IP加端口就可以远程访问数据库，并对数据库进行任意操作。下面介绍一下如何开启docker中MongoDB的权限认证。

安装完MongoDB服务后默认有一个admin数据库，此时admin数据库是空的，没有记录任何权限相关的信息。当admin.system.users一个用户都没有时，即使MongoDB启动时添加了`--auth`参数，如果没有在admin数据库中添加用户，此时不进行任何认证还是可以做任何操作(不管是否以`--auth `参数启动)，直到在admin.system.users中添加一个用户。因此要开启MongoDB的权限认证需要满足两点条件：

* admin.system.users中添加用户
* MongoDB以`--auth`启动

## 实践

* 进入相应的docker容器：

  ```bash
  docker exec -it 容器ID /bin/bash
  ```

* 进入MongoDB交互式命令行并切换到admin数据库

  ```bash
  mongo
  ```

  ```javascript
  use admin
  ```

  MongoDB自带一个数据库叫admin，可以在这个数据库中创建管理员账号。

  所有创建的用户都会存储在system.users集合中。

  ```javascript
  db.getCollectionNames()
  // [ "system.users", "system.version" ]
  ```

* 创建管理员，赋予root权限

  ```javascript
  db.createUser({user:"admin",pwd:"xiiasdfiik34",roles:[{role: 'root', db: 'admin'}]})
  /*
  Successfully added user: {
          "user" : "admin",
          "roles" : [
                  {
                          "role" : "root",
                          "db" : "admin"
                  }
          ]
  }
  */
  ```

* 认证当前用户管理员

  ```javascript
  db.auth('admin', 'xiiasdfiik34')  // 返回1 就是认证成功
  ```

* 创建普通用户
  切换到目标数据库：

  ```javascript
  use test_db
  ```

  创建用户角色，并赋予读写权限

  ```javascript
  db.createUser({
      user: 'test_user',
      pwd: 'vfeqwerfd4334',
      roles: [{role: 'readWrite', db: 'test_db'}]
  })
  /*
  Successfully added user: {
          "user" : "test_user",
          "roles" : [
                  {
                          "role" : "readWrite",
                          "db" : "test_db"
                  }
          ]
  }
  */
  ```

  认证试试：

  ```javascript
  db.auth('test_user', 'vfeqwerfd4334')
  ```

  注意：
  用户只能在创建该用户时所在的数据库进行认证。认证成功后，就可以对该数据库执行权限范围内的操作。

* 修改MongoDB的docker-compose.yml，指定带认证启动

  ```yaml
  yikaoyan-mongodb:
      image: mongo:latest
      volumes:
        - ./data/db:/data/db
      ports:
          - 27017:27017
      command: [--auth]  # 指定需要认证
  ```

* 重启MongoDB docker容器后，认证就生效了。

* 连接数据库时，指定用户和密码

  ```python
  mongodb_uri = 'mongodb://test_user:vfeqwerfd4334@localhost/test_db?authSource=test_db&authMechanism=SCRAM-SHA-1'
  ```

  MongoDB连接URL可以参考：<https://api.mongodb.com/python/current/examples/authentication.html>

* 其他命令：
  * 删除用户：`db.dropUser('username')`