epoll简单理解  
===

### epoll核心  
![epoll](https://github.com/KissMyLady/Web-of-Python/blob/master/Web_Server/Img/epoll.jpg)  
epoll核心，将绿色的部分(服务端口列表)拿出来了，此时是OS与Server共享这个一坨绿色`共享套接字内存列表`；  
使用的是`内存映射技术`，减少了复制；
相比较上一页的并发服务器，使用list存储，当数据非常多的时候，会导致速度直线下降，  
好比厨师在大街上问每个人: ‘吃了么您嘞？’， 显然，这种效率非常低。  
 于是epoll出现了，‘要吃饭的人，会直接告诉厨师，我要吃饭(我有数据)’， 这样的方式，叫：`以事假通知为信号`;  

### 介绍  
当今Linux服务器里采用的方式，没有多进程，没有多线程，也没有采用非阻塞，而是epoll；像Nginx， Apache等都是epoll实现；  
另外，gevent(延时阻塞，实现伪线程)的底层就是epoll。  
#### epoll是服务器的主流方式  

### 使用epoll实现高并发服务器  
![](https://github.com/KissMyLady/Web-of-Python/blob/master/Web_Server/Img/epoll_main1.jpg)  



### 看完了  
- [返回主页](https://github.com/KissMyLady)  
- [返回Web主页](https://github.com/KissMyLady/Web-of-Python)  
- [上一页-单进程单线程并发_长链接](https://github.com/KissMyLady/Web-of-Python/blob/master/Web_Server/long_server.md)  
- [下一页- TCP/IP协议](https://github.com/KissMyLady/Web-of-Python/blob/master/Communicationg/TCP.md)   
