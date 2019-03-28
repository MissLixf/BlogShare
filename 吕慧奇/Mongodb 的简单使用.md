# Mongodb 数据库的简单使用

## 一、数据库的概述：

### 1.什么是数据库？

* 数据库(Database)是按照`数据结构来组织、存储和管理数据`的仓库。

* `关系型数据库`：Access、mySql、SqlServer、oracle、db2、SQLite等。

* `非关系型数据库`: MongoDB、Redis、HBase、CouchDB等。

* 图解SQL与NoSQL的对比：

  ![1553219195416](C:\Users\lh9\Desktop\各种DOM\mongodb简介\01.png)

   * NoSQL中，最小的`数据条目，不是行，而是文档`。

   * `文档就是键值对的一个集合，实际上表达方式和JSON一样`。

      * 简单来说：其实这个`文档就是 BSON` 。

      * 那么什么是BSON呢？

         * `bson 是 json 的超集`，比如 `json 中没法储存二进制类型，而 bson 拓展了类型，提供了二进制支持`。

           

### 2.MongoDB 简介：

* MongoDB是一个基于**分布式**文件存储的数据库。由 C++ 语言编写。为 WEB 应用提供`可扩展的高性能`数据存储解决方案。

* 它支持的数据结构非常松散，是类似`json的bson格式`，因此可以`存储比较复杂的数据类型`。

* Mongo最大的特点是它支持的`查询语言非常强大`，其语法有点类似于`面向对象的查询语言`，几乎可以实现类似关系数据库`单表查询的绝大部分功能`，而且还支持对数据`建立索引`。 

* **MongoDB 是非关系型数据库当中功能最丰富，最像关系型数据库的。**

  

### 3.MongoDB的优点：

* 它的特点是`高性能、易部署、易使用，存储数据非常方便`。

* 主要功能特性有：

  * 面向集合存储，易存储对象类型的数据。

  *  模式自由。

  *  支持动态查询。

  *  支持完全索引，包含内部对象。

  * 支持查询。

  *  支持复制和故障恢复。

  * 使用高效的二进制数据存储，包括大型对象（如视频等）。

  *  自动处理碎片，以支持云计算层次的扩展性

  * 支持RUBY，PYTHON，JAVA，C++，PHP等多种语言。

  * 文件存储格式为BSON（一种JSON的扩展）

    

### 4.BSON数据格式介绍：

* BSON是一种类似`json的二进制形式的存储格式`，简称Binary JSON，它和JSON一样，支持内嵌的`文档对象`和`数组对象`，但是**BSON有JSON没有的一些数据类型，**如Date和BinData类型。

* BSON有三个特点：`轻量性、可遍历性、高效性`。

  

### 5.非关系型数据库与关系型数据库区别：

* **非关系型数据库的优势**：

  1.性能：

   * NOSQL是基于键值对的，可以想象成表中的`主键和值的对应关系`，而且不需要经过SQL层的解析，所以`性能非常高`。

  2.可扩展性：

  * 同样也是因为基于键值对，数据之间没有耦合性，所以`非常容易水平扩展`。

* **关系型数据库的优劣**：

  1.复杂查询：

  * 可以用SQL语句方便的在多个表之间做`非常复杂的数据查询`。

  2.事务支持：
   * 使得对于安全性能很高的数据访问要求得以实现。

     

## 二、MongoDB下载与安装：

### 1.下载MongoDB:

* 根据自己的操作系统上官网进行下载：

 * MongoDB官网下载地址：https://www.mongodb.com/download-center#community



### 2.安装MongoDB:

* 下载完成后，点击安装，一路next即可完成安装。 (本人win10操作系统)

* 安装完成后，进入目录：

  ```json
  D:\Program Files\MongoDB\Server\3.6\bin    // 可能安装目录会有不同, 我安装在D盘
  ```



### 3.MongoDB文件介绍：

* 进入目录后：

* **mongo.exe**：用于连接MongoDB数据库服务器  (管理器)

* **mongod.exe**：用于开启服务器 

* mongodump.exe：数据备份

* mongoexport.exe：数据导出

* mongoimport.exe：数据导入

* mongorestore.exe：数据还原

  

### 4.配置环境变量：

* 将目录进行复制：

  ```json
  D:\Program Files\MongoDB\Server\3.6\bin 
  ```

* 到桌面鼠标右键点击`我的电脑`，单机属性：

* 点击`高级系统设置`

  ![1553318648137](C:\Users\lh9\Desktop\各种DOM\mongodb简介\02.png)

  

* 点击环境变量：

  ![1553318693762](C:\Users\lh9\Desktop\各种DOM\mongodb简介\03.png)

* 选中`PATH`点击`编辑`:

  ![1553318731916](C:\Users\lh9\Desktop\各种DOM\mongodb简介\04.png)

* 将mongo的目录添加到环境变量中，点击`确定`：

  ![1553318827178](C:\Users\lh9\Desktop\各种DOM\mongodb简介\05.png)

* 如果你是win7的电脑，添加环境变量的时候，以`;`英文的分号进行`分隔目录路径`。



### 5.创建文件夹：

* 创建文件夹，`存放日志和数据库`，存放的目录中`不要有中文或特殊符号`。

* 将文件夹创建在D盘中，创建mongodb文件夹用于存储日志和数据库：

  ```json
  D:\mongodb
  ```

* 在mongodb文件夹中，在创建两个文件夹：

  ```json
  D:\mongodb\db   // 用于存储数据库
  D:\mongodb\log   // 用于存储日志
  ```

  ![1553319454889](C:\Users\lh9\Desktop\各种DOM\mongodb简介\06.png)



### 6.开启服务器(挂起)：

#### 6.1：手动开启：

* 将存储的日志与数据库文件夹挂载到mongo服务器并开启服务器：
  * 该操作一定是`管理员权限`(按win键，输入cmd，看见`命令提示符`点击右键，选择`以管理员身份运行`)

    ![1553319856117](C:\Users\lh9\Desktop\各种DOM\mongodb简介\07.png) 

* 以管理员权限执行命令：`mongod --dbpath D:\mongodb\db --logpath D:\mongodb\log\mongod.log`

  * 该命令执行完，就是启动一台mongo服务器，`千万不要关闭`，否则Mongo客户端无法连接，下面介绍一另一种方法，来解决该问题。

#### 6.2：自动开启：

* `将mongo挂载成windows的服务，开启自动启动`：

  * 将mongo添加到windows服务中：

    * 管理员身份执行命令：`mongod --dbpath D:\mongodb\db --logpath D:\mongodb\log\mongod.log --install --serviceName "Mongodb"`

  * 操作服务的命令，开启/停止/卸载：

    * 管理员身份执行开启命令：`net start mongodb`

      ```json
      C:\WINDOWS\system32>net start mongodb[服务的名字，windowns不区分大小写的]
      Mongodb 服务正在启动 .
      Mongodb 服务已经启动成功。
      ```

    * 管理员身份执行停止命令：`net stop mongodb`

    * 管理员身份执行卸载命令：`sc delete mongodb`

* 命令参数：
  - --storageEngine mmapv1：
    - MongoDB 支持WiredTiger引擎与MMAPv1引擎，为了兼容`可视化工具，采用MMAPv1方式启动`。
  - --dbpath：
    - 数据库保存位置
  - --logpath：
    - 数据库日志保存位置
  - --install：
    - 将本服务器安装到windowns中
  -  --serviceName:
    - 安装的服务起的名字
  - **注意**：如果是32位操作系统的必须加存储引擎 `--storageEngine mmapv1`
    - `mongod --storageEngine mmapv1 --dbpath D:\mongodb\db --logpath D:\mongodb\log\mongod.log`



### 7.连接服务器：

* 连接启动mongo服务器，再开启一个`黑窗口(cmd)`，执行命令mongo，提示下方说明安装成功:

  ```json
  C:\Users\lh9>mongo
  MongoDB shell version v3.6.4
  connecting to: mongodb://127.0.0.1:27017
  MongoDB server version: 3.6.4 
  # 输入 show dbs;  显示数据库列表
  > show dbs;  
  admin  0.000GB
  local  0.000GB
  ```



## 三、MongoDB基础命令：

### 1.数据库操作：

* **显示数据库列表**：`show dbs；`

  ```json
  > show dbs;
  admin   0.000GB   // admin：数据库名称   0.0000GB 数据库使用空间的大小
  config  0.000GB
  course  0.000GB
  local   0.000GB
  ```

