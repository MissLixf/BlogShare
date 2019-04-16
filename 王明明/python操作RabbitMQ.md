## RabbitMQ介绍

RabbitMQ是一个由erlang开发的AMQP（Advanced Message Queue ）的开源实现的产品，RabbitMQ是一个消息代理，从“生产者”接收消息并传递消息至“消费者”，期间可根据规则路由、缓存、持久化消息。“生产者”也即message发送者以下简称P，相对应的“消费者”乃message接收者以下简称C，message通过queue由P到C，queue存在于RabbitMQ，可存储尽可能多的message，多个P可向同一queue发送message，多个C可从同一个queue接收message

![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926856-1132336716.jpg)

### Message

RabbitMQ 转发的二进制对象，包括Headers（头）、Properties （属性）和 Data （数据），其中数据部分不是必要的。

### Consumer 

使用队列 Queue 从 Exchange 中获取消息的应用。

### Producer

消息的生产者，负责产生消息并把消息发到交换机

### Exchange 

负责接收生产者的消息并把它转到到合适的队列

### Queue 

一个存储Exchange 发来的消息的缓冲，并将消息主动发送给Consumer，或者 Consumer 主动来获取消息

### Binding 

队列 和 交换机 之间的关系。Exchange 根据消息的属性和 Binding 的属性来转发消息。绑定的一个重要属性是 binding_key

### Connection （连接）和 Channel （通道）

生产者和消费者需要和 RabbitMQ 建立 TCP 连接。一些应用需要多个connection，为了节省TCP 连接，可以使用 Channel，它可以被认为是一种轻型的共享 TCP 连接的连接。

## 简单消息队列

一个Product向queue发送一个message，一个Client从该queue接收message并打印

![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926294-627960883.png)

### Product

```python
import pika   

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))     #定义连接池
channel = connection.channel()          
channel.queue_declare(queue='test')    #声明队列以向其发送消息消息
channel.basic_publish(exchange='', routing_key='test', body='Hello World!')  #注意当未定义exchange时，routing_key需和queue的值保持一致
print('send success msg to rabbitmq')
connection.close()   #关闭连接
```

### Consume

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.queue_declare(queue='test')


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)


channel.basic_consume(callback,queue='test',no_ack=True)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

### 结果

```
#product端：
send success msg to rabbitmq

#client端：
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received b'Hello World!'
```

## 消息确认

当客户端从队列中取出消息之后，可能需要一段时间才能处理完成，如果在这个过程中，客户端出错了，异常退出了，而数据还没有处理完成，那么非常不幸，这段数据就丢失了，因为rabbitmq默认会把此消息标记为已完成，然后从队列中移除，

消息确认是客户端从rabbitmq中取出消息，并处理完成之后，会发送一个ack告诉rabbitmq，消息处理完成，当rabbitmq收到客户端的获取消息请求之后，或标记为处理中，当再次收到ack之后，才会标记为已完成，然后从队列中删除。当rabbitmq检测到客户端和自己断开链接之后，还没收到ack，则会重新将消息放回消息队列，交给下一个客户端处理，保证消息不丢失，也就是说，RabbitMQ给了客户端足够长的时间来做数据处理。

在客户端使用no_ack来标记是否需要发送ack，默认是False，开启状态

加入ack

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.queue_declare(queue='test')


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    time.sleep(5)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息




channel.basic_consume(callback,queue='test',no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

## 改变消息获取顺序

默认消息队列里的数据是按照顺序被消费者拿走

例如：消费者1 去队列中获取 奇数 序列的任务，消费者2去队列中获取 偶数 序列的任务。

channel.basic_qos(prefetch_count=1) 表示谁来谁取，不再按照奇偶数排列

## 消息持久化

消息确认机制使得客户端在崩溃的时候，服务端消息不丢失，但是如果rabbitmq奔溃了呢？该如何保证队列中的消息不丢失？ 

此就需要product在往队列中push消息的时候，告诉rabbitmq，此队列中的消息需要持久化

用到的参数：durable=True

```python
channel.basic_publish(exchange='',  
                      routing_key="test",  
                      body=message,  
                      properties=pika.BasicProperties(  
                         delivery_mode = 2, # make message persistent  
                      ))  
```

Product

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))     #定义连接池
channel = connection.channel()          #声明队列以向其发送消息消息

channel.queue_declare(queue='test_persistent',durable=True)
for i in range(10):
    # 消息持久化
    channel.basic_publish(exchange='', routing_key='test_persistent', body=str(i),properties=pika.BasicProperties(delivery_mode=2))
    print('send success msg[%s] to rabbitmq' %i)
