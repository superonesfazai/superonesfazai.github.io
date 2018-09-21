---
layout: post
title: rpc远程过程调用之利用rabbitmq实现rpc
---

# RabbitMQ
RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种，最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

## 安装RabbitMQ
`$ brew install rabbitmq`

- 登录管理页面
```bash
http://localhost:15672/

# 账号密码初始默认都为guest
```
- 启动
```bash
# rabbitmq的安装目录
$ cd /usr/local/Cellar/rabbitmq/3.7.7_1
# 启动
$ sbin/rabbitmq-server
```

## server.py
```python
# coding:utf-8

'''
@author = super_fazai
@File    : server.py
@connect : superonesfazai@gmail.com
'''

# import pika
from pika import (
    BasicProperties,
    BlockingConnection,
    ConnectionParameters,)

connection = BlockingConnection(ConnectionParameters(host='localhost'))
channel = connection.channel()
channel.queue_declare(queue='rpc_queue')

def a():
    return "a"

def b():
    return "b"

def on_request(ch, method, props, body):
    funname = props.app_id
    if funname == "a":
        response = a()
    elif funname == "b":
        response = b()
    else:
        raise ValueError('未定义该{}方法!'.format(funname))

    ch.basic_publish(
        exchange='',
        routing_key=props.reply_to,
        properties=BasicProperties(correlation_id=props.correlation_id),
        body=str(response))
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(on_request, queue='rpc_queue')
print("[x] 等待 rpc requests...")
channel.start_consuming()
```

## client.py
```python
# coding:utf-8

'''
@author = super_fazai
@File    : client.py
@connect : superonesfazai@gmail.com
'''

# import pika
from pika import (
    BlockingConnection,
    ConnectionParameters,
    BasicProperties,
)
from uuid import uuid4

class RpcClient(object):
    def __init__(self):
        self.connection = BlockingConnection(ConnectionParameters(host='localhost'))
        self.channel = self.connection.channel()
        result = self.channel.queue_declare(exclusive=True)
        self.callback_queue = result.method.queue
        self.channel.basic_consume(self.on_response, no_ack=True,
                                   queue=self.callback_queue)

    def on_response(self, ch, method, props, body):
        if self.corr_id == props.correlation_id:
            self.response = body

    def call(self, name):
        '''
        接收参数name作为被调用的远程函数的名字，通过app_id传给服务端程序
        :param name:
        :return:
        '''
        self.response = None
        self.corr_id = str(uuid4())
        self.channel.basic_publish(
            exchange='',
            routing_key='rpc_queue',
            properties=BasicProperties(
               reply_to=self.callback_queue,
               correlation_id=self.corr_id,
               app_id=str(name),
            ),
            body="request")

        while self.response is None:
            self.connection.process_data_events()

        return str(self.response)

rpc = RpcClient()
print("[x] Requesting")
index = 0
while True:
    if index > 1000:
        break
    response = rpc.call("b")
    print("[.] Got %r" % response)
    index += 1
```