WSGI协议
======

## 这是什么? 
![ScreenShot-00250](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00250.jpg)  
不修改服务器和架构代码而确保可以在多个架构下运行web服务器    
Python Web Server Gateway Interface(简称WSGI，读威士忌)     
 
知识引导:  
web服务器必须具备WSGI接口，所有的现代Python Web框架都已具备WSGI接口，它让你不对代码作修改就能使服务器和特点的web框架协同工作   

WSGI由web服务器支持，而web框架允许你选择适合自己的配对，但它同样对于服务器和框架开发者提供便利使他们可以专注于自己偏爱的领域和专长而不至于相互牵制   
其他语言也有类似接口：Java 有Servlet API，Ruby 有Rack    


## WSGI协议具体表现  
我们先来看一段简单实现WSGI的代码:      
```Python
import multiprocessing
import socket, re, mini

class WSGIServer(object):
    def __init__(self):
        self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.tcp_server.bind(('', 8080))
        self.tcp_server.listen(128)
    
    def service_client(self, new_socket):
        request = new_socket.recv(1024).decode('utf-8')
        request_lines = request.splitlines()
        ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
        file_nema = ret[0]
        if file_nema:
            if file_nema == '/':
                file_nema = "/index.html"
                
        if not file_nema.endswith(".py"):
            try:
                path = r'D:/web' + file_nema
                f = open(path, "rb")
            
            except:
                response = 'HTTP/1.1 404 NOT FOUND\r\n'
                response += "\r\n"
                response += "----404 Not found----"
                new_socket.send(response.encode('utf-8'))
            else:
                response = 'HTTP/1.1 200 OK\r\n'
                response += "\r\n"
                html_content = f.read()
                f.close()
                new_socket.send(response.encode('utf-8'))
                new_socket.send(html_content)
                new_socket.send(response.encode('utf-8'))
        else:
            env = dict()
            body = mini.application(env, self.set_resopnse_header)
            header = 'HTTP/1.1 %s\r\n' % self.status
            # [('Content-Type', 'text/html'),(xxx:yyy)]
            for temp in self.headers:
                header += '%s:%s\r\n' % (temp[0], temp[1])
            header += '\r\n'
            response = header + body
            new_socket.send(response.encode('utf-8'))
        new_socket.close()
    
    def set_resopnse_header(self, status, headers):
        self.status  = status
        self.headers = headers
        

    def run_forever(self):
        while True:
            new_socket, client_addr = self.tcp_server.accept()
            p = multiprocessing.Process(target=self.service_client, args=(new_socket,))
            p.start()
            new_socket.close()
        self.tcp_server.close()

def main():
    wsgi_server = WSGIServer()
    wsgi_server.run_forever()

if __name__ == '__main__':
    main()
```
## 启动方式
不难发现这里, 通过class封装了一个小型服务器, 仔细看看, 这是通过main函数启动, 调用`run`方法启动的一个代码  
```Python
def main():
    wsgi_server = WSGIServer()
    wsgi_server.run_forever()
```
实例化对象, 启动实例化对象的`run`方法  

## run方法是什么样的  
run方法封装在class里面, 我们首先得先看一看class初始化了什么    
```Python
def __init__(self):
    self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    self.tcp_server.bind(('', 8080))
    self.tcp_server.listen(128)
```
我们看到, init初始化了`tcp_server`套接字, 绑定了端口和主机, 并且改为被动监听套接字   
然后初始化创建的实例变量传到下面的run方法:  
```Python
def run_forever(self):
    while True:	
        new_socket, client_addr = self.tcp_server.accept()
        p = multiprocessing.Process(target=self.service_client, args=(new_socket,))
        p.start()
        new_socket.close()
    self.tcp_server.close()
```
同前面一样, 将解堵塞方法外包给其他函数来做    
这里我们启动了进程, 并在最后关闭tcp套接字, 请注意这里, 进程属于写实拷贝, 进程类的关闭不代表外面环境的关闭  
进程将监听到的套接字转发给类里面的方法`service_client`:  		
`target=self.service_client, args=(new_socket,)`
我们来看看`service_client`是怎么回事:  

## 类的service_client方法   
```Python
def service_client(self, new_socket):
    request = new_socket.recv(1024).decode('utf-8')
    request_lines = request.splitlines()
    ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
    file_nema = ret[0]
    if file_nema:
        if file_nema == '/':
            file_nema = "/index.html"
            
    if not file_nema.endswith(".py"):
        try:
            path = r'D:/web' + file_nema
            f = open(path, "rb")
        except:
            response = 'HTTP/1.1 404 NOT FOUND\r\n'
            response += "\r\n"
            response += "----404 Not found----"
            new_socket.send(response.encode('utf-8'))
        else:
            response = 'HTTP/1.1 200 OK\r\n'
            response += "\r\n"
            html_content = f.read()
            f.close()
            new_socket.send(response.encode('utf-8'))
            new_socket.send(html_content)
            new_socket.send(response.encode('utf-8'))
    else:
        env = dict()
        body = mini.application(env, self.set_resopnse_header)
        header = 'HTTP/1.1 %s\r\n' % self.status
        # [('Content-Type', 'text/html'),(xxx:yyy)]
        for temp in self.headers:
            header += '%s:%s\r\n' % (temp[0], temp[1])
        header += '\r\n'
        response = header + body
        new_socket.send(response.encode('utf-8'))
    new_socket.close()
```
这段代码稍微有点长, 但是不难发现, 它有三个主题部分:   
1. 解析浏览器的请求   
2. 判断请求是否为静态, 动态/`.py`资源   

如果请求的是静态资源: `.html结尾`或其他格式结尾      
```Python
if not file_nema.endswith(".py"):
    try:
        path = r'D:/web' + file_nema
        f = open(path, "rb")
    except:
        response = 'HTTP/1.1 404 NOT FOUND\r\n'
        response += "\r\n"
        response += "----404 Not found----"
        new_socket.send(response.encode('utf-8'))
    else:
        response = 'HTTP/1.1 200 OK\r\n'
        response += "\r\n"
        html_content = f.read()
        f.close()
        new_socket.send(response.encode('utf-8'))
        new_socket.send(html_content)
        new_socket.send(response.encode('utf-8'))
``` 
常规的打开文件并返回数据, 没有就返回404

## WSGI协议核心  
这段代码就是WSGI的转发核心了:  
```
else:
    env = dict()
    body = mini.application(env, self.set_resopnse_header)
    header = 'HTTP/1.1 %s\r\n' % self.status

    for temp in self.headers:
        header += '%s:%s\r\n' % (temp[0], temp[1])
    header += '\r\n'
    response = header + body
    new_socket.send(response.encode('utf-8'))
```
可以发现, 我们定义了字典`env = dict()`   
我们使用了`mini.application()`, 调用了包  
字典我们先不看, 先看看内部是如何实现的   
![ScreenShot-00251](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00251.jpg)   
红色框起来的部分, 就称之为WSGI接口  
它只要我们实现这个函数,  就可以相应HTTP请求   