connection.close()   #关闭连接
```

Consume

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()
# 消息持久化
channel.queue_declare(queue='test_persistent',durable=True)


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    #time.sleep(5)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息



channel.basic_qos(prefetch_count=1)  # 表示谁来谁取
channel.basic_consume(callback,queue='test_persistent',no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
注意：client端也需配置durable=True，否则将报下面错误：

pika.exceptions.ChannelClosed: (406, "PRECONDITION_FAILED - parameters for queue 'test_persistent' in vhost '/' not equivalent")
```

## 使用Exchange

exchanges主要负责从product那里接受push的消息，根据product定义的规则，投递到queue中，是product和queue的中间件
![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926419-1937096317.jpg)￼

- Exchange 类型
  - direct 关键字类型
  - topic 模糊匹配类型
  - fanout 广播类型

- 使用fanout实现发布订阅者模型

![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926778-1544074422.jpg)

发布订阅和简单的消息队列区别在于，发布订阅会将消息发送给所有的订阅者，而消息队列中的数据被消费一次便消失。所以，RabbitMQ实现发布和订阅时，会为每一个订阅者创建一个队列，而发布者发布消息时，会将消息放置在所有相关队列中

订阅者

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.exchange_declare(exchange='test123',type='fanout')  #定义一个exchange ,类型为fanout
rest = channel.queue_declare(exclusive=True)   #创建一个随机队列,并启用exchange
queue_name = rest.method.queue          #获取队列名
channel.queue_bind(exchange='test123',queue=queue_name)   #将随机队列名和exchange进行绑定


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    time.sleep(1)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息



channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,queue=queue_name,no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

发布者

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))     #定义连接池
channel = connection.channel()          #声明队列以向其发送消息消息

channel.exchange_declare(exchange='test123',type='fanout')
for i in range(10):
    channel.basic_publish(exchange='test123', routing_key='', body=str(i),properties=pika.BasicProperties(delivery_mode=2))
    print('send success msg[%s] to rabbitmq' %i)
connection.close()   #关闭连接
```

- 使用direct 实现根据关键字发布消息

![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926653-1969440238.jpg)

消息发布订阅者模型是发布者发布一条消息，所有订阅者都可以收到，现在rabbitmq还支持根据关键字发送，在发送消息的时候使用routing_key参数指定关键字，rabbitmq的exchange会判断routing_key的值，然后只将消息转发至匹配的队列，

注意，此时需要订阅者先创建队列

配置参数为exchange的type＝direct，然后定义routing_key即可

订阅者1： 订阅error,warning,info

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.exchange_declare(exchange='test321',type='direct')  #定义一个exchange ,类型为fanout
rest = channel.queue_declare(exclusive=True)   #创建一个随机队列,并启用exchange
queue_name = rest.method.queue          #获取队列名

severities = ['error','warning','info']   #定义三个routing_key

for severity in severities:
    channel.queue_bind(exchange='test321', routing_key=severity,queue=queue_name)


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    time.sleep(1)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息


channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,queue=queue_name,no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

订阅者2：订阅error,warning

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.exchange_declare(exchange='test321',type='direct')  #定义一个exchange ,类型为fanout
rest = channel.queue_declare(exclusive=True)   #创建一个随机队列,并启用exchange
queue_name = rest.method.queue          #获取队列名

severities = ['error','warning']   #定义两个routing_key

for severity in severities:
    channel.queue_bind(exchange='test321', routing_key=severity,queue=queue_name)


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    time.sleep(1)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息



channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,queue=queue_name,no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

发布者

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))     #定义连接池
channel = connection.channel()          #声明队列以向其发送消息消息

channel.exchange_declare(exchange='test321',type='direct')
channel.basic_publish(exchange='test321', routing_key='info', body='info msg',properties=pika.BasicProperties(delivery_mode=2))  #发送info msg到 info routing_key
channel.basic_publish(exchange='test321', routing_key='error', body='error msg',properties=pika.BasicProperties(delivery_mode=2)) #发送error msg到 error routing_key

print('send success msg[] to rabbitmq')
connection.close()   #关闭连接**
```

- 使用topic实现模糊匹配发布消息

direct实现了根据自定义的routing_key来标示不同的queue，使用topic可以让队列绑定几个模糊的关键字，之后发送者将数据发送到exchange，exchange将传入”路由值“和 ”关键字“进行匹配，匹配成功，则将数据发送到指定队列

```
# 表示可以匹配 0 个 或 多个 单词
*  表示只能匹配 一个 单词

如：
fuzj.test 和fuzj.test.test
fuzj.# 会匹配到 fuzj.test 和fuzj.test.test
fuzj.* 只会匹配到fuzj.test
```

