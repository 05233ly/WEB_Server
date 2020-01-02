Socket通信之UDP    
====

这是什么?  
1. 它是进程间通信的一种方式  
2. 更厉害的是, 它能实现不同主机之间的通信   

网络上各种服务, 大多是基于socket实现的     


Python实现socket  
```Python
import socket
s_tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s_udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```
注意这里的细节:   
细节一:  后面的`STREAM`类比为流水, 比喻tcp像流水一样流畅, 连贯, 于是有了stream  
细节二: 创建记得关闭, 上面省略了`s.close()`


那么问题来了, UDP提供服务, 怎么收发数据?    
UDP收发是可以分开写的    
```python
s_udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s_udp.bind(('', 6666))
s_udp.recvfrom(1024)
print(s_udp)
```
细节一: bind绑定的是元祖  
细节二: 发送的是二进制文件, 需要encode与decode  
```Python
s_udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s_udp.sendto('hhh'.encode(), ('192.168.0.102', 6666))
print(s_udp)
```
上面是核心代码   
接下来演示, 不同主机之间, 在局域网如何发送消息   
主机与服务器:  
![ScreenShot-00238](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00238.jpg)  
左边是数据的发送方, 右边服务器是数据的接收方   
注意ip地址, 我们接下来要用到  

左边Ubuntu发送数据方:  
```Python
import socket
s_udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s_udp.sendto('hhhh'.encode(), ('192.168.0.102', 6666))
s_udp.close()
```
右边Server接受数据方:    
```Python
import socket
s_udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s_udp.bind(('', 6666))

recv_data = s_udp.recvfrom(1024)
print(recv_data[0].decode())
s_udp.close()
```
![ScreenShot-00237](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00237.jpg)  
请看图, 继续我们上面的代码  
从`发送方`代码我们能略知一二UDP的性质:  
1. 发送方需要知道数据发到哪台主机, 主机上的那个端口号:  `bind('192.168.0.102', 6666)`   
2. 发送方需要发送数据, 而且数据要压缩成bytes类型的  
3. 发送完毕后需要关闭, 端口占用会浪费资源  

从`接收方`代码我们能略知一二UDP的性质:  
1. 本机需要知道接收哪里的数据, 哪里端口的数据: `bind(('', 6666))`      
2. 本机需要等待数据发过来:  `recvfrom(1024)`     
3. 本机需要对数据解压才能正常显示数据:  `decode()`    
![ScreenShot-00239](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00239.jpg)   



## 多线程UDP   
效果图    
![ScreenShot-00240](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00240.jpg)    
代码图      
![ScreenShot-00241](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00241.jpg)    

```Python
def msg_send(s):
	content = inpu('Pleast input your wants send massages, q is default')
	if content =='q':
		content = 'hhhh'
	s.sendto(content.encode(),('192.168.0.102', 6666))

def msg_recv(s):
	s.bind(('', 6666))
	recv_data = s.recvfrom(1024)
	print(recv_data[0].decode())

def main():
	s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

	t1 = threading.Thread(target=msg_send, args=(s, ))
	t2 = threading.Thread(target=msg_recv, args=(s, ))

	t1.start()
	t2.start()
 
 if __name__ == "__main__":
    main()
```




`-------分界线-------`

## UDP客户端  
![UDP通信](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/simple1.jpg)
1、socket创建一个套接字；  
2、然后使用send发送数据，在后面绑定ip与端口，同时指定编码格式；


可以在虚拟机看到主机发出的数据；
![send](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/send1.jpg)
UDP基本不用，互联网开始起步，设备差，UDP用的多。  


windows里默认编码是GBK；  
Linux里默认编码是utf-8;  

私有ip：  
10.0.0.0  ~  10.255.255.255  
172.16.0.0  ~  172.31.255.255  
192.168.0.0  ~ 192.168.255.255