* **显示当前正在使用的数据库**：`db；`

  ```json
  > db   // 如果刚使用Mongodb数据库的话, 默认数据库是test
  course
  ```

* **创建或切换数据库**：`use dbName`

  * `如果数据库不存在，就创建数据库  dbName， 否则切换到指定数据库`。
  * `创建的数据库并不在数据库列表中，要显示它，就需要向数据库  dbName插入一些数据`。

  ```json
  > use cow[数据库名称]
  switched to db cow
  ```

  ```json
  > show dbs;  // 发现再数据库显示列表中, 并没有刚创建的数据库名字
  admin   0.000GB
  config  0.000GB
  course  0.000GB
  local   0.000GB
  ```

  ```json
  > db.course[集合名称].insert({"name":"张三", "age":18})   // 需要插入一条数据
  WriteResult({ "nInserted" : 1 })
  ```

  ```json
  > show dbs;
  admin   0.000GB
  config  0.000GB
  course  0.000GB
  cow     0.000GB   // 插入一条数据后，再数据库列表中，显示了该数据库名称
  local   0.000GB
  ```

* **删除数据库**：`db.dropDatabase();`

  * **注意**：删除数据库的时候，要`切换到准备删除的数据库`，删除前先`查看`一下当前所在的是哪个数据库。

    ```json
    > db  // 查看当前所在数据库
    cow
    ```

    ```json
    > db.dropDatabase();   // 会删除当前所在的数据库 cow
    { "dropped" : "cow", "ok" : 1 }
    ```

    ```json
    > show dbs;   // 数据库列表页中，已经将删除的cow 移除
    admin   0.000GB
    config  0.000GB
    course  0.000GB
    local   0.000GB
    ```

* **MongoDB中默认的数据库为test，如果你没有创建新的数据库，集合将存放在test数据库中**。

* **MongoDB是要区分大小写的**。

  

### 2.集合操作：

* **创建集合**：`db.createCollection('name', options);`

  * name：是要创建集合的名称(必选)

  * options：是一个文档，用于指定集合的位置(可选)

  * 举个栗子：不限制集合的大小

    * `db.createCollection('course')`

      ```json
      > db.createCollection("course");
      { "ok" : 1 }
      ```

  * 举个栗子：限制集合的大小

    * `db.createCollection('basecourse', {capped: true, size: 10})`

      ```json
      > db.createCollection('basecourse', {capped:true, size:10})
      { "ok" : 1 }
      ```

      * 参数capped：默认为false表示限制不设置上限，值为true表示设置上限。
      * 参数size：当capped值为true时，需要指定此参数，表示上限大小，当文档达到上限时，会将之前的数据覆盖，`单位为字节`。

  * **再MongoDB中不需要创建集合，当插入一些文档时，会自动创建集合。**

* **显示集合**：`show  collections;`或`show tables;`

  * 显示此数据库中集合列表

    ```json
    > show collections; 
    basecourse
    course
    ```

    ```json
    > show tables;
    basecourse
    course
    ```

* **删除集合**：` db.集合名.drop();`

  ```json
  > db.basecourse.drop();
  true
  ```

  ```json
  > show collections;  // 已经将basecourse集合删掉了
  course
  ```



### 3.文档操作：

* **新增数据**：`db.集合名.insert({"健名"： "值名"， "KeyName":  "ValueName"})`

  ```json
  > db.course.insert({"price": 18 , "name": "计算机"});
  WriteResult({ "nInserted" : 1 })
  ```

  * **如果集合名存在则写入数据，不存在就创建集合并写入数据。**

* **查看数据**：`db.集合名.find();`

  ```json
  > db.course.find();
  { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "price" : 18, "name" : "计算机" }
  ```

  * `_id是自动生成主键，主键是每条数据的唯一标识，不能重复，就像身份证是每个人唯一的编号一样。`

  * **格式化查询数据**：`db.集合名.find().pretty();`

    ```json
    > db.course.find(); // 没有使用pretty()的样式
    { "_id" : ObjectId("5c95f427b96cea18058444cd"), "price" : 22, "name" : "计算机", "desc" : "这是一节课程, 师资力量很强大 ！" }
    { "_id" : ObjectId("5c95f43eb96cea18058444ce"), "price" : 22, "name" : "计算机", "desc" : "这是一节课程, 师资力量很强大 ！", "uni" : "清华大学" }
    ```

    ```json
    > db.course.find().pretty(); // 使用pretty()的样式
    {
            "_id" : ObjectId("5c95f427b96cea18058444cd"),
            "price" : 22,
            "name" : "计算机",
            "desc" : "这是一节课程, 师资力量很强大！"
    }
    {
            "_id" : ObjectId("5c95f43eb96cea18058444ce"),
            "price" : 22,
            "name" : "计算机",
            "desc" : "这是一节课程, 师资力量很强大！",
            "uni" : "清华大学"
    }
    ```

* **修改数据**：

  * 修改整个文档成指定的值：`db.集合名.update({"查询条件"}, {"修改目标"});`

    ```json
    > db.course.find();  // 先查看course集合的文档
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "price" : 18, "name" : "计算机" }
    ```

    ```json
    > db.course.update({'price': 18}, {'name': '土木工程'}); // 将price为18(条件), 修改name
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    ```

    ```json
    > db.course.find();
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "name" : "土木工程" }
    # 可以看到仅剩下name， 而price(做为查询条件)没有了
    ```

  * 修改指定键名的值：`db.集合名.update({"查询条件"}, {$set: {"键名"："新的值名"}});`

    ```json
    > db.course.find() // course集合有2条文档
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "name" : "土木工程" }
    { "_id" : ObjectId("5c95ef8cb96cea18058444c7"), "price" : 20, "name" : "计算机" }
    ```

    ```json
    > db.course.update({'price':20}, {$set: {'name': '土木工程'}});
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    # 将price=20作为查询条件, 将name的值进行更改
    ```

    ```json
    > db.course.find() // 可以看到, price:20 的值还存在, 仅将指定的name值进行了更改 
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "name" : "土木工程" }
    { "_id" : ObjectId("5c95ef8cb96cea18058444c7"), "price" : 20, "name" : "土木工程" }
    ```

* **删除数据**：

  * 删除指定的数据：`db.集合名.remove({"查询条件"});`

    ```json
    > db.course.find(); // 查看course集合的所有文档
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "name" : "土木工程" }
    { "_id" : ObjectId("5c95ef8cb96cea18058444c7"), "price" : 20, "name" : "土木工程" }
    { "_id" : ObjectId("5c95f200b96cea18058444c8"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f201b96cea18058444c9"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f202b96cea18058444ca"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f202b96cea18058444cb"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f205b96cea18058444cc"), "price" : 22, "name" : "计算机" }
    ```

    ```json
    > db.course.remove({'price': 22});  // 删除键名为price 值为22的文档
    WriteResult({ "nRemoved" : 1 })  
    ```

    ```json
    > db.course.find() // 可以看到键名price 值22的文档已经删除
    { "_id" : ObjectId("5c95ebf8b96cea18058444c6"), "name" : "土木工程" }
    { "_id" : ObjectId("5c95ef8cb96cea18058444c7"), "price" : 20, "name" : "土木工程" }
    { "_id" : ObjectId("5c95f200b96cea18058444c8"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f201b96cea18058444c9"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f202b96cea18058444ca"), "price" : 20, "name" : "计算机" }
    { "_id" : ObjectId("5c95f202b96cea18058444cb"), "price" : 20, "name" : "计算机" }
    ```

  * 删除此集合所有的数据：`db.集合名.remove({});`

    ```json
    > db.course.remove({});  // 删除此集合下所有的文档
    WriteResult({ "nRemoved" : 6 })
    ```

    ```json
    > db.course.find();  // 下面已经没有任何文档
    ```



## 四、数据高级命令操作：

### 1.按条件查询：