订阅者1: 使用#匹配

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672))
channel = connection.channel()

channel.exchange_declare(exchange='test333',type='topic')  #定义一个exchange ,类型为fanout
rest = channel.queue_declare(exclusive=True)   #创建一个随机队列,并启用exchange
queue_name = rest.method.queue          #获取队列名

channel.queue_bind(exchange='test333', routing_key='test.#',queue=queue_name)


def callback(ch, method, properties, body):
    '''回调函数,处理从rabbitmq中取出的消息'''
    print(" [x] Received %r" % body)
    time.sleep(1)
    ch.basic_ack(delivery_tag = method.delivery_tag)  #发送ack消息



channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,queue=queue_name,no_ack=False)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()    #开始监听 接受消息
```

发布者：

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))     #定义连接池
channel = connection.channel()          #声明队列以向其发送消息消息

channel.exchange_declare(exchange='test333',type='topic')
channel.basic_publish(exchange='test333', routing_key='test.123', body='test.123 msg',properties=pika.BasicProperties(delivery_mode=2))
channel.basic_publish(exchange='test333', routing_key='test.123.321', body=' test.123.321 msg',properties=pika.BasicProperties(delivery_mode=2))

print('send success msg[] to rabbitmq')
connection.close()   #关闭连接
```

实现RPC
![img](https://images2015.cnblogs.com/blog/952555/201607/952555-20160729110926638-376110352.jpg)

一个RPC请求，客户端发送消息包括两个参数，reply_to:是设置回调队列，correlation_id是唯一请求标识

请求发送到rpc_queue队列

服务端等待rpc_queue中的请求，当得到一个请求，把结果通过reply_to回调队列发送给客户端

客户端等待reply_to队列的结果，当得到一个消息，检查correlation_id，如果和请求匹配就返回response

service

```python
import pika
import subprocess
connection = pika.BlockingConnection(pika.ConnectionParameters(
    host='127.0.0.1', port=5672, ))       #定义连接池

channel = connection.channel()    #创建通道

channel.queue_declare(queue='rpc_queue')            #创建rpc_queue队列

def operating(arg):
    p = subprocess.Popen(arg, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)   #执行系统命令
    res = p.stdout.read()       #取出标准输出
    if not res:                 #判断是否有执行结果
        responses_msg = p.stderr.read()         #没有执行结果则取出标准错误输出
    else:
        responses_msg = res
    return responses_msg

def on_request(ch, method, props, body):
    command = str(body,encoding='utf-8')
    print(" [.] start Processing command : %s" % command)
    response_msg = operating(body)          #调用函数执行命令
    ch.basic_publish(exchange='',
                     routing_key=props.reply_to,
                     properties=pika.BasicProperties(correlation_id = props.correlation_id),body=str(response_msg))
    ch.basic_ack(delivery_tag = method.delivery_tag)


channel.basic_qos(prefetch_count=1)       #消息不平均分配,谁取谁得
channel.basic_consume(on_request, queue='rpc_queue')    #监听队列

print(" [x] Awaiting RPC requests")
channel.start_consuming()
```

client

```python
import pika
import uuid
import time

class FibonacciRpcClient(object):
    def __init__(self):
        self.connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='127.0.0.1',port=5672,))     #定义连接池

        self.channel = self.connection.channel()        #创建通道

        result = self.channel.queue_declare(exclusive=True,auto_delete=True)  #创建客户端短接受服务端回应消息的队列,\exclusive=True表示只队列只允许当前链接进行连接,auto_delete=True表示自动删除
        self.callback_queue = result.method.queue     #获取队列名称

        self.channel.basic_consume(self.on_response, no_ack=True,
                                   queue=self.callback_queue)            #从队列中获取消息

    def on_response(self, ch, method, props, body):
        if self.corr_id == props.correlation_id:     #判断
            self.response = body

    def call(self, n):
        self.response = None
        self.corr_id = str(uuid.uuid4())
        self.channel.basic_publish(exchange='',
                                   routing_key='rpc_queue',
                                   properties=pika.BasicProperties(
                                         reply_to = self.callback_queue,   #回应消息的队列
                                         correlation_id = self.corr_id, #correlation id可以理解为请求的唯一标识码
                                         ),
                                   body=str(n))
        while self.response is None:        #不断从自己监听的队列里取消息,直到取到消息
            self.connection.process_data_events()
        return self.response.decode()

fibonacci_rpc = FibonacciRpcClient()

print(" [x] Requesting server" )
time.sleep(0.1)
while True:
    command = input('>> ')
    response = fibonacci_rpc.call(command)
    print(" [.] Get %r \n" % response)
```