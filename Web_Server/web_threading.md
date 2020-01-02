Web和Socket通信--开启多线程--继承实现网页标签返回数据
=====


我们先看一段代码:   
```Python
import threading
import socket
import re, os

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
        t = threading.Thread(target=service_client, args=(new_socket,))
        t.start()
    tcp_server.close()

if __name__ == '__main__':
    main()
```
承接[上上文的-网页标签返回数据](https://github.com/KissMyLady/WEB_Server/blob/master/Web_Server/server_one.md), 通过上面的方法, 我们就可以得到一个服务器, 但浏览器访问时, 就可以让服务器做到自由访问了   
但是, 可以更进一步, 提高效率吗?   
这里我们采用了多线程,  线程在这个方面就比多进程要好点, 开辟的速度要快, 但因为c语言写的解释器原因, 假多线程并不是真正意义上的多线程  

## 来看下多进程怎么实现的:   
```Python
tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('', 8080))
tcp_server.listen(128)
while True:
    new_socket, client_addr = tcp_server.accept()
    t = threading.Thread(target=service_client, args=(new_socket,))
    t.start()
tcp_server.close()
```
```Python
t = threading.Thread(target=service_client, args=(new_socket,))
```
也就是相对于多进程的`multiprocessing.Process`替换成了`threading.Thread`   
这里有个需要我们注意的细节:  
线程共享全局变量   
关闭套接字的时候需要多加注意  