* **根据条件进行查询**：`db.集合名.find({"查询条件"});`

  ```json
  > db.course.find().pretty(); // 查看当前集合的数据
  {
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c07b96cea18058444df"),
          "name" : "哈尔滨工业大学课程",
          "price" : 288,
          "desc" : "这是哈尔滨工业大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c16b96cea18058444e0"),
          "name" : "北京大学",
          "price" : 2288,
          "desc" : "这是北京大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c34b96cea18058444e1"),
          "name" : "沈阳医药大学",
          "price" : 388,
          "desc" : "这是沈阳医药大学的考研课程！"
  }
  ```

  ```json
  > db.course.find({"price": 2288}); // 查询课程价格(price)等于2288的课程
  { "_id" : ObjectId("5c960c16b96cea18058444e0"), "name" : "北京大学", "price" : 2288, "desc" : "这是北京大学的考研课程！" }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price = 2288`

* **$gt  大于 、$gte  大于等于**：`db.集合名.find({"键名"： {$gt：值} })`

  ```json
  > db.course.find({"price": {$gt: 600}}).pretty(); // 查询课程价格大于600元的课程
  {
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c16b96cea18058444e0"),
          "name" : "北京大学",
          "price" : 2288,
          "desc" : "这是北京大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price > 600`

* **$ lt 小于、$lte  小于等于**：`db.集合名.find({"键名"： {$lte：值} })`

  ```json
  > db.course.find({"price": {$lte: 600}}).pretty(); // 查询课程价格小于等于600元的课程
  {
          "_id" : ObjectId("5c960c07b96cea18058444df"),
          "name" : "哈尔滨工业大学课程",
          "price" : 288,
          "desc" : "这是哈尔滨工业大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c34b96cea18058444e1"),
          "name" : "沈阳医药大学",
          "price" : 388,
          "desc" : "这是沈阳医药大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price <= 600`

* **选择区间(and操作符)**：`db.集合名.find({"键名"： {$gt：值，$lt：值}});`

  ```json
  > db.course.find({"price": {$gt:388, $lt:2288}}).pretty();//查询课程价格大于388元小于2288元的课程
  {
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price > 388 and price < 2288;`

* **$ne  不等于**：`db.集合名.find({"键名"： {$ne：值}})`

  ```json
  > db.course.find({"price": {$ne: 388}}).pretty();//查询课程价格不等于388元的课程
  {
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c07b96cea18058444df"),
          "name" : "哈尔滨工业大学课程",
          "price" : 288,
          "desc" : "这是哈尔滨工业大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c16b96cea18058444e0"),
          "name" : "北京大学",
          "price" : 2288,
          "desc" : "这是北京大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price != 388;`

* **$in 再集合中**：`db.集合名.find({"键名"： {$in：["值1"， "值2"， "值3"]}})；`

  ```json
  > db.course.find({"price": {$in: [288, 888]}}).pretty();// 查询课程价格是288元、888元课程
  {
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c07b96cea18058444df"),
          "name" : "哈尔滨工业大学课程",
          "price" : 288,
          "desc" : "这是哈尔滨工业大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price in (288, 888);`

* **$nin 不再集合中**：`db.集合名.find({"键名"： {$nin：["值1"， "值2"， "值3"]}})；`

  ```json
  > db.course.find({"price": {$nin: [288, 888]}}).pretty();//查询课程价格不是288、888的课程
  {
          "_id" : ObjectId("5c960c16b96cea18058444e0"),
          "name" : "北京大学",
          "price" : 2288,
          "desc" : "这是北京大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c34b96cea18058444e1"),
          "name" : "沈阳医药大学",
          "price" : 388,
          "desc" : "这是沈阳医药大学的考研课程！"
  }
  ```

  * 类似SQL语法：`SELECT * FROM Course WHERE price not in (288, 888);`

* **$size 值的个数(值必须是数组)**：`db.course.find({"键名"：{$size：值得个数}})`

  ```json
  db.course.find().pretty();// 目前course集合中的数据
  {
          "_id" : ObjectId("5c9622cab96cea18058444e2"),
          "name" : "上海财经大学",
          "price" : 888,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学"
          ]
  }
  {
          "_id" : ObjectId("5c9622e0b96cea18058444e3"),
          "name" : "北京大学",
          "price" : 688,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学",
                  "经济学"
          ]
  }
  {
          "_id" : ObjectId("5c9622fcb96cea18058444e4"),
          "name" : "哈尔滨工业大学",
          "price" : 588,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学",
                  "经济学",
                  "心理学"
          ]
  }
  ```

  ```json
  > db.course.find({"discipline": {$size: 3}}).pretty();// 查询课程专业有3个的课程
  {
          "_id" : ObjectId("5c9622cab96cea18058444e2"),
          "name" : "上海财经大学",
          "price" : 888,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学"
          ]
  }
  ```

* **$exists  是否存在某个键名**：`db.集合名.find({"键名": {$exists：true |  false}})`

  ```json
  > db.admin.find(); // 首先查看一下, admin集合中有哪些文档
  { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
  { "_id" : ObjectId("5c96fb63652272d45ce19952"), "name" : "李四", "age" : 28 }
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32 }
  ```

  ```json
  > db.admin.find({"height": {$exists: true}});//查找admin集合中有height的键, 有哪些
  { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
  ```

  ```json
  > db.admin.find({"height": {$exists: false}});// 查找admin集合中没有height的键，有哪些
  { "_id" : ObjectId("5c96fb63652272d45ce19952"), "name" : "李四", "age" : 28 }
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32 }
  ```

* **$or  或者，多个条件满足一个**：`db.集合名.find.({$or：[{"条件1"}，{"条件2"}， {"条件3"}]})；`

  ```json
   //查询姓名是张三或年龄是32岁的管理员
  > db.admin.find({$or: [{"name": "张三"}, {"age": 32}]}).pretty();
  {
          "_id" : ObjectId("5c96fb55652272d45ce19951"),
          "name" : "张三",
          "age" : 18,
          "height" : 180
  }
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32 }
  ```

  	* 类似SQL语法：`SELECT * FROM Admin  WHERE name="张三" OR age=32;`

* **模糊查询**：`db.集合名.find({"键名"：值[必须是正则表达式]});`

  ```json
  > db.admin.find({"name": /张/}); // 查询管理员名字包含张的有哪些人员
  { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
  { "_id" : ObjectId("5c9708ea652272d45ce19954"), "name" : "张三三", "age" : 22 }
  ```

  * 类似SQL语法：`SELECT * FROM Admin WHERE name LIKE "%张%"；`



### 2.排序查询：

* **升序、降序**：`db.集合名.find().sort({"键名1"：1[代表升序]，"键名2"：-1[代表降序]});`

  * 升序：指从小到大进行排序。
  * 降序：指从大到小进行排序。

  ```json
  > db.admin.find().sort({"age": 1}); // 按管理员年龄进行升序排序
  { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
  { "_id" : ObjectId("5c9708ea652272d45ce19954"), "name" : "张三三", "age" : 22 }
  { "_id" : ObjectId("5c96fb63652272d45ce19952"), "name" : "李四", "age" : 28 }
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32 }
  ```

  ```json
  > db.admin.find().sort({"age": -1}); // 按管理员年龄进行降序排序
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32 }
  { "_id" : ObjectId("5c96fb63652272d45ce19952"), "name" : "李四", "age" : 28 }
  { "_id" : ObjectId("5c9708ea652272d45ce19954"), "name" : "张三三", "age" : 22 }
  { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
  ```

  * 类似SQL语法：`SELECT * FROM Admin  ORDER BY age  ASC[升序]| DESC[降序]`

    

### 3.索引：

* `索引通常能够极大的提高查询的效率`，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种`扫描全集合的查询效率是非常低的`，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

* `索引是特殊的数据结构`，索引`存储在一个易于遍历读取的数据集合中`，索引是对数据库表中一列或多列的值进行排序的一种结构。

* ##### 索引的性能

  * 索引使得可以通过关键字段获取数据，能够使得快速查询和更新数据。

* **注意**：

  * `索引也会在插入和删除的时候增加一些系统的负担`。往集合中插入数据的时候，`索引的字段必须加入到B-Tree中去`，因此，`索引适合建立在读远多于写的数据集上`，对于写入频繁的集合，在某些情况下，索引反而有副作用。不过大多数集合都是读频繁的集合，所以集合在大多数情况下是有用的。 

  

