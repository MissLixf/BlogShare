## 安装配置

### haystack

`pip install django-haystack`

添加haystack 到 INSTALLED_APPS

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',

    # Added.
    'haystack',

    # Then your usual apps...
    'blog',
]
```

### 安装搜索引擎

#### 安装Java 环境

Elasticsearch  要求Java8，推荐使用Oracle JDK version 1.8.0_131 ，安装参考甲骨文官网：http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html

Java SDK 8 下載地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

下載下来rpm包后（测试服~下有这个包），执行安装：

```shell
rpm -ivh jdk-8u181-linux-x64.rpm
```

然后java -version 检查版本。

#### 配置JAVA环境变量

```bash
# 1.修改 /etc/profile
在打开的文件末尾添加如下内容：
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64  # jdk的实际安装目录
export JRE_HOME=/usr/java/jdk1.8.0_181-amd64/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH

# 2. 生效
source /etc/profile 　　　#使配置文件立即生效

# 3. 注意事项：
# 在linux下用冒号“:”来分隔路径 ，windows下是“；”。
# $JAVA_HOME，$PATH，$CLASSPATH是用来引用原来的环境变量的值 。
# 在设置环境变量时特别要注意不能把原来的值给覆盖掉了。
# CLASSPATH中当前目录“.”不能丢
```



#### 配置[elasticsearch](https://www.elastic.co/downloads/elasticsearch)  环境

官网文档地址：https://www.elastic.co/guide/en/elasticsearch/reference

中文文档参考：https://www.elastic.co/guide/cn/elasticsearch/guide/current/running-elasticsearch.html

安装：

通过压缩包：（这种方式后面运行时容易报错）

https://www.elastic.co/guide/en/elasticsearch/reference/5.6/zip-targz.html#install-targz

解压后cd 进去并启动（会有各种意想不到的错误！）

`cd elasticsearch-5.6.10/bin ` `./elasticsearch`



通过RPM(推荐)

Create a file called `elasticsearch.repo` in the `/etc/yum.repos.d/` directory for RedHat based distributions（使用于centos）, or in the `/etc/zypp/repos.d/` directory for OpenSuSE based distributions, containing: 

```ini
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
; 如果gpgkey总是获取失败，可以修改gpgcheck=0, 选择信任 repo
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

然后执行：`yum install elasticsearch` 即可。个人将上面所有的的6.x改为5.x，也安装成功了。

安装成功过后，执行`systemctl start elasticsearch` 即可启动服务。启动服务时，如果提示内存不足，修改`/etc/elasticsearch/jvm.options`如下：（注意，不能写1.5g这种格式，报错）

```shell
-Xms512m
-Xmx1g
```

打开新的终端，执行以下命令查看是否启动成功：

```shell
curl 'http://localhost:9200/?pretty'
```

另外如果通过putty设置了本地代理通道，那么在本机访问http://localhost:9200/?pretty，也应该能得到一样的结果。（访问失败折磨良久：最终参考https://stackoverflow.com/questions/31677563/connection-refused-error-on-elastic-search，的设置，解决了。）

add following to elasticsearch.yml

```yaml
network:
  host: 0.0.0.0
http:
  port: 9200
```

看到如下信息即可：

```json
{
  "name" : "4IOXzCZ",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Ch3VidM1SQyY0ajWwGnrbQ",
  "version" : {
    "number" : "5.6.10",
    "build_hash" : "b727a60",
    "build_date" : "2018-06-06T15:48:34.860Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

另外，rpm 安装方式，配置目录在：`/etc/elasticsearch`

日志目录：`/var/log/elasticsearch/`

#### 安装head插件（可跳过）

head插件用于提供一个可视化的界面。新版的elasticsearch中，该服务不再作为plugins提供，需要单独安装。head基于node，通过grunt执行。所以需要安装node环境。

https://www.jianshu.com/p/75bbafd6b22d

#### python安装 elasticsearch backend（重要）

服务器装的ES5搜索引擎, 安装django-haystack测试时，总是被拒。需要安装这个包：

django-haystack-elasticsearch： https://github.com/CraveFood/django-haystack-elasticsearch
并按照Elasticsearch 5.x进行配置即可。

### 配置搜索引擎

在项目的settings中：

```python
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack_elasticsearch.elasticsearch5.Elasticsearch5SearchEngine',
        # 'URL': 'http://localhost:9200/',  # 正式服配置
        'URL': 'http://localhost:9300/',  # 测试服配置
        'INDEX_NAME': 'haystack',
    },
}
```

## 创建 SearchIndexes

Haystack 通过`SearchIndex` 来决定索引范围和处理数据，它基于字段，通过它来操作和存储数据。通常需要为每一个需要索引的Model，创建一个独一无二的SearchIndex。

创建方式：

* 子类化 `indexes.SearchIndex` & `indexes.Indexable`
*  定义你想存储的字段
* 定义一个 `get_model` 方法 

在app的根目录下新建search_indexes.py 脚本，并创建QuestionIndex，如下：

```python
from haystack import indexes
from forum_api_service import models


class QuestionIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True)  # document=True属性定义的字段名不要轻易更改，建议使用text命名
    detail = indexes.CharField(model_attr='detail')
    university = indexes.CharField(model_attr='university')
    discipline = indexes.CharField(model_attr='discipline')

    def get_model(self):
        return models.Question
```

说明：

* search_indexes.py Haystack 会自动搜寻该模块
* 每个SearchIndex类中，有且只有一个字段能定义document=True属性，这将告诉Haystach和搜索引擎，该字段是搜索的主字段。

## 创建视图

### 定义路由

```python
url(r'^v1/search/', include('haystack.urls'))，
```

## 重建索引

将数据库中的数据放入search index中，只需执行Haystack提供的命令即可。

```shell
./manage.py rebuild_index
```

### 自动更新

所以内容只有在执行update_index 或者 rebuild_index 时才会更新。因此应该创建一个定时任务 update_index，来进行更新。或者利用`RealtimeSignalProcessor`  来自动处理更新和删除。

```python
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```

但是这可能导致接口响应速度变慢，有时需要等待索引完成，才能返回结果。

## SearchQuerySet

SearchQuerySet 的使用方式与 QuerySet 类似，搜索接口都是通过SearchQuerySet 对象进行，并返回一个有序集合。支持链式调用，且具有惰性。基本使用如下：

```python
from haystack.query import SearchQuerySet

res = SearchQuerySet().all()
res = SearchQuerySet().filter(content='hello')
res = SearchQuerySet().filter(content='hello world')
res = SearchQuerySet().exclude(content='hello').filter(content='world')
res = SearchQuerySet().order_by('-pub_date')[:5]
```

集合中的元素就是SearchResult对象，可以理解为一个类似于model中的一条记录。

### 输入类型

但是SearchQuerySet 提供了输入类型(Input Types)，比如： `AutoQuery`, `Exact`, `Clean`  

```python

```

### content 快捷方式

使用content可以进行简化搜索，无需指定字段名。自动从所有document=True的字段中进行搜索。

```python
from haystack.query import SearchQuerySet

# This searches whatever fields were marked ``document=True``.
results = SearchQuerySet().exclude(content='hello')
```

## 分词

ik 分词，支持中文。https://github.com/medcl/elasticsearch-analysis-ik 安装说明。

下载对应的压缩包，解压到/etc/elasticsearch/plugins/ik/目录下即可（需手动创建目录），重启es即可。

注意，ik的版本选择取决于es的版本，比如按照的是:`elasticsearch.noarch 0:5.6.13-1`，那么选择`v5.6.13`的ik

以下不需要。

通过手动安装，checkout 和当前es版本匹配的版本。

手动安装ik分词需要Maven(java下的项目管理及构建工具)： https://www.cnblogs.com/freeweb/p/5241013.html

https://github.com/medcl/elasticsearch-analysis-ik



Maven 配置环境变量（依赖于JAVA环境变量）：

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export JRE_HOME=/usr/java/jdk1.8.0_181-amd64/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH:$MAVEN_HOME/bin

MAVEN_HOME=/usr/local/maven
export MAVEN_HOME
```

配置完之后，执行 `mvn -v ` 检查是否安装成功

## 测试

ES返回数据如下：

```json
{
	"took": 180,
	"timed_out": false,
	"_shards": {
		"total": 5,
		"successful": 5,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": 7,
		"max_score": 1.0,
		"hits": [{
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.3",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "3\n台湾当局错误政策导致市场“堵塞” 果农弃收让猴子吃到饱\n北京大学\n力学（可授工学、理学学位）",
				"django_id": "3",
				"id": "forum_api_service.question.3"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.5",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "5\n侠客岛：这个县级市的经验，为何让高层如此看重？\n北京师范大学\n控制科学与工程",
				"django_id": "5",
				"id": "forum_api_service.question.5"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.2",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "2\nAyhan_huang的博客\n北京大学\n马克思主义理论",
				"django_id": "2",
				"id": "forum_api_service.question.2"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.4",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "4\n台湾当局错误政策导致市场“堵塞” 果农弃收让猴子吃到饱\n北京大学\n力学（可授工学、理学学位）",
				"django_id": "4",
				"id": "forum_api_service.question.4"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.6",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "6\n侠客岛：这个县级市的经验，为何让高层如此看重？\n北京印刷学院\n中国语言文学",
				"django_id": "6",
				"id": "forum_api_service.question.6"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.7",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "7\n侠客岛：好奇怪的名字？\n北京印刷学院\n中国语言文学",
				"django_id": "7",
				"id": "forum_api_service.question.7"
			}
		}, {
			"_index": "haystack",
			"_type": "modelresult",
			"_id": "forum_api_service.question.8",
			"_score": 1.0,
			"_source": {
				"django_ct": "forum_api_service.question",
				"text": "8\n侠客岛：好奇怪的名字？\n清华大学\n应用经济学",
				"django_id": "8",
				"id": "forum_api_service.question.8"
			}
		}]
	}
}
```

