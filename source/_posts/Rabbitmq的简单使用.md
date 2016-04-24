title: RabbitMQ的简单使用
date: 2015-07-05 20:47:20
tags: [RabbitMQ,系统服务,Python,Pika,消息队列]
categories: 系统服务
---

本文主要对RabbitMQ进行介绍，说明一些基本特性及使用，结合官方文档进行解释说明。

# RabbitMQ简介
RabbitMQ是消息代理(Message Broker)，核心思想是接受和转发消息。

RabbitMQ结构图如下：

![RabbitMQ](http://7xjc5f.com1.z0.glb.clouddn.com/img/rabbitmq.jpg)

RabbitMQ的一些术语：

1. 生产(Producing)：即发送；发送信息的程序被称为生产者(producer)；
2. 队列(Queue)：相当于邮箱，存在于RabbitMQ中，所有的信息流都只存储在队列中。队列没有任何存储的限制；
3. 消费(Consuming)：即接受；等待接受消息的程序被称为消费者(Cosumer)。

说明：

+ 多个生产者可以发送信息到一个队列中，多个消费者可以从一个队列中获取数据；
+ 消费者、生产者以及队列可以不是同一台机器。

# RabbitMQ实例
下面通过一个简单的实例来来说明一个RabbitMQ的使用，即：

+ 生产者通过发送"Hello World"到一个名为"hello"的队列中;
+ 消费者获取到队列中的消息。

## 构建环境
RabbitMQ基于AMQP协议建立的消息队列，在Python中的库的实践选择很多，本教程使用[pika](https://pika.readthedocs.org)进行使用说明，在MAC下完成教程的内容。

1.安装RabbitMq服务器：

``` shell
brew install rabbitmq
ln -sfv /usr/local/opt/rabbitmq/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.rabbitmq.plist

```

2.安装pika进行使用：

``` shell
mkdir Rabbit
cd Rabbit
pip install virtualenv
virtualenv env
source env/bin/active
pip install pika
```

+ 第一、二行创建实例代码放置的文件夹；
+ 第三、四、五行 安装virtualenv并使用创建一个名为env虚拟环境，并激活该环境；
+ 安装pika库。

解释说明：

+ 虚拟环境：使用python会需要使用很多的框架和库，为了方便管理不同项目以及做好各种库的控制，使用虚拟环境可以隔绝其他库的影响，进行干净的使用。

## 发送代码

``` python
import pika                                        
connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='hello')
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!')
connection.close()
```
 
代码解释：

+ 第一行 导入pika库进行使用；
+ 第二行 创建一个阻塞连接(BlockingConnection)，设置连接参数(ConnectionParameters)为'localhost'；
+ 第三行 创建一个虚拟通道(channel)；
+ 第四行 创建一个名为'hello'的队列；
+ 第五行 配置进行送'Hello World!'到'hello'队列中；
+ 第六行 关闭连接。

内容解释：

+ 阻塞：在网络服务中，会涉及到阻塞和非阻塞的概念，阻塞会造成网络拥塞；
+ 阻塞连接(BlockingConnection)：在pika库中定义了五种连接形式异步连接(AsyncoreConnection)、阻塞连接、选择连接(SelectionConnection)、Tornado和Twisted连接，阻塞连接是一个比较简单的连接形式；
+ 连接参数(ConnectionParameters)：在pika库中定义了14种连接参数，对连接的基本信息进行设定；
+ localhost：连接RabbitMQ需要制定地址，在计算机中localhost指代本机，地址为127.0.0.1；
+ queue_declare：要发送消息首先需要建立一个队列，如果没有队列消息就会被抛弃，创建队列时需要指名队列名称；
+ basic_publish：交换机(exchange)在RabbitMQ作为帮助路由转发的作用，由于实例简单，则使用默认的规则。


## 接收代码

``` python
import pika                                      
connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='hello')
def callback(ch, method, properties, body):
    print " [x] Received %r" % (body,)
channel.basic_consume(callback,
                      queue='hello',
                      no_ack=True)
channel.start_consuming()
```

代码解释：

+ 前四行 与发送代码相同，重复定义queue_declare是防止使用中未被创立；
+ 第五、六行 定义一个简单的callback函数来进行处理消息；
+ 第七行 确认接受消息的函数，队列以及是否设置确认(ack)。
+ 第八行 开始消费。

内容解释：

+ callback： 由于队列接受消息会比较复杂，这里简单定义了一个callback函数来模拟，callback作为回调函数实际中应用也是比较多的。

# RabbitMQ特性

RabbitMQ有四个特性：

+ 循环分发(Round-robin Dispatching)
+ 消息确认(Message acknowledgment)
+ 消息持久化(Message durability)
+ 公平分发(Fair dispatch)

## 循环分发
当生产大量信息到一个队列中，如果存在多个消费者，那么这些消费者按顺序依次循环获取队列中消息，直到结束。

## 消息确认
在消费过程中，如果一个消费者突然中止，从队列中消费出的内容也会被丢失。类似TCP一样，为了建立一个可靠的连接，需要通过确认来进行。
即在basic_publish方法中将参数no_ack设为True，默认情况下方法默认开启确认。

## 消息持久化
当队列或者消费者中止掉时，为了不让数据进行丢失，需要对队列及消费过程中进行设置来保证消息的持久化，即在queue_declare方法中设置参数durable为True，在basic_consume方法中对BaiscProperties中的delivery_mode配置为2并传给参数properties使用。

注意：
***对一个已存在的队列，加入durable，不会起作用需要重新定义一个队列。***


## 公平分发

在Rabbitmq中，由于循环分发的机制存在这样一种情况，假设有两个消费者等待队列的处理，由于循环者给，但是奇数的任务都苦活累活，偶数的都是简单的，这样不能提升整体效率，所以来设置让队列等之前的处理完在分配新任务，这样保证一直都有事情且高效。

这属于qos部分即服务质量部分的设置，prefetch_count设置
前面等待的任务数量。


# Exchange

在RabbitMQ中，生产者生成消息进行发送，而消费者进行接受消费，这种模式被称为“出版/订阅”的模式。实际中生产者不是直接讲消息发送到一个队列中。而是通过exchange来讲消息进行发送。

Exchange有几种类型：direct、topic、headers、fanout。

## fanout类型
fanout是分发的意思，所以这个类型就是广播所有消息到所有已知的队列中去。以一个日志系统举例。

1.定义fanout类型：

``` python
channel.exchange_declare(exchange='logs',
                         type='fanout')
```

这里的exchange定义了这个交换器的名字叫log，类型为fanout。

注：在之前例子中将exchange定义为空，会将生产的消息发送到routing_key设定的key中。方法中需要把exchange改成logs。

2.创建临时队列：

```python
result = channel.queue_declare(exclusive=True)
```

在日志系统中，我们不会关系是哪个队列，也就是不在乎队列的名称，我们只需要知道所有的日志信息。 因此通过在queue_declare不加名称来自动生成队列。

同时 当消费者断开连接时也需要队列被删除，因此需要设立 exclusive=True。

3.绑定队列与exchange：

```python
channel.queue_bind(exchange='logs',
                   queue=result.method.queue)
```

这样就能完成一个基本的fanout类型的设置。

## Direct类型

在设置了一个队列后，有时候要对广播的消息中特定内容进行获取，譬如在日志系统中的错误信息进行获取。为了对这些定制化的内容进行获取，这时需要使用exchange中的direct类型。

Direct模式下会根据绑定键匹配到不同的信息分到不同的队列上去，当然在direct模式下可以将一个绑定键绑定到多个队列中，有点类似fanout的模式。

Direct类型绑定：

``` python
channel.queue_bind(exchange=exchange_name,
                   queue=queue_name,
                   routing_key='black')
```

对exchange和队列的绑定可以加入一个参数routing_key来指定到一个队列中，叫做绑定键binding_key。

## Topic类型

有时候对对于一个队列，光根据一个信息进行定制化是不够的，还需要根据其他来建立一个更有弹性的队列，这时候就需要使用Topic类型了。

topic类型，可以过滤出特定的消息发送到不同的队列中。

队列的routing_key是通过单词之间加点来进行设置，长度最大有255比特。它有两种匹配规则：

+ *匹配的是只有一个词语， *.a.*,则匹配 一个词语.a.一个词语的形式，多一个也不行
+ \#匹配任意多个词语。

这样可以让exchange 更加灵活匹配关键词语进行分发消息。


## Headers类型

头部类型的exchange比起routing_key作为信息头部更容易路由多个属性，Headers exchange忽略routing key属性。
当头部值等于绑定指定的值时消息匹配。

可以使用多个header来匹配绑定一个队列到一个Headers类型的Exchange上。


# 消息的属性
AMQP预定义了14个属性，以下几个是平时用的比较多的：

+ delivery_mode: 设置信息持久化（通常设定值为2）。
+ content_type: 描述mime-type的encoding。例如设置为JSON格式：设置property为application/json。
+ reply_to: 指明回调的queue（通常为callback队列的名称）。
+ correlation_id: 在请求中关联处理RPC响应，设置一个唯一值id，在接受响应后可以判断是否是自己的响应。如果不是，就不去处理。


***

版本修改记录：

+ 2015-07-05 完成本篇文章全部内容。 By Octavian