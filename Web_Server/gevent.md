## 协程Gevet服务器  

在一个网站访问量很多的情况下， 多线程， 多进程100%会挂掉， 而协程效率一定是最高的  
Gevent很简单，在调用函数时，如果发生阻塞，立马调用下一个函数，这样就看起来像是多线程，但是本质上不是。  

让我们先看一段代码:  
```Python
from gevent import monkey
import gevent
import socket
import re, os
monkey.patch_all()

def service_client(new_socket):
    request = new_socket.recv(1024).decode('utf-8')
    request_lines = request.splitlines()
    ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
    file_nema = ret[0]
    if file_nema:
        if file_nema == '/':
            file_nema = "/index.html"
    response = 'HTTP/1.1 200 OK\r\n'
    response += "\r\n"
    
    try:
        path = r'D:/web' + file_nema
        f = open(path, "rb")
    
    except:
        response = 'HTTP/1.1 404 NOT FOUND\r\n'
        response += "\r\n"
        response += "----Not found----"
        new_socket.send(response.encode('utf-8'))
    
    else:
        html_content = f.read()
        f.close()
        new_socket.send(response.encode('utf-8'))
        new_socket.send(html_content)
        new_socket.send(response.encode('utf-8'))
        new_socket.close()

def main():
    tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_server.bind(('', 8080))
    tcp_server.listen(128)
    while True:
        new_socket, client_addr = tcp_server.accept()
        gevent.spawn(service_client, new_socket)
    tcp_server.close()

if __name__ == '__main__':
    main()
```
老样子, `service_client`处理函数没变, 我们需要的是转发效率的高效  
这里我们引用了`gevent`模块   
```Python
from gevent import monkey
import gevent
monkey.patch_all()
```
并且在里面添加了`monkey.patch_all()`方法, 该方法的作用就是替换
[猴子补丁](https://zhuanlan.zhihu.com/p/37679547), 这里先把方法给实现了, 后面谈谈什么是猴子补丁  

## 函数`main`  
```Python
tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('', 8080))
tcp_server.listen(128)
while True:
    new_socket, client_addr = tcp_server.accept()
    gevent.spawn(service_client, new_socket)
tcp_server.close()
```