* **使用ensureIndex() 方法来创建索引**：`db.集合名.ensureIndex({"键名"：1});`

  * `"键名"为你要创建的索引字段，1指定按升序创建索引，-1按降序来创建索引`。

  * 举个栗子：

    * 查询管理员姓名是张三的信息：

      ```json
      db.admin.find({"name": "张三"});
      ```

    * 如果`没有`对name 字段建立索引，数据库在`查询的时候会扫描所有的数据`，如果数据量小的时候，感觉不出来速度慢，当数据越来越多的时候，就会越来越慢。

    * 这个时候如果给name 建立一个索引，查询速度就会加快。

      ```json
      > db.admin.ensureIndex({"name": 1}); // 给name键创建索引
      {
              "createdCollectionAutomatically" : false,
              "numIndexesBefore" : 1,
              "numIndexesAfter" : 2,
              "ok" : 1
      }
      ```

* **查询当前聚集集合所有索引**：`db.集合名.getIndexes ();`

  ```json
  > db.admin.getIndexes();
  [
          {
                  "v" : 2,	// 索引版本
                  "key" : {  // 索引字段及排序方向
                          "_id" : 1 // 根据_id字段升序索引  -1 代表字段降序
                  },
                  "name" : "_id_",  // 索引的名称
                  "ns" : "cow.admin"  // 集合名
          },
          {
                  "v" : 2,
                  "key" : { 
                          "name" : 1 
                  },
                  "name" : "name_1", 
                  "ns" : "cow.admin" 
          }
  ]
  ```

* **查看总索引记录大小**：`db.集合名.totalIndexSize();`

  ```json
  > db.admin.totalIndexSize();
  53248
  ```

* **读取当前集合的所有索引信息**：`db.集合名.reIndex();`

  ```json
  > db.admin.reIndex();
  {
          "nIndexesWas" : 2,
          "nIndexes" : 2,
          "indexes" : [
                  {
                          "v" : 2,
                          "key" : {
                                  "_id" : 1
                          },
                          "name" : "_id_",
                          "ns" : "cow.admin"
                  },
                  {
                          "v" : 2,
                          "key" : {
                                  "name" : 1
                          },
                          "name" : "name_1",
                          "ns" : "cow.admin"
                  }
          ],
          "ok" : 1
  }
  >
  ```

* **删除指定索引**：`db.集合名.dropIndex("name_1"); `

  ```json
  > db.admin.dropIndex("name_1"); // 注意索引名
  { "nIndexesWas" : 2, "ok" : 1 }
  ```

* **删除所有索引索引**：`db.集合名.dropIndexes();`

  ```json
  > db.admin.dropIndexes();
  {
          "nIndexesWas" : 1,
          "msg" : "non-_id indexes dropped for collection",
          "ok" : 1
  }
  ```

  ```json
  > db.admin.reIndex(); // 可以看到_id的索引是删除不了的，只能删除自己创建的索引。
  {
          "nIndexesWas" : 1,
          "nIndexes" : 1,
          "indexes" : [
                  {
                          "v" : 2,
                          "key" : {
                                  "_id" : 1
                          },
                          "name" : "_id_",
                          "ns" : "cow.admin"
                  }
          ],
          "ok" : 1
  }
  ```

* **默认会在_id字段上创建索引，而且这个特别的索引不能删除。_id字段是强制唯一的，由数据库维护。**

  

### 4.限制输出：

* **limit()限制输出n条**：`db.集合名.find().limit(n);`

  * n：表示限制的数量

  * 如果参数是0，则没有约束，limit()将不起作用

    ```json
    > db.admin.find().sort({"age": 1}).limit(2); // 查询年龄最小的2个管理员文档
    { "_id" : ObjectId("5c96fb55652272d45ce19951"), "name" : "张三", "age" : 18, "height" : 180 }
    { "_id" : ObjectId("5c9708ea652272d45ce19954"), "name" : "张三三", "age" : 22 }
    ```

  * 类似SQL语法：`SELECT * FROM  Admin ORDER BY  age  ASC  LIMIT  2;`

* **skip()跳过n条**：`db.集合名.find().skip(n);`

  * n：表示跳过的数量

  * 如果参数是0，则当作没有约束，skip()将不起作用，或者说跳过了0条

    ```json
    // 找出年龄不包含最小的二位管理员信息(跳过1条, 输出2条)
    > db.admin.find().sort({"age": 1}).skip(1).limit(2);
    { "_id" : ObjectId("5c9708ea652272d45ce19954"), "name" : "张三三", "age" : 22 }
    { "_id" : ObjectId("5c96fb63652272d45ce19952"), "name" : "李四", "age" : 28 }
    ```

  * 类似SQL语法：`SELECT * FROM Admin ORDER BY ASC  LIMIT  1, 2；`

* **limit  和  skip  通常用于实现分页处理**

  

## 五、数据类型：

* MongoDB中常用的几种数据类型：
  * Object ID：文档ID
  * String：字符串，最常用，必须是有效的UTF-8
  * Boolean：存储一个布尔值，true或false
  * Integer：整数可以是32位或64位，这取决于服务器
  * Double：存储浮点值
  * Arrays：数组或列表，多个值存储到一个键
  * Object：用于嵌入式的文档，即一个值为一个文档
  * Null：存储Null值
  * Timestamp：时间戳
  * Date：存储当前日期或时间的UNIX时间格式

### 1.object id:

* 每个文档都有一个属性，为_id，保证每个文档的唯一性
* 可以自己去设置_id插入文档
* 如果没有提供，那么MongoDB为每个文档提供了一个独特的_id，类型为objectID
* objectID是一个12字节的十六进制数
  * 前4个字节为当前时间戳
  * 接下来3个字节的机器ID
  * 接下来的2个字节中MongoDB的服务进程id



## 六、聚合函数 aggregate：

* `聚合(aggregate)主要用于计算数据`，类似sql中的sum()、avg()。

* **核心语法**：

  ```json
  db.集合名.aggregate(
      [
          {管道1：{表达式}},
          {管道2：{表达式}},
          {管道3：{表达式}}, 
      ]
  )
  ```

* **管道概念**：

  * 管道一般`用于将当前命令的输出结果作为下一个命令的输入`，在mongodb中，管道具有同样的作用，`文档处理完毕后，通过管道进行下一次处理`。

* **表达式**：处理输入文档并输出。

* **语法**：表达式：'$列名'。

* 下面所有栗子，所用到的文档：

  ```json
  > db.admin.find().pretty(); // 管理员集合所有的文档
  {
          "_id" : ObjectId("5c96fb55652272d45ce19951"),
          "name" : "张三",
          "age" : 18,
          "height" : 180,
          "sex" : "男"
  }
  {
          "_id" : ObjectId("5c96fb63652272d45ce19952"),
          "name" : "李四",
          "age" : 28,
          "sex" : "女"
  }
  {
          "_id" : ObjectId("5c96fb9a652272d45ce19953"),
          "name" : "王五",
          "age" : 32,
          "sex" : "女"
  }
  {
          "_id" : ObjectId("5c9708ea652272d45ce19954"),
          "name" : "张三三",
          "age" : 22,
          "sex" : "男"
  }
  ```



### 1.常用管道：

* $group：将集合中的文档分组，可用于统计结果

* $match：过滤数据，只输出符合条件的文档

* $project：修改输入文档的结构，如重命名、增加、删除字段、创建计算结果

* $sort：将输入文档排序后输出

* $limit：限制聚合管道返回的文档数

* $skip：跳过指定数量的文档，并返回余下的文档

* $unwind：将数组类型的字段进行拆分

  

### 2.常用表达式(分组-聚合函数)：

* $sum：计算总和
  * 统计个数：$sum:1同count
  * 累加：$sum：'$字段名' 
* $avg：计算平均值
* $min：获取最小值
* $max：获取最大值
* $push：在结果文档中插入值到一个数组中
* $first：根据资源文档的排序获取第一个文档数据
* $last：根据资源文档的排序获取最后一个文档数据



### 3.聚合示例：

