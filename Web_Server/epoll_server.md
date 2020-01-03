Epol模型讲解   
===========

## Epol知识引导
epoll核心，将绿色的部分(服务端口列表)拿出来了，此时是OS与Server共享这个一坨绿色`共享套接字内存列表`；  
使用的是`内存映射技术`，减少了复制；
![img244](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00244.jpg)    
相比较上一页的并发服务器，使用list存储，当数据非常多的时候，会导致速度直线下降，  
好比厨师在大街上问每个人: ‘吃了么您嘞？’， 显然，这种效率非常低。  
 于是epoll出现了，‘要吃饭的人，会直接告诉厨师，我要吃饭(我有数据)’， 这样的方式，叫：`以事假通知为信号`;  

Epol是什么?  
* 1. Linux服务器用的epol实现的  
* 2. Gevent底层就是用的epol  
* 3. 一般的服务器用的也还是epol, 像Nginc, Apache等著名服务器   

它能够单进程, 单线程实现高并发   

## epol讲解  
我们先来看一段代码:   
```Python
import socket
import select

# 创建套接字
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 设置可以重复使用绑定的信息
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR,1)

# 绑定本机信息
s.bind(("",7788))

# 变为被动
s.listen(10)

# 创建一个epoll对象
epoll = select.epoll()

# 测试，用来打印套接字对应的文件描述符
# print(s.fileno())
# print(select.EPOLLIN|select.EPOLLET)

# 注册事件到epoll中
# epoll.register(fd[, eventmask])
# 注意，如果fd已经注册过，则会发生异常
# 将创建的套接字添加到epoll的事件监听中
epoll.register(s.fileno(), select.EPOLLIN|select.EPOLLET)

connections = {}
addresses = {}

# 循环等待客户端的到来或者对方发送数据
while True:
    epoll_list = epoll.poll()
    for fd, events in epoll_list:
        if fd == s.fileno():
            new_socket, new_addr = s.accept()
            connections[new_socket.fileno()] = new_socket
            addresses[new_socket.fileno()] = new_addr
            epoll.register(new_socket.fileno(), select.EPOLLIN|select.EPOLLET)

        elif events == select.EPOLLIN:
            recvData = connections[fd].recv(1024).decode("utf-8")
            if recvData:
                print('recv:%s' % recvData)
            else:
                epoll.unregister(fd)
                connections[fd].close()
                del connections[fd]
                del addresses[fd]
```



```Python
import socket, time
import sys, re ,select

class WSGIServer(object):
    def __init__(self, port, documents_root):
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind(("", port))
        self.server_socket.listen(128)

        self.documents_root = documents_root
        self.epoll = select.epoll()
        self.epoll.register(self.server_socket.fileno(), select.EPOLLIN|select.EPOLLET)
        self.fd_socket = dict()

    def run_forever(self):

        # 等待对方链接
        while True:
            epoll_list = self.epoll.poll()

            for fd, event in epoll_list:
                if fd == self.server_socket.fileno():
                    new_socket, new_addr = self.server_socket.accept()
                    self.epoll.register(new_socket.fileno(), select.EPOLLIN | select.EPOLLET)
                    self.fd_socket[new_socket.fileno()] = new_socket
                elif event == select.EPOLLIN:
                    request = self.fd_socket[fd].recv(1024).decode("utf-8")
                    if request:
                        self.deal_with_request(request, self.fd_socket[fd])
                    else:
                        self.epoll.unregister(fd)
                        self.fd_socket[fd].close()
                        del self.fd_socket[fd]

    def deal_with_request(self, request, client_socket):
        if not request:
            return
        request_lines = request.splitlines()
        for i, line in enumerate(request_lines):
            print(i, line)

        # 提取请求的文件(index.html)
        # GET /a/b/c/d/e/index.html HTTP/1.1
        ret = re.match(r"([^/]*)([^ ]+)", request_lines[0])
        if ret:
            file_name = ret.group(2)
            if file_name == "/":
                file_name = "/index.html"


        # 读取文件数据
        try:
            f = open(self.documents_root+file_name, "rb")
        except:
            response_body = "file not found, 请输入正确的url"

            response_header = "HTTP/1.1 404 not found\r\n"
            response_header += "Content-Type: text/html; charset=utf-8\r\n"
            response_header += "Content-Length: %d\r\n" % len(response_body)
            response_header += "\r\n"
            client_socket.send(response_header.encode('utf-8'))
            client_socket.send(response_body.encode("utf-8"))
        else:
            content = f.read()
            f.close()
            response_body = content
            response_header = "HTTP/1.1 200 OK\r\n"
            response_header += "Content-Length: %d\r\n" % len(response_body)
            response_header += "\r\n"
            client_socket.send(response_header.encode("utf-8")+response_body)


# 设置服务器服务静态资源时的路径
DOCUMENTS_ROOT = "./html"


def main():
    """控制web服务器整体"""
    # python3 xxxx.py 7890
    if len(sys.argv) == 2:
        port = sys.argv[1]
        if port.isdigit():
            port = int(port)
    else:
        print("运行方式如: python3 xxx.py 7890")
        return

    print("http服务器使用的port:%s" % port)
    http_server = WSGIServer(port, DOCUMENTS_ROOT)
    http_server.run_forever()


if __name__ == "__main__":
    main()
```




