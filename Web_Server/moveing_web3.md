动态Mini-静动态分离  
=====

## main  
```Python
import multiprocessing
import socket, re, time
import mini  # 调用外部py文件

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
        # 2.1，不是py认为是静态资源(html/css/js/png/jpg等)
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
                
                # header
                new_socket.send(response.encode('utf-8'))
                # body
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
            file_nema = str("/login.py")
            body = mini.application(file_nema)
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
## Mini  
```Python
import time

def login():
    l = '---------login---------.....time: %s' % time.ctime()
    return l

def register():
    c = '---------register---------.....time: %s' % time.ctime()
    return c

def profile():
    return '---------profile---------.....time: %s' % time.ctime()

def application(file_name):
    print(file_name)
    if file_name == "/login.py":
        return login()
    elif file_name == "/register.py":
        return register()
    elif file_name == "/profile.py":
        return profile()
    else:
        return "not found your page..."

```



