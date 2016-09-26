---
title: Scala练手——斗鱼弹幕监控
date: 2016-09-19 17:19:00
categories: technology
tags: scala
---
# Scala练手——斗鱼弹幕监控

## Why
最近在学习hadoop, spark, kafka, zookeeper等大数据处理框架，也就顺带学习了一下scala这门编程语言，以阅读spark的源代码（spark核心使用scala编写）。没想到scala越学越上瘾，越来越喜欢函数式编程了。

在这里以`斗鱼弹幕监控`为课题，使用scala编写了一个斗鱼弹幕获取、分析的小程序练练手，在这里记录一下遇到的问题以及解决方案。

## 斗鱼API
斗鱼API使用时，需要与斗鱼服务器建立TCP长连接，使用心跳包维护连接，每一个连接对应一个斗鱼房间。当有消息时，斗鱼服务器会通过连接将消息推送过来。消息为Key-Value的格式，使用斗鱼自定义的序列化方法进行序列化，在客户端需要自行反序列化。

需要注意的是，斗鱼服务器限制连接个数，连接超出一定个数（我测试的是150）时，连接会被重置（RST）。

## 程序实现
程序需要实现以下功能：
1. 获取斗鱼房间列表，为每一个房间创建TCP长连接
2. 监听每一个TCP连接，有消息时接收消息并打印
3. 通过心跳包维护TCP连接

### BiMap
程序需要保存建立的长连接的信息，包括Socket和房间ID（room_id）的对应关系。由于Socket与房间ID一一对应，需要从Socket查房间ID，也需要从房间ID反查Socket，因此需要一个双向Map结构（BiMap）。

这里使用了两个ConcurrentHashMap构建了一个BiMap，保存Socket与room_id的双向映射关系。使用ConcurrentHashMap的原因是，创建TCP连接，向Map插入值时使用了多线程，因此需要使用线程安全类型。

```scala
class BiMap[TK, TV] {
    val leftMap: mutable.Map[TK, TV] = new ConcurrentHashMap[TK, TV]()
    val rightMap: mutable.Map[TV, TK] = new ConcurrentHashMap[TV, TK]()

    def addItem(key: TK, value: TV): Option[TV] = {
        rightMap.put(value, key)
        leftMap.put(key, value)
    }

    def findLeft(key: TK): Option[TV] = {
        leftMap.get(key)
    }

    def findRight(key: TV): Option[TK] = {
        rightMap.get(key)
    }

    def deleteLeft(key: TK): Option[TV] = {
        leftMap.remove(key)
    }

    def deleteRight(key: TV): Option[TK] = {
        rightMap.remove(key)
    }
}
```

### 房间列表获取
首先通过`getGameList`获取斗鱼房间类型列表，在通过`getRoomList`获取对应房间类型的房间列表。即斗鱼有很多房间类型，每一种房间类型下面又有许多房间。

```scala
    def getGameList: Future[List[Map[String, String]]] = Future {
        val s = fromUrlWithTimeout("http://open.douyucdn.cn/api/RoomApi/game")
        val root = parse(s)
        val ret: List[Map[String, String]] = for {
            JField("data", JArray(data)) <- root
            JObject(categories) <- data
        } yield (for {
            JField(field, JString(value)) <- categories
        } yield (field, value)) toMap

        ret
    }

    def getRoomList(categoryId: Int): Future[List[Map[String, String]]] = Future {
        val s = fromUrlWithTimeout(s"http://open.douyucdn.cn/api/RoomApi/live/$categoryId")
        val root = parse(s)
        val ret: List[Map[String, String]] = for {
            JField("data", JArray(data)) <- root
            JObject(rooms) <- data
        } yield (for {
            JField(field, JString(value)) <- rooms
        } yield (field, value)) toMap

        ret
    }

    val dytask = for {
            categories <- DouyuAPI.getGameList
        } yield for {
            category <- categories
            categoryId <- category.get("cate_id")
        } yield {
            DouyuAPI.getRoomList(categoryId.toInt)
        }
```

这里使用了scala的Future实现异步IO。Future可以快速的创建多线程任务，返回的值为Future[T]类型的数据。通过liftweb.json库解析斗鱼房间信息并返回。返回值为键值对列表`[{key:value},{key:value}]`

