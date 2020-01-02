Socket通信之TCP
====

## TCP是什么--知识引导  
TCP协议，传输控制协议Transmission Control Protocol   
* TCP是一种`面向连接的`、可靠的、基于字节流的传输层通信协议   

TCP通信需要经过1. 创建连接、 2. 数据传送、 3. 终止连接三个步骤      

TCP通信模型中，在通信开始之前，一定要先建立相关的链接，才能发送数据，类似于生活中，"打电话"     

## 1. 面向连接     
* 1. 双方必须先建立连接才能进行数据的传输      
* 2. 双方都必须为该连接分配必要的内核资源，认真对待这件事      
* 3. 双方间的数据传输都可以通过这一个连接进行       
* 4. 完成数据交换后，双方必须断开此连接，以释放系统资源   
* 5. 这种连接是一对一的    

## 2. 可靠传输   
* TCP采用发送应答机制   
TCP发送的每个报文段都必须得到接收方的应答才认为这个TCP报文段传输成功  

* 超时重传   
发送端发出一个报文段之后就启动定时器，如果在定时时间内没有收到应答就重新发送这个报文段   
> TCP为了保证不发生丢包，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收   
> 然后接收端实体对已成功收到的包发回一个相应的确认(ACK)     
>> 如果发送端实体在合理的往返时延(RTT) 内未收到确认，那么对应的数据包就被假设为已丢失将会被进行重传   

* 错误校验
TCP用一个校验和函数来检验数据是否有错误；在发送和接收时都要计算校验和。

* 流量控制和阻塞管理
流量控制用来避免主机发送得过快而使接收方来不及完全收下  


## TCP客户端实现  
```Python
import socket

tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

tcp.connect((ip, port))

tcp.send(data.encode())
recvData = tcp.recv(1024)
print(recvData.decode())

tcp.close()
```
我们看到上述创建tcp客户端的代码并不陌生,  这与udp的创建没什么两样, 的确就是这样  
但同时, 我们需要看到里面的细节    
从`客户端`代码我们能略知一二TCP的性质:   
```Python
tcp.connect((ip, port))
```
* 1. tcp客户端接收数据, 需要绑定服务器, 不绑定没法接收    
上面的知识引导, 已经说了tcp是面向连接的, `双方必须先建立连接才能进行数据的传输`      
发送数据没什么好说的, 编码发送即可:     
```Python
tcp.send(data.encode())
```
* 2. 接收数据也没什么好说的, 接收即可:     
```Python
recvData = tcp.recv(1024)
```
* 3. 然后显示接收的数据, 再关闭连接:  
```Python
print(recvData.decode())
tcp.close()
```

## TCP服务端实现    
```Python
from socket import *
server = socket(AF_INET, SOCK_STREAM)

server.bind(('', 7788))

server.listen(128)

c, addr = server.accept()

recv_data = client_socket.recv(1024)
print(recv_data.decode('gbk'))

client_socket.close()
```
TCP服务端, 顾名思义, 就是提供服务的一方   
怎么才算叫提供服务呢?  得能够接收数据, 能够返回数据, 像浏览网站, 我们访问的就是服务器   
在上面的知识引导可以看到: `TCP通信需要经过1. 创建连接、 2. 数据传送、 3. 终止连接三个步骤`, 我们的TCP服务端也不例外  
从`服务端`代码我们能略知一二`TCP服务端`的性质:     
* 1. 同UDP通信一样, 需要绑定主机服务器, 绑定端口号, 这样数据来了就有地方留住     
```Python
server.bind(('', 7788))
```
* 2. 看到上方的`TCP客户端`, 我们可以看到TCP客户端的代码:    
```Python
tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp.connect((ip, port))
```
显然, 作为服务器, 我们希望的是别人连接过来时, 我们能够听到(看到)别人过来了, 这里我们需要改一改:     
```Python   
server.listen(128)
```
这样, 我们就把正常的连接`connect`变为被动连接: 被动的等待别人过来访问我   

* 3. tcp连接过来后, 一个套接字里面含有`(客户端的套接字, 客户端的地址)`, 我们需要提取数据:  
```Python 
c, addr = server.accept()
```
这样, 我们就把客户端提取出来了`c`   

* 4. 接下来就是正常的收发数据了, 最后关闭客户端, 释放宝贵的资源     
```Python 
recv_data = c.recv(1024)
print(recv_data.decode())
c.close()
```


## 注意点--知识引导    
1, tcp服务器一般情况下都需要绑定，否则客户端找不到这个服务器       
2. tcp客户端一般不绑定，因为是主动链接服务器，所以只要确定好服务器的ip、port等信息就好，本地客户端可以随机     
3. tcp服务器中通过listen可以将socket创建出来的主动套接字变为被动的，这是做tcp服务器时必须要做的   

4. 当客户端需要链接服务器时，就需要使用connect进行链接，udp是不需要链接的而是直接发送，但是tcp必须先链接，只有链接成功才能通信   
5. 当一个tcp客户端连接服务器时，服务器端会有1个新的套接字，这个套接字用来标记这个客户端，单独为这个客户端服务   

6. listen后的套接字是被动套接字，用来接收新的客户端的链接请求的，而accept返回的新套接字是标记这个新客户端的   
7. 关闭listen后的套接字意味着被动套接字关闭了，会导致新的客户端不能够链接服务器，但是之前已经链接成功的客户端正常通信。   
8. 关闭accept返回的套接字意味着这个客户端已经服务完毕   

9. 当客户端的套接字调用close后，服务器端会recv解堵塞，并且返回的长度为0，因此服务器可以通过返回数据的长度来区别客户端是否已经下线   

## 其他说明
TCP客户端   
![TCP客户端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/tcp_client3.jpg)
TCP服务端  
![TCP客户端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/tcp_server.jpg)  
