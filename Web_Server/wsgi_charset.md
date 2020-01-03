WSGI协议--中文添加及服务器版本
======
我们先看下面一段代码:  
```Python
import multiprocessing
import socket, re, mini
import mini

class WSGIServer(object):
    def __init__(self):
        self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # tcp_server.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
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
                response += "----Not found----"
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
            for temp in self.headers:
                header += '%s:%s\r\n' % (temp[0], temp[1])
            header += '\r\n'
            response = header + body
            new_socket.send(response.encode('utf-8'))
        new_socket.close()
    
    def set_resopnse_header(self, status, headers):
        self.status = status
        self.headers = [("server", "Mini_web_frame_2.1")]
        self.headers += headers

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
## 问题一:
当我们在调用服务器时, 如何在浏览器上面显示服务器版本号?  
显然不能在WSGI右边的框架中添加, 框架只负责返回数据, 服务器才是合并数据的  

所以, 我们在设置头函数这里添加如下代码:  
```Python
def set_resopnse_header(self, status, headers):
    self.status = status
    self.headers = [("server", "Mini_web_frame_2.1")]
    self.headers += headers
```
定义`server:bbbsss`, 再合并头, 而后在   
``` Python
env = dict()
    body = mini.application(env, self.set_resopnse_header)
    header = 'HTTP/1.1 %s\r\n' % self.status
    for temp in self.headers:
        header += '%s:%s\r\n' % (temp[0], temp[1])
    header += '\r\n'
    response = header + body
    new_socket.send(response.encode('utf-8'))
```
这里进行头合并主体, 返回网页        

## 问题二:   
网页正常情况是默认显示中文乱码, 如何解决?  
```Python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'),('charset=utf-8')])
    return 'Hello World! kiss my lady'
```
在WSGI中添加请求头说明即可   
在Django中, 修改更为简单,  在srtting中修改即可  