* **$group  管道、分组**：

  ```json
  // 语法
  db.集合名.aggregate(
      {$group: 
           {
              _id：'$字段名',  // 标识分组的依据，根据某个字段进行分组
               别名：{$聚合函数：'$字段名'}
          }
     }
  )
  ```

  ```json
  // 统计男管理员、女管理员的总数人数
  > db.admin.aggregate({$group: {_id: "$sex", count: {$sum: 1}}});
  { "_id" : "女", "count" : 2 }
  { "_id" : "男", "count" : 2 }
  ```

  ```json
  // 统计管理员的总数人、平均年龄(计算总人数, 不需要分组,_id设为：null)
  > db.admin.aggregate({$group: {_id: null , count: {$sum:1}, avg: {$avg: "$age"}}});
  { "_id" : null, "count" : 4, "avg" : 25 }
  ```

  ```json
  // 根据性别统计所有管理员人数、所有姓名名单
  > db.admin.aggregate({$group:
                        {_id: "$sex", count: {$sum:1}, name_list: {$push: "$name"}}
                       });
  { "_id" : "女", "count" : 2, "name_list" : [ "李四", "王五" ] }
  { "_id" : "男", "count" : 2, "name_list" : [ "张三", "张三三" ] }
  ```

  ```json
  // 常用表达式的用法
  // 统计管理员的总数人、总年龄、平均年龄
  db.admin.aggregate({$group:
                       {_id:"$sex",  // 按照性别进行分组
                        总人数:{$sum:1},  // 统计总个数
                        总年龄:{$sum:"$age"},   // 累加
                        平均年龄:{$avg:"$age"},  // 平均值
                        最大年龄:{$max:"$age"},  // 最大值
                        最小年龄:{$min:"$age"},  // 最小值
                        名单:{$push:"$name"},  // 按照性别进行分组, 将名字进行添加到数组中
                        管理员位置第一:{$first: "$name"}, // 分组后, 第一个管理员的名字
                        管理员位置最后:{$last: "$name"} // 分组后, 最后一个管理员的名字
  			  }});
  
  { "_id" : "女", "总人数" : 2, "总年龄" : 60, "平均年龄" : 30, "最大年龄" : 32, "最小年龄" : 28, "名单" : [ "李四", "王五" ], "管理员位置第一" : "李四", "管理员位置最后" : "王五" }
  
  { "_id" : "男", "总人数" : 2, "总年龄" : 40, "平均年龄" : 20, "最大年龄" : 22, "最小年龄" : 18, "名单" : [ "张三", "张三三" ], "管理员位置第一" : "张三", "管理员位置最后" : "张三三" }
  ```

* **match  管道、匹配条件**：

  ```json
  //语法：
  db.集合名.aggregate(
      {$match:
       	{"键名":  {$gt：条件}}
      }
  )；
  ```

  ```json
  // 找出年龄大于30岁的管理员
  > db.admin.aggregate({$match: {"age": {$gt: 30}}});
  { "_id" : ObjectId("5c96fb9a652272d45ce19953"), "name" : "王五", "age" : 32, "sex" : "女" }
  ```

  ```json
  // 对年龄大于20岁的管理员进行性别分组统计（多个条件, 使用[]隔开）
  > db.admin.aggregate(
      [
          {$match: {
              "age": {$gt:20}}},
          {$group: {_id: "$sex", 总人数: {$sum:1}}}
      ]
  );
  { "_id" : "男", "总人数" : 1 }
  { "_id" : "女", "总人数" : 2 }
  ```

* **$project  管道、限定输出字段**：

  ```json
  // 语法
  db.集合名.aggregate({
      $project: {
          name:1 | 0  // 1 表示显示字段,  0 表示不显示字段  name 指字段名
      }
  })
  ```

  * 类似于SQL语法：`SELECT name , age  FROM Admin;`

  ```json
  // 找出年龄大于20岁的管理员，仅显示姓名和年龄
  > db.admin.aggregate(
      [
          {$match: {"age": {$gt: 20}}},
          {$project: {_id:0, name:1, age:1}}// 默认情况下_id是显示的，为了不显示_id 把它设置为0
      ]
  );
  { "name" : "李四", "age" : 28 }
  { "name" : "王五", "age" : 32 }
  { "name" : "张三三", "age" : 22 }
  ```

* **$sort  排序管道**：

  ```json
  //语法
  db.集合名.aggregate({
      $sort: {
          "键名"：1 | -1  // 1:代表升序   -1：代表降序
      }
  })
  ```

  ```json
  // 找出年龄大于20岁的管理员，仅显示姓名和年龄, 并按照年龄进行升序
  > db.admin.aggregate(
      [
          {$match: {"age": {$gt:20}}},
          {$project: {_id:0, name:1, age:1}},
          {$sort: {"age": 1}}
      ]
  );
  
  { "name" : "张三三", "age" : 22 }
  { "name" : "李四", "age" : 28 }
  { "name" : "王五", "age" : 32 }
  ```

* **$limit  限制输出**：

  ```json
  //语法
  db.集合名.aggregate(
      	{$limit：限制输出的条数}
  );
  ```

  ```json
  // 找出年龄大于20岁的管理员，仅显示姓名和年龄, 并按照年龄进行升序, 且输出前2条信息
  > db.admin.aggregate(
      [
          {$match: {"age": {$gt:20}}},
          {$project: {_id:0, name:1, age:1}},
          {$sort: {"age": 1}},
          {$limit: 2}
      ]
  );
  { "name" : "张三三", "age" : 22 }
  { "name" : "李四", "age" : 28 }
  ```

* **$skip  管道、跳过n条**：

  ```json
  db.集合名.aggregate({
      $skip: n
  });
  ```

  ```json
  //找出年龄大于20岁的管理员，仅显示姓名和年龄, 并按照年龄进行升序,跳过1条信息, 且输出前1条信息
  > db.admin.aggregate(
      [
          {$match: {"age": {$gt:20}}},
          {$project: {_id:0, name:1, age:1}},
          {$sort: {"age": 1}},
          {$skip: 1},
          {$limit: 1}
      ]
  );
  { "name" : "李四", "age" : 28 }
  ```

* **unwind 管道、将数组字段进行拆分**：

  ```json
  // 语法
  db.集合名.aggregate(
      {$unwind: "$键名"}
  )
  ```

  * 该栗子的文档进行了更换，文档内容如下：

    ```json
    > db.course.find().pretty();
    {
            "_id" : ObjectId("5c960be9b96cea18058444de"),
            "name" : "上海财经大学课程",
            "price" : 888,
            "desc" : "这是上海财经大学的考研课程！"
    }
    {
            "_id" : ObjectId("5c960c07b96cea18058444df"),
            "name" : "哈尔滨工业大学课程",
            "price" : 288,
            "desc" : "这是哈尔滨工业大学的考研课程！"
    }
    {
            "_id" : ObjectId("5c960c16b96cea18058444e0"),
            "name" : "北京大学",
            "price" : 2288,
            "desc" : "这是北京大学的考研课程！"
    }
    {
            "_id" : ObjectId("5c960c34b96cea18058444e1"),
            "name" : "沈阳医药大学",
            "price" : 388,
            "desc" : "这是沈阳医药大学的考研课程！"
    }
    {
            "_id" : ObjectId("5c9622cab96cea18058444e2"),
            "name" : "上海财经大学",
            "price" : 888,
            "discipline" : [
                    "哲学",
                    "建筑学",
                    "体育学"
            ]
    }
    {
            "_id" : ObjectId("5c9622e0b96cea18058444e3"),
            "name" : "北京大学",
            "price" : 688,
            "discipline" : [
                    "哲学",
                    "建筑学",
                    "体育学",
                    "经济学"
            ]
    }
    {
            "_id" : ObjectId("5c9622fcb96cea18058444e4"),
            "name" : "哈尔滨工业大学",
            "price" : 588,
            "discipline" : [
                    "哲学",
                    "建筑学",
                    "体育学",
                    "经济学",
                    "心理学"
            ]
    }
    ```

    ```json
    // 找出课程中包含专业、且课程价格大于700元、并且按照专业字段进行拆分
    > db.course.aggregate(
        [
            {$match:
             	{"discipline": {$exists: true}, "price": {$gt: 700}}},
            {$unwind: "$discipline"}
        ]).pretty();
    
    {
            "_id" : ObjectId("5c9622cab96cea18058444e2"),
            "name" : "上海财经大学",
            "price" : 888,
            "discipline" : "哲学"
    }
    {
            "_id" : ObjectId("5c9622cab96cea18058444e2"),
            "name" : "上海财经大学",
            "price" : 888,
            "discipline" : "建筑学"
    }
    {
            "_id" : ObjectId("5c9622cab96cea18058444e2"),
            "name" : "上海财经大学",
            "price" : 888,
            "discipline" : "体育学"
    }
    ```

## 

## 七、数据库安全：

### 1.进入管理平台：

* 首先以无密码形式登陆

  ```json
  C:\Users\lh9>mongo
  MongoDB shell version v3.6.4
  connecting to: mongodb://127.0.0.1:27017
  MongoDB server version: 3.6.4
  ```

