---
title: tornado开发之使用tcpserver和tcpclient实现echo服务器
date: 2020-05-30 11:57:21
tags: tornado
categories: Python
---

本文主要介绍了在tornado框架中使用tcpserver,tcpclient,struct.pack(),struct.unpack实现简单echo服务器的过程。

<!--more-->

在网络通信中，需要发送二进制流数据；struct.pack()函数负责数据组包，即将数据按照规定的传输协议组合起来；struct.unpack()函数负责数据拆包，即按照规定的协议将数据拆分开来。

不多说，具体实现代码咱们来看一下。

tcp客户端代码如下:

```python
class ChatClient(object):
    def __init__(self, host, port):
        self.host = host
        self.port = port

    @gen.coroutine
    def start(self):
        self.stream = yield TCPClient().connect(self.host, self.port)
        while True:
            yield self.send_message()
            yield self.receive_message()

    @gen.coroutine
    def send_message(self):
        # 待发送数据
        msg = input("输入:")
        bytes_msg = bytes(msg.encode("utf-8"))
        # 消息发送者
        chat_id = 10000000
        # 消息接收者
        receive_id = 10000001
        # 消息类型 1-文本 2-图片 3-语音 4-视频 等
        msg_type = 1

        binary_msg = struct.pack("!IIBI"+str(len(msg))+"s", chat_id, receive_id, msg_type, len(msg), bytes_msg)
        # 发送数据
        yield self.stream.write(binary_msg)

    @gen.coroutine
    def receive_message(self):
        """
        接收数据
        :return:
        """
        try:
            logger.debug("receive data ...")
            # 消息发送者 4字节
            sender = yield self.stream.read_bytes(4, partial=True)
            sender = struct.unpack('!I', sender)[0]
            logger.debug("sender:%s", sender)

            # 消息类型 1字节
            msg_type = yield self.stream.read_bytes(1, partial=True)
            msg_type = struct.unpack('!B', msg_type)[0]
            logger.debug("msg_type:%s", msg_type)

            # 消息长度 4字节
            msg_len = yield self.stream.read_bytes(4, partial=True)
            msg_len = struct.unpack('!I', msg_len)[0]
            logger.debug("msg_len:%s", msg_len)

            # 真实数据
            data = yield self.stream.read_bytes(msg_len, partial=True)
            data = struct.unpack("!" + str(msg_len) + "s", data)
            logger.debug("data:%s", data)
        except Exception as e:
            logger.error("tcp client exception:%s", e)


def main():
    c1 = ChatClient("127.0.0.1", 8888)
    c1.start()
    ioloop.IOLoop.instance().start()


if __name__ == '__main__':
    main()
```

tcp服务端代码：

```python
# coding=utf-8


import struct
import logging

from tornado.tcpserver import TCPServer
from tornado.netutil import bind_sockets
from tornado.iostream import StreamClosedError
from tornado import gen
from tornado.ioloop import IOLoop


"""
tcpserver-struct.unpack()拆包
接收数据包格式:消息头+消息体
消息头:消息发送者(4字节)+消息接收者(4字节)+消息类型(1字节)+消息体中数据长度(4字节)
消息体:待接收数据
struct.pack()组包
转发数据包格式:消息头+消息体
消息头:消息发送者(4字节)+消息类型(1字节)+消息体中数据长度(4字节)
消息体:待发送数据
"""


logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)


class ChatServer(TCPServer):

    PORT = 8888
    clients = dict()

    @gen.coroutine
    def handle_stream(self, stream, address):
        """
        数据拆包并解析
        :param stream:
        :param address:
        :return:
        """
        logger.debug("%s已上线", address)
        ChatServer.clients[address] = stream
        while True:
            try:
                # !表示使用大端方式解析数据
                # 消息发送者 4字节
                sender = yield stream.read_bytes(4, partial=True)
                sender = struct.unpack('!I', sender)[0]
                logger.debug("sender:%s", sender)

                # 消息接收者 4字节
                receiver = yield stream.read_bytes(4, partial=True)
                receiver = struct.unpack('!I', receiver)[0]
                logger.debug("receiver:%s", receiver)

                # 消息类型 1字节
                msg_type = yield stream.read_bytes(1, partial=True)
                msg_type = struct.unpack('!B', msg_type)[0]
                logger.debug("msg_type:%s", msg_type)

                # 消息长度 4字节
                msg_len = yield stream.read_bytes(4, partial=True)
                msg_len = struct.unpack('!I', msg_len)[0]
                logger.debug("msg_len:%s", msg_len)

                if msg_type == 1:
                    # 文本信息处理
                    logger.debug("text message ...")
                    self.handle_text_stream(stream, sender, msg_len)
                elif msg_type == 2:
                    logger.debug("picture message ...")
                    self.handle_pic_stream(stream, sender, msg_len)

            except StreamClosedError:
                logger.debug("%s已下线", address)
                del ChatServer.clients[address]
                break

    @gen.coroutine
    def handle_text_stream(self, stream, sender, msg_len):
        """
        处理文本数据
        :param stream:
        :param send_to:
        :param msg_len:
        :return:
        """
        data = yield stream.read_bytes(msg_len, partial=True)
        data = struct.unpack("!"+str(msg_len)+"s", data)
        logger.debug("data:%s", data)
        try:
            # 打包数据,数据格式:数据发送者+数据类型+数据长度+数据体
            binary_msg = struct.pack("!IBI" + str(msg_len) + "s", sender, 1, msg_len, data[0])
            # 发送数据
            yield stream.write(binary_msg)
            logger.debug("="*25)
        except KeyError:
            # 将离线消息保存到数据库
            pass

    @gen.coroutine
    def handle_pic_stream(self, stream, sender, msg_len):
        pass


if __name__ == '__main__':
    sockets = bind_sockets(ChatServer.PORT)
    server = ChatServer()
    server.add_sockets(sockets)
    IOLoop.current().start()
```

以上就是具体的代码实现。

