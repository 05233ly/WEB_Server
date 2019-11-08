## 单线程单进程并发服务器  

先看看局部核心代码，通过循环控制客户端监听，控制套接字创建，数据接收  

![server](https://github.com/KissMyLady/Web-of-Python/blob/master/Web_Server/Img/one_server.jpg)  


## 加强版-完整代码展示  
第一层： 循环等待客户端连接  
第二层： 循环等待套接字获取数据  

![mulit_server](https://github.com/KissMyLady/Web-of-Python/blob/master/Web_Server/Img/mulit_server1.jpg)  
