## Main_server  
```Python
import multiprocessing
import socket, re
import mini

# 说   明：通过传递字典实现浏览器请求的资源不一样，响应要不一样
# 目   标：字典有什么用

class WSGIServer(object):
    def __init__(self):
        # 创建套接字
        self.tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # tcp_server.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
        self.tcp_server.bind(('', 8080))  # 绑定
        self.tcp_server.listen(128)       # 监听
    
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
                path = r'D:/Python/web_server' + file_nema
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
        # 动态网页
        else:
            env = dict()
            env['PATH_INFO'] = file_nema
            # 调用框架
            body = mini.application(env, self.set_resopnse_header)
            # 只有当函数执行完毕，服务器才有头、状态码，数据
            header = 'HTTP/1.1 %s\r\n' % self.status
            
            # [('Content-Type', 'text/html'),(xxx:yyy)]
            for temp in self.headers:
                header += '%s:%s\r\n' % (temp[0], temp[1])
            header += '\r\n'  # 头传输完毕
            response = header + body  # 合并头和body
            
            new_socket.send(response.encode('utf-8'))
        new_socket.close()
    
    def set_resopnse_header(self, status, headers):
        # 服务器保存状态码和头
        self.status = status  # '200 OK'
        self.headers = [("server","Mini_web_frame_4.1")]
        self.headers += headers  # [('Content-Type', 'text/html')]
    
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
## mini_web_fream  
```SQL
def index():
	return '这是主页'
  
def login():
	return '这是登录页面'

def application(env, start_response):
    start_response('200 OK', [('Content-Type', 'text/html;charset=utf-8')])
    file_name = env['PATH_INFO']
    # file_name = "/index.py"
    if file_name == "/index.py":
        return index()
    elif file_name == "/login.py":
        return login()
    else:
        return 'Hello World! kiss my lady 爱你三千零一遍'

```