### 2.创建管理员账号密码：

* 如果默认没有 admin数据库，可以自己添加一个。

  ```json
  user admin
  ```

* 添加好数据库可以使用命令添加账户：

  ```json
  db.createUser({user: "root", pwd: "123456", roles: ["root"]})
  Successfully added user: { "user" : "root", "roles" : [ "root" ] }
  ```

  * user：表示用户名
  * pwd：表示设置的密码
  * roles：角色的拥有的权限

* 添加账户后，会发现该admin 数据库下，多了个集合：

  ```json
  > show tables;
  system.users  // 新增集合
  system.version
  ```

* 可以查看一下新增加的集合，有哪些文档：

  ```json
  > db.system.users.find();
  { "_id" : "admin.root", "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "IMreq+7O277NF8N7pmEWnA==", "storedKey" : "oYszRjRN/Ii1lGd31xabkl+Dois=", "serverKey" : "c9cxly4EAsiq+Arx+02FOfR8ps8=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
  ```

### 3.验证密码：

```json
> db.auth("root", "123456")
1    // 1代表成功   0 代表失败
```

### 4.重新挂mongoDB服务：

* 先停止mongodb再windows的服务：`net stop mongodb`

  ```json
  // 以管理员身份操作
  C:\WINDOWS\system32>net stop mongodb
  Mongodb 服务正在停止.
  Mongodb 服务已成功停止。
  ```

* 卸载mongodb的服务：`sc delete mongodb`

  ```json
  //以管理员身份操作
  C:\WINDOWS\system32>sc delete mongodb
  [SC] DeleteService 成功
  ```

* 重新挂载带认证的Mongo服务：

  ```json
  // 以管理员身份操作
  mongod --dbpath D:\mongodb\db --logpath D:\mongodb\log\mongodb.log --install --serviceName "Mongodb" --auth
  ```

  * 参数：
    * --auth：是指身份验证

* 启动服务：

  ```json
  // 以管理身份操作
  C:\WINDOWS\system32>net start mongodb
  Mongodb 服务正在启动 .
  Mongodb 服务已经启动成功。
  ```

### 5.测试设置的验证是否生效：

* 登陆mongo客户端：

  ```json
  C:\WINDOWS\system32>mongo
  MongoDB shell version v3.6.4
  connecting to: mongodb://127.0.0.1:27017
  MongoDB server version: 3.6.4
  ```

* 进入admin 数据库，查看一下拥有哪些集合？

  ```json
  > show collections; // 报错 说明你没有验证 身份
  2019-03-25T10:08:25.243+0800 E QUERY    [thread1] Error: listCollections failed: {
          "ok" : 0,
          "errmsg" : "not authorized on admin to execute command { listCollections: 1.0, filter: {}, $db: \"admin\" }",
          "code" : 13,
          "codeName" : "Unauthorized"
  } :
  _getErrorWithCode@src/mongo/shell/utils.js:25:13
  DB.prototype._getCollectionInfosCommand@src/mongo/shell/db.js:941:1
  DB.prototype.getCollectionInfos@src/mongo/shell/db.js:953:19
  DB.prototype.getCollectionNames@src/mongo/shell/db.js:964:16
  shellHelper.show@src/mongo/shell/utils.js:813:9
  shellHelper@src/mongo/shell/utils.js:710:15
  @(shellhelp2):1:1
  ```

* 验证权限：

  ```json
  > db.auth("root", "123456"); 
  1   // 说明验证成功
  ```

* 再次显示所有集合：

  ```json
  > show collections;  // 可以正常显示，说明权限已经生效
  system.users
  system.version
  ```

* **注意**：`验证权限时，一定先切换到admin数据库，然后再进行权限的验证`。

### 6.为其他用户添加数据库：

* 先通过身份验证，再进入指定的数据库，添加用户。

* 切换到准备添加用户的数据库中：

  ```json
  use cow
  ```

* 数据库添加用户：

  ```json
  > db.createUser({user:"user",pwd:"654321",roles:[{role:"dbOwner",db:"cow"}]}); 
  Successfully added user: {
          "user" : "user",
          "roles" : [
                  {
                          "role" : "dbOwner",
                          "db" : "cow"
                  }
          ]
  }
  >
  ```

  * user：表示用户名
  * pwd： 表示密码
  * role：表示该角色
  * db：该用户指定操作的数据库,

* 该用户切换到指定的数据库后，进行权限验证：

  ```json
  use cow  // 切换到cow数据库
  db.auth("user", "654321");
  1  // 1表示认证成功
  ```

* 该用户验证成功后，只能再cow数据库进行操作, 不能操作其他数据库。



## 八、主从服务器(复制配置)：

* 主从复制是mongodb最常用的复制方式,也是一个简单的`数据库同步备份的集群技术`,这种方式很灵活，`可用于备份,故障恢复,读扩展等`。 

* 图解：

  ![1553565163051](C:\Users\lh9\Desktop\各种DOM\mongodb简介\08.png)

### 1.什么是复制？

* 复制提供了数据的`冗余备份`，并在`多个服务器上存储数据副本，提高了数据的可用性`，并可以保证`数据的安全性。复制还允许从硬件故障和服务中断中恢复数据`。

### 2.为什么要复制？

​	1.数据备份

​	2.数据灾难恢复

​	3.读写分离

​	4.高数据可用性

​	5.无宕机维护

​	6.副本集对应用程序是透明(服务器更新，客户端是感觉不到)

### 3.复制的工作原理：

​	1.最基本的设置方式就是建立一个主节点(A)和一个或多个从节点(B、C)，每个从节点要知道主节点的地址。 

​	2.A是主节点，负责处理客户端请求。

​	3.其余的都是从节点，负责复制主节点上的数据。

​	4.节点常见的搭配方式为：一主一从、一主多从。

​	5.主节点记录在其上的所有操作，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操			作，从而保证从节点的数据与主节点一致。

​	6.主节点与从节点进行数据交互保障数据的一致性。

### 4.复制的特点：

 	1.N 个节点的集群。

​	2.任何节点可作为主节点。

​	3.所有写入操作都在主节点上。

​	4.自动故障转移。

​	5.自动恢复。

### 5.设置复制节点：

* **注意**：**`以下操作都是以管理员身份进行`**

#### 5.1:创建数据库存放目录：

* 创建两个目录mongodb1、mongodb2，用于挂起mongodb服务器：

  ```json
  D:\mongodb\mongodb1  // 目录存放位置
  D:\mongodb\mongodb2  
  ```

#### 5.2:启动mongo服务器：

* **注意**：replSet 的名称是一样的。

* 执行以下命令：

  ```json
  mongod --bind_ip 192.168.66.147 --port 27017 --dbpath D:\mongodb\mongodb1 --replSet rs0
  mongod --bind_ip 192.168.66.147 --port 27018 --dbpath D:\mongodb\mongodb2 --replSet rs0
  ```

  * 参数：
    * --bind_ip：本地的ip地址
    * --port：数据库的端口，注意主从数据库端口不能重复
    * --dbpath：数据库存放目录
    * --replSet：设置集群，rs0 是设置的集群名字，必须是同一个名字

#### 5.3:连接主服务器:

* 此处设置192.168.66.147:27017为`主服务器`。

* 执行以下命令：

  ```json
  C:\Users\lh9>mongo --host 192.168.66.147 --port 27017   // 指定IP地址、端口 主服务器
  MongoDB shell version v3.6.4
  connecting to: mongodb://192.168.66.147:27017/
  MongoDB server version: 3.6.4
  Server has startup warnings:
  ```

* 查看数据库的时候，会发现报错，原因是没有配置主从服务器：

  ```json
  > show dbs  // 查看数据库
  2019-03-26T10:11:17.925+0800 E QUERY    [thread1] Error: listDatabases failed:{
          "ok" : 0,
          "errmsg" : "not master and slaveOk=false",
          "code" : 13435,
          "codeName" : "NotMasterNoSlaveOk"
  } :
  _getErrorWithCode@src/mongo/shell/utils.js:25:13
  Mongo.prototype.getDBs@src/mongo/shell/mongo.js:65:1
  shellHelper.show@src/mongo/shell/utils.js:820:19
  shellHelper@src/mongo/shell/utils.js:710:15
  @(shellhelp2):1:1
  > show dbs;
  2019-03-26T10:12:50.307+0800 E QUERY    [thread1] Error: listDatabases failed:{
          "ok" : 0,
          "errmsg" : "not master and slaveOk=false",
          "code" : 13435,
          "codeName" : "NotMasterNoSlaveOk"
  } :
  _getErrorWithCode@src/mongo/shell/utils.js:25:13
  Mongo.prototype.getDBs@src/mongo/shell/mongo.js:65:1
  shellHelper.show@src/mongo/shell/utils.js:820:19
  shellHelper@src/mongo/shell/utils.js:710:15
  @(shellhelp2):1:1
  ```