`dytask`变量存储了所有房间的信息，类型为`Future[List[Future[List[Map[String, String]]]]]`。这里使用了scala的for关键字，scala的for关键字其实只是一个语法糖（syntax sugar），编译器会将for编译为一系列的map/flatMap/foreach/withFilter/filter。

具体如何展开可以参考官网的[FAQ](http://docs.scala-lang.org/tutorials/FAQ/yield.html)。

然后是使用返回的Future结构的数据，异步添加各个房间到监控列表中。
```scala
dytask onSuccess {
    case categories =>
        categories foreach (_.onSuccess {
            case room =>
                room foreach (_.get("room_id") match {
                    case Some(roomIdStr) =>
                        if (roomSockMapping.leftMap.size < 150) {
                            val roomId = roomIdStr.toInt
                            val chann = SocketChannel.open(new InetSocketAddress(HOST, PORT))
                            chann.configureBlocking(false)
                            val selKey = chann.register(selector, SelectionKey.OP_READ)

                            roomSockMapping.synchronized({
                                if (roomSockMapping.leftMap.size < 150) {
                                    roomSockMapping.addItem(roomId, chann)

                                    val buff = ByteBuffer.allocate(MAX_BUFFER_LENGTH)
                                    buff.order(ByteOrder.LITTLE_ENDIAN)
                                    roomBufferMapping.put(chann, buff)

                                    loginRoom(roomId)
                                    joinGroup(roomId, -9999)
                                } else {
                                    selKey.cancel()
                                    chann.close()
                                }
                            })
                        }
                    case None =>
                        println("error")
                })
        })
}
```

这里使用了NIO的`Selector`，这样就可以在一个线程中同时监控多个Socket，注意Socket必须设置为非阻塞。Socket初始化，缓存初始化，初始房间登录消息都在这里完成。

## 循环获取弹幕
首先是使用selector监控Socket，当Socket可读时，读取Socket并将数据放入对应的缓存中。

`proc`函数用于处理*一条*消息，在`proc`的尾部递归调用自身来处理剩余的消息。在其他命令式语言中一般使用循环来完成这个功能，但scala的循环没有break，不好实现这样的功能（无法在数据不足时break出循环），这是为了提倡函数式编程，函数式编程中都没有`while`语句，而是使用递归来实现与循环一样的效果。

尾递归通过编译器优化能够达到与`while`相同的效率，为了防止尾递归不规范，可以加上`@tailrec`让编译器检查，如果函数不能转化为尾递归编译器会报错。

这里使用了ByteBuffer作为缓冲，由于接收的信息可能不全无法处理，比如一条消息长度为2000字节，当前只收到了1000字节，那就需要等待下次读取了新内容后再次检查数据是否足够。因此buffer中可能还有尚未处理的数据，只能使用`compact`而不是`clear`。

使用`compact`会将没有处理的数据移到buffer头部，可能带来一定的开销。可以考虑使用*环形缓冲区*来实现ByteBuffer以降低移动数据带来的开销。

```scala
def getServerMsg = {
    @tailrec
    def proc(b: ByteBuffer): Unit = {
        if (b.remaining() > 4) {
            val packetLen = b.getInt
            if (b.remaining() >= packetLen) {
                if (packetLen <= MAX_BUFFER_LENGTH && packetLen > 0) {
                    val strMsg = new String(b.array(), b.position() + 8, packetLen - 8)
                    b.position(b.position() + packetLen)
                    parseServerMsg(DyMessage(strMsg).msg)
                    proc(b)
                } else {
                    println(s"[FAIL] wrong length: $packetLen")
                }
            } else {
                b.position(b.position() - 4)
                println("[INFO] not enough, wait for more...")
            }
        }
    }

    //初始化获取弹幕服务器返回信息包大小
    try {
        val readyCount = selector.select(1000)
        for {key: SelectionKey <- selector.selectedKeys} {
            if (key.isReadable) {
                val sock = key.channel().asInstanceOf[SocketChannel]
                val buff = roomBufferMapping.get(sock).get

                // read new data into buffer
                val len = sock.read(buff)
                buff.flip()

                proc(buff)
                buff.compact
            }
        }
        selector.selectedKeys().clear()
    } catch {
        case ex: Exception => ex.printStackTrace();
    }
}
```