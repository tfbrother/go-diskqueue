# go-diskqueue
A Go package providing a filesystem-backed FIFO queue

Pulled out of https://github.com/nsqio/nsq

# diskqueue 源码阅读
## 存在的问题
### 文件读取问题
1. 简介

> 文件读取分为两层，因为加入的读取缓冲区。所以设计到两个读取位置的索引。首先是最底层文件读取到哪个位置了，其次是缓冲区中的内容读取到哪个位置了。缓冲区里面的索引是bufio.Reader自身维护的，应用程序不需要关心。
2. readPos,readFileNum,nextReadPos,nextReadFileNum之间的关系

| 名字 | 含义 |
| --- | --- |
| readFileNum | 外界消费到了哪个文件了 |
| readPos | 外界消费到了文件内的哪个位置了 | 
| nextReadFileNum | 下一次从哪个文件读取 |
| nextReadPos | 下一次从文件内哪个位置读取 |

> 队列里面的数据此处称之为元素。元素具有多个状态，第一还未被读取，第二被读取未被消费，第三被读取且已经被消费。当元素被读取出来，尚未被外界消费时，此时readPos != nextReadPos;当元素被取出来，且刚刚被消费(尚未进行下一次读取)，此时readPos==nextReadPos;