#### 5.4:初始化：

* `rs.initiate()`

* 初始化完成后，提示以下信息：

  ```json
  > rs.initiate(); 
  {
          "info2" : "no configuration specified. Using a default configuration for the set",
          "me" : "192.168.66.147:27017", //
          "ok" : 1,
          "operationTime" : Timestamp(1553579849, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553579849, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  ```

#### 5.5:查看当前的状态：

* `rs.status()`

* 查看后的状态如下：

  ```json
  rs0:PRIMARY> rs.status();
  {
          "set" : "rs0",
          "date" : ISODate("2019-03-26T05:58:16.001Z"),
          "myState" : 1,
          "term" : NumberLong(1),
          "heartbeatIntervalMillis" : NumberLong(2000),
          "optimes" : {
                  "lastCommittedOpTime" : {
                          "ts" : Timestamp(1553579891, 1),
                          "t" : NumberLong(1)
                  },
                  "readConcernMajorityOpTime" : {
                          "ts" : Timestamp(1553579891, 1),
                          "t" : NumberLong(1)
                  },
                  "appliedOpTime" : {
                          "ts" : Timestamp(1553579891, 1),
                          "t" : NumberLong(1)
                  },
                  "durableOpTime" : {
                          "ts" : Timestamp(1553579891, 1),
                          "t" : NumberLong(1)
                  }
          },
          "members" : [
                  {
                          "_id" : 0,
                          "name" : "192.168.66.147:27017",
                          "health" : 1,
                          "state" : 1,
                          "stateStr" : "PRIMARY", // 主数据库
                          "uptime" : 14246,
                          "optime" : {
                                  "ts" : Timestamp(1553579891, 1),
                                  "t" : NumberLong(1)
                          },
                          "optimeDate" : ISODate("2019-03-26T05:58:11Z"),
                          "infoMessage" : "could not find member to sync from",
                          "electionTime" : Timestamp(1553579849, 2),
                          "electionDate" : ISODate("2019-03-26T05:57:29Z"),
                          "configVersion" : 1,
                          "self" : true
                  }
          ],
          "ok" : 1,
          "operationTime" : Timestamp(1553579891, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553579891, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  rs0:PRIMARY>
  ```

#### 5.6:添加副本集：

* 添加副本集：`rs.add('192.168.66.174:27018')`

  ```json
  rs0:PRIMARY> rs.add('192.168.66.174:27018') // 
  {
          "ok" : 1,
          "operationTime" : Timestamp(1553607377, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553607377, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  ```

  ```json
  // 添加完副本集后，再次查看一下状态， 会发现已经配置了主从服务器
  rs0:PRIMARY> rs.status()
  {
          "set" : "rs0",
          "date" : ISODate("2019-03-26T13:38:03.658Z"),
          "myState" : 1,
          "term" : NumberLong(1),
          "heartbeatIntervalMillis" : NumberLong(2000),
          "optimes" : {
                  "lastCommittedOpTime" : {
                          "ts" : Timestamp(1553607478, 1),
                          "t" : NumberLong(1)
                  },
                  "readConcernMajorityOpTime" : {
                          "ts" : Timestamp(1553607478, 1),
                          "t" : NumberLong(1)
                  },
                  "appliedOpTime" : {
                          "ts" : Timestamp(1553607478, 1),
                          "t" : NumberLong(1)
                  },
                  "durableOpTime" : {
                          "ts" : Timestamp(1553607478, 1),
                          "t" : NumberLong(1)
                  }
          },
          "members" : [
                  {
                          "_id" : 0,
                          "name" : "192.168.66.147:27017",
                          "health" : 1,
                          "state" : 1,
                          "stateStr" : "PRIMARY", // 主服务器
                          "uptime" : 498,
                          "optime" : {
                                  "ts" : Timestamp(1553607478, 1),
                                  "t" : NumberLong(1)
                          },
                          "optimeDate" : ISODate("2019-03-26T13:37:58Z"),
                          "electionTime" : Timestamp(1553607196, 2),
                          "electionDate" : ISODate("2019-03-26T13:33:16Z"),
                          "configVersion" : 2,
                          "self" : true
                  },
                  {
                          "_id" : 1,
                          "name" : "192.168.66.147:27018",
                          "health" : 1,
                          "state" : 2,
                          "stateStr" : "SECONDARY", // 从服务器
                          "uptime" : 106,
                          "optime" : {
                                  "ts" : Timestamp(1553607478, 1),
                                  "t" : NumberLong(1)
                          },
                          "optimeDurable" : {
                                  "ts" : Timestamp(1553607478, 1),
                                  "t" : NumberLong(1)
                          },
                          "optimeDate" : ISODate("2019-03-26T13:37:58Z"),
                          "optimeDurableDate" : ISODate("2019-03-26T13:37:58Z"),
                          "lastHeartbeat" : ISODate("2019-03-26T13:38:03.653Z"),
                          "lastHeartbeatRecv" : ISODate("2019-03-26T13:38:03.421Z"),
                          "pingMs" : NumberLong(0),
                          "syncingTo" : "192.168.1.2:27017",
                          "configVersion" : 2
                  }
          ],
          "ok" : 1, // 1表示成功
          "operationTime" : Timestamp(1553607478, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553607478, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  rs0:PRIMARY>
  ```

#### 5.7:连接从服务器：

* 此处设置192.168.66.147:27018为`从服务器`。

* 执行以下命令：

  ```json
  C:\Users\lh9>mongo --host 192.168.66.147 --port 27018   // 指定IP地址、端口 配置从服务器
  MongoDB shell version v3.6.4
  connecting to: mongodb://192.168.66.147:27018/
  MongoDB server version: 3.6.4
  Server has startup warnings:
  rs0:SECONDARY>  // 连接完从服务器后，仔细观察这里的变化  
  ```

#### 5.8:配置从服务器：

```json
rs0:SECONDARY> rs.slaveOk()  // 在从服务器输入命令
rs0:SECONDARY>
```

#### 5.9：像主服务器插入数据，在从服务器中查询主服务插入的数据：

* 首先查看从服务的数据库：

  ```json
  rs0:SECONDARY> show dbs; // 可以看到目前的数据库
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  rs0:SECONDARY>
  ```

* 在主服务器上进行操作，创建数据库、集合、插入文档：

  ```json
  rs0:PRIMARY> show dbs;  // 主服务器目前的数据库
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  
  rs0:PRIMARY> use cow  // 切换数据库
  switched to db cow
  
  rs0:PRIMARY> db.cow.insert({"name": "张三", "age": 18});  // 在主服务器插入文档
  WriteResult({ "nInserted" : 1 })
  
  rs0:PRIMARY> db.cow.find() // 查看刚创建集合中的文档
  { "_id" : ObjectId("5c9a2e883b041e86cda061ae"), "name" : "张三", "age" : 18 }
  ```

* 切换到从服务器，进行读取主服务器刚创建的数据库、集合以及插入的文档：

  ```json
  rs0:SECONDARY> show dbs; // 查看从服务器数据库
  admin   0.000GB
  config  0.000GB
  cow     0.000GB  // 在主服务器新创建的，已经在从服务生成
  local   0.000GB
  
  rs0:SECONDARY> use cow  // 进入主服务创建的数据库
  switched to db cow
  
  rs0:SECONDARY> show collections; 
  cow
  rs0:SECONDARY> db.cow.find();  // 可以读取到主服务插入的文档，说明集群搭建成功！
  { "_id" : ObjectId("5c9a2e883b041e86cda061ae"), "name" : "张三", "age" : 18 }
  ```

#### 5.10:其他说明：

* `主服务器宕机后，从服务器会当自动切换成主服务器`。

* 关闭主服务器后，再重新启动，会发现原来的从服务器变为了主服务器，新启动的服务器（原来的主服务器）变为了从服务器。

