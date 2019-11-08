## Gevnet实现协程服务器  

在一个网站访问量很多的情况下， 多线程， 多进程100%会挂掉， 而协程效率一定是最高的  
gevent很简单，在调用函数时，如果发生阻塞，立马调用下一个函数，这样就看起来像是多线程，但是本质上不是。  

![gevnet](https://github.com/KissMyLady/Web-of-Python/upload/master/Web_Server/Img/gevent_web.jpg)  

  
## 接下来  
- [长链接，短链接]()  
