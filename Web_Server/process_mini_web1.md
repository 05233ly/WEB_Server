多进程_OOP_web服务器  
=====

## 多进程_OOP_web服务器  
```Python
import multiprocessing
import socket, re

class WSGIServer(object):

    def __init__(self):
        # 创建套接字
        self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # tcp_server.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
        self.tcp_server.bind(('', 8080))  # 绑定
        self.tcp_server.listen(128)  # 监听

    def service_client(self, new_socket):
        request = new_socket.recv(1024).decode('utf-8')
        request_lines = request.splitlines()
        print(request_lines[0])
        # GET /favicon.ico HTTP/1.1
        ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
        ret_1 = ret[0]
        if ret_1:
            file_nema = ret_1
            print('>' * 50, file_nema)
            if file_nema == '/':
                file_nema = "/index.html"
                print('输入信息没有内容，默认为index,打印默认页:', file_nema)

        response = 'HTTP/1.1 200 OK\r\n'
        response += "\r\n"
        try:
            path = r'D:/Python初级工程师/19、网络编程/2、web服务器' + file_nema
            print('这是路径', path)
            f = open(path, "rb")
        
        except:
            response = 'HTTP/1.1 404 NOT FOUND\r\n'
            response += "\r\n"
            response += "----Not found----"
            new_socket.send(response.encode('utf-8'))
        
        else:
            html_content = f.read()  # 网页小, 直接读
            f.close()
            
            # header
            new_socket.send(response.encode('utf-8'))
            # body
            new_socket.send(html_content)
            
            # 2、返回http格式的数据，给浏览器
            print('发送数据给客户端')
            new_socket.send(response.encode('utf-8'))
            
            # 关闭
            new_socket.close()

    def run_forever(self):
        print('开始连接')
        while True:
            new_socket, client_addr = self.tcp_server.accept()
            p = multiprocessing.Process(target=self.service_client, args=(new_socket,))
            p.start()
            
            # 请注意，多进程写实拷贝，new_socket
            # 子进程确确实实关闭了，但这里还没关(文件rm也是这样，只是名义上的删除)
            new_socket.close()
        
        self.tcp_server.close()
        

def main():
    wsgi_server = WSGIServer()
    wsgi_server.run_forever()


if __name__ == '__main__':
    main()

```

多进程_OOP_web动态服务器   
=====
```Python
import multiprocessing
import socket, re
import time

# 更改函数式服务器， 变成OOP_多进程
class WSGIServer(object):
    
    def __init__(self):
        # 创建套接字
        self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # tcp_server.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
        self.tcp_server.bind(('', 8080))  # 绑定
        self.tcp_server.listen(128)  # 监听
    
    def service_client(self, new_socket):
        request = new_socket.recv(1024).decode('utf-8')
        request_lines = request.splitlines()
        print(request_lines[0])
        # GET /favicon.ico HTTP/1.1
        ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
        ret_1 = ret[0]
        if ret_1:
            file_nema = ret_1
            print('>' * 50, file_nema)
            if file_nema == '/':
                file_nema = "/index.html"
                print('输入信息没有内容，默认为index,打印默认页:', file_nema)

        # 2、判断是否是py结尾
        # 2.1，如果不是py，认为是静态资源(html/css/js/png/jpg等)
        if not file_nema.endswith(".py"):
            try:
                path = r'D:/Python初级工程师/19、网络编程/2、web服务器' + file_nema
                print('这是路径', path)
                f = open(path, "rb")
            
            except:
                response = 'HTTP/1.1 404 NOT FOUND\r\n'
                response += "\r\n"
                response += "----Not found----"
                new_socket.send(response.encode('utf-8'))
            
            else:
                response = 'HTTP/1.1 200 OK\r\n'
                response += "\r\n"
                html_content = f.read()  # 网页小, 直接读
                f.close()
                new_socket.send(response.encode('utf-8'))
                new_socket.send(html_content)
                
                # 2、返回http格式的数据，给浏览器
                print('发送数据给客户端')
                new_socket.send(response.encode('utf-8'))
                
                # 关闭
                new_socket.close()
        else:
            # 2.2 动态资源
            header = "HTTP/1.1 200 OK\r\n"
            header += "\r\n"
            
            # 此时刷新页面，可以看到页面的时间在变化，此时，这是一个简单的动态web网站
            body = 'hahahaha%s' % (time.ctime()) # 这里就是web框架要发挥作用的地方，单独建立py文件，降低耦合度
            # 发送请response相应
            response = header + body
            new_socket.send(response.encode('utf-8'))
    
    def run_forever(self):
        print('开始连接')
        while True:
            new_socket, client_addr = self.tcp_server.accept()
            p = multiprocessing.Process(target=self.service_client, args=(new_socket,))
            p.start()
            
            # 请注意，多进程写实拷贝，new_socket
            # 子进程确确实实关闭了，但这里还没关(文件rm也是这样，只是名义上的删除)
            new_socket.close()
        
        self.tcp_server.close()


def main():
    wsgi_server = WSGIServer()
    wsgi_server.run_forever()


if __name__ == '__main__':
    main()

```











