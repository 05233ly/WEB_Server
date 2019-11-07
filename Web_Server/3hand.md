Welcome to the Web-of-Python wiki!
# 三次握手--顶层抽象
首先，移除细节层面，让我们看看高度抽象层面是怎样的。就像使用高级语法的程序员不用管底层的计算机细节一样。

理解： 类比人与人的语言交流，交流实质内容之前，我首先得确认你的耳朵、声带是可以工作的，同样，对方也是这么想的。
## 三次握手
1. 第一次握手：客户端(client)发送，服务端收到，这样服务器说：客户端发、服务端收能力正常；
![一次握手](https://github.com/KissMyLady/Web-of-Python/blob/master/hand1.jpg)
2. 第二次握手：服务端(server)发送，客户端收到，这样客户端说：服务端收发， 客户端收发正常；
![二次握手](https://github.com/KissMyLady/Web-of-Python/blob/master/hand2.jpg)
3. 第三次握手：客户端(client)发送，服务端收到，这样服务端说，客户端收发， 服务端收发正常；
![三次握手](https://github.com/KissMyLady/Web-of-Python/blob/master/hand3.jpg)
   至此，双方都已经确认对方的收发没有问题，开始双方间数据(Data)传输; 