* 在主服务器上，删除从节点：

  ```json
  rs0:PRIMARY> rs.remove('192.168.66.147:27017') // 因为现在27017 是从服务器， 27018是主服务
  {
          "ok" : 1,
          "operationTime" : Timestamp(1553609662, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553609662, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  ```

  ```json
  rs0:PRIMARY> rs.status() // 在查看状态，发现只有27018主服务器的节点，已经将从节点移除
  {
          "set" : "rs0",
          "date" : ISODate("2019-03-26T14:14:28.141Z"),
          "myState" : 1,
          "term" : NumberLong(2),
          "heartbeatIntervalMillis" : NumberLong(2000),
          "optimes" : {
                  "lastCommittedOpTime" : {
                          "ts" : Timestamp(1553609662, 1),
                          "t" : NumberLong(2)
                  },
                  "readConcernMajorityOpTime" : {
                          "ts" : Timestamp(1553609662, 1),
                          "t" : NumberLong(2)
                  },
                  "appliedOpTime" : {
                          "ts" : Timestamp(1553609662, 1),
                          "t" : NumberLong(2)
                  },
                  "durableOpTime" : {
                          "ts" : Timestamp(1553609662, 1),
                          "t" : NumberLong(2)
                  }
          },
          "members" : [
                  {
                          "_id" : 1,
                          "name" : "192.168.66.147:27018",
                          "health" : 1,
                          "state" : 1,
                          "stateStr" : "PRIMARY",
                          "uptime" : 2621,
                          "optime" : {
                                  "ts" : Timestamp(1553609662, 1),
                                  "t" : NumberLong(2)
                          },
                          "optimeDate" : ISODate("2019-03-26T14:14:22Z"),
                          "electionTime" : Timestamp(1553609416, 1),
                          "electionDate" : ISODate("2019-03-26T14:10:16Z"),
                          "configVersion" : 3,
                          "self" : true
                  }
          ],
          "ok" : 1,
          "operationTime" : Timestamp(1553609662, 1),
          "$clusterTime" : {
                  "clusterTime" : Timestamp(1553609662, 1),
                  "signature" : {
                          "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                          "keyId" : NumberLong(0)
                  }
          }
  }
  ```



## 九、备份与恢复：

### 1.备份：

* 首先确保mongod服务器启动，并创建好将备份的文件目录[新创建的：D:\mongodb\bak]。

* 语法：`mongodump -h dbhost -d dbname -o dbdirectory`

  * -h：服务器地址，也可以指定端口号
  * -d：需要备份的数据库名称
  * -o：备份的数据存放位置，此目录中存放的备份出来的数据

* 举个栗子：

  ```json
  // 管理员身份进行操作 cow:数据库的名称  -o 后面跟的就是备份出来数据的存储目录
  C:\WINDOWS\system32>mongodump -h 127.0.0.1 -d cow -o D:\mongodb\bak
  2019-03-26T22:39:05.784+0800    writing cow.singer to
  2019-03-26T22:39:05.862+0800    writing cow.course to
  2019-03-26T22:39:05.862+0800    writing cow.admin to
  2019-03-26T22:39:05.875+0800    done dumping cow.admin (4 documents)
  2019-03-26T22:39:05.878+0800    done dumping cow.course (7 documents)
  2019-03-26T22:39:05.878+0800    done dumping cow.singer (15 documents)
  ```

* 可以去bak文件夹中，查看刚刚备份cow数据库的文档。



### 2.恢复：

* 先登录准备将文档内容恢复到的数据库:

  ```json
  // 管理员身份登陆
  C:\WINDOWS\system32>mongo
  MongoDB shell version v3.6.4
  connecting to: mongodb://127.0.0.1:27017
  MongoDB server version: 3.6.4
  Server has startup warnings:
  ```

  ```json
  > show dbs;
  admin   0.000GB
  bak     0.000GB // 看数据库中的文档
  config  0.000GB
  local   0.000GB
  
  > use bak  // 切换到bak 数据库
  switched to db bak
  
  > show collections  // 查看该数据库下的集合
  course
  
  > db.course.find()  // 仅有一条文档
  { "_id" : ObjectId("5c9a3bde787acd37b275f815"), "name" : "张三", "age" : 18 }
  >
  ```

* 语法：`mongorestore -h dbhost -d dbname --dir dbdirectory `

  * -h：服务器地址
  * -d：需要恢复的数据库实例
  * --dir：备份数据所在位置

* 举个栗子：

  ```json
   // 管理员身份执行
  C:\WINDOWS\system32>mongorestore -h 127.0.0.1 -d bak --dir D:\mongodb\bak\cow
  
  2019-03-26T22:58:43.758+0800    the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
  2019-03-26T22:58:43.785+0800    building a list of collections to restore from D:\mongodb\bak\cow dir
  2019-03-26T22:58:43.791+0800    reading metadata for bak.singer from D:\mongodb\bak\cow\singer.metadata.json
  2019-03-26T22:58:43.791+0800    reading metadata for bak.course from D:\mongodb\bak\cow\course.metadata.json
  2019-03-26T22:58:43.792+0800    reading metadata for bak.admin from D:\mongodb\bak\cow\admin.metadata.json
  2019-03-26T22:58:43.793+0800    restoring bak.course from D:\mongodb\bak\cow\course.bson
  2019-03-26T22:58:43.829+0800    restoring bak.singer from D:\mongodb\bak\cow\singer.bson
  2019-03-26T22:58:43.861+0800    restoring bak.admin from D:\mongodb\bak\cow\admin.bson
  2019-03-26T22:58:43.861+0800    no indexes to restore
  2019-03-26T22:58:43.862+0800    finished restoring bak.singer (15 documents)
  2019-03-26T22:58:43.862+0800    no indexes to restore
  2019-03-26T22:58:43.862+0800    finished restoring bak.admin (4 documents)
  2019-03-26T22:58:43.862+0800    no indexes to restore
  2019-03-26T22:58:43.862+0800    finished restoring bak.course (7 documents)
  2019-03-26T22:58:43.863+0800    done
  ```

* 查看刚刚恢复的数据，是否到指定的数据库中：

  ```json
  C:\WINDOWS\system32>mongo
  MongoDB shell version v3.6.4
  connecting to: mongodb://127.0.0.1:27017
  MongoDB server version: 3.6.4
  Server has startup warnings:
  
  > show dbs
  admin   0.000GB
  bak     0.000GB  // 将恢复到这个数据库中
  config  0.000GB
  local   0.000GB
  
  > use bak  // 切到该数据库
  switched to db bak
  
  > show collections  // 发现，已经将数据恢复到指定的数据库中
  admin  // 新增的集合
  course // 之前该数据库下有该集合名，恢复的数据也有course集合，所以将恢复的文档追加到其中
  singer  // 新增的集合
  ```

  ```json
  > db.course.find().pretty() // 查看一下course文档
  { "_id" : ObjectId("5c9a3bde787acd37b275f815"), "name" : "张三", "age" : 18 } // 原本有的
  
  // 以下都是恢复时追加其中
  { 
          "_id" : ObjectId("5c960be9b96cea18058444de"),
          "name" : "上海财经大学课程",
          "price" : 888,
          "desc" : "这是上海财经大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c07b96cea18058444df"),
          "name" : "哈尔滨工业大学课程",
          "price" : 288,
          "desc" : "这是哈尔滨工业大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c16b96cea18058444e0"),
          "name" : "北京大学",
          "price" : 2288,
          "desc" : "这是北京大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c960c34b96cea18058444e1"),
          "name" : "沈阳医药大学",
          "price" : 388,
          "desc" : "这是沈阳医药大学的考研课程！"
  }
  {
          "_id" : ObjectId("5c9622cab96cea18058444e2"),
          "name" : "上海财经大学",
          "price" : 888,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学"
          ]
  }
  {
          "_id" : ObjectId("5c9622e0b96cea18058444e3"),
          "name" : "北京大学",
          "price" : 688,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学",
                  "经济学"
          ]
  }
  {
          "_id" : ObjectId("5c9622fcb96cea18058444e4"),
          "name" : "哈尔滨工业大学",
          "price" : 588,
          "discipline" : [
                  "哲学",
                  "建筑学",
                  "体育学",
                  "经济学",
                  "心理学"
          ]
  }
  ```

  



