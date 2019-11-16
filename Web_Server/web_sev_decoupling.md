服务-框架高度解耦  
=====

## 解耦
目标: python3 web_server.py 7788 mini_frame:application  
执行: 1、端口解耦；
      2、指定模块；
      3、配置服务器文件；
      
## 代码总览  
```Python
import socket, re, sys
import multiprocessing
# import dynamic.mini_frame

# 加载配置文件
# 让web服务器支持配置文件
class WSGIServer(object):
    def __init__(self, port, app, static_path):
        self.tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        
        # 这里port其他地方不用，就不写self了
        self.tcp_server_socket.bind(("", port))
        self.application = app
        self.static_path = static_path
        # 监 听
        self.tcp_server_socket.listen(128)

    def service_client(self, new_socket):
        request = new_socket.recv(1024).decode("utf-8")
        print(">>>"*50)
        request_lines = request.splitlines()
        print("")
        print(">"*20)
        print(request_lines)

        # GET /index.html HTTP/1.1
        # get post put del
        file_name = ""
        ret = re.match(r"[^/]+(/[^ ]*)", request_lines[0])
        if ret:
            file_name = ret.group(1)
            print("*"*50, file_name)
            if file_name == "/":
                file_name = "/index.html"

        # 2. 返回http格式的数据，给浏览器
        # 2.1 如果请求的资源不是以.py结尾，那么就认为是静态资源（html/css/js/png，jpg等）
        if not file_name.endswith(".py"):
            try:
                f = open(self.static_path + file_name, "rb")
            except:
                response = "HTTP/1.1 404 NOT FOUND\r\n"
                response += "\r\n"
                response += "------file not found-----"
                new_socket.send(response.encode("utf-8"))
            else:
                html_content = f.read()
                f.close()
                # 2.1 准备发送给浏览器的数据---header
                response = "HTTP/1.1 200 OK\r\n"
                response += "\r\n"
                new_socket.send(response.encode("utf-8"))
                new_socket.send(html_content)
        
        else:
            # 2.2 如果是以.py结尾，那么就认为是动态资源的请求
            env = dict()
            env['PATH_INFO'] = file_name
            # {"PATH_INFO": "/index.py"}
            # body = dynamic.mini_frame.application(env, self.set_response_header)
            body = self.application(env, self.set_response_header)
            header = "HTTP/1.1 %s\r\n" % self.status
            for temp in self.headers:
                header += "%s:%s\r\n" % (temp[0], temp[1])
            header += "\r\n"
            response = header+body
            new_socket.send(response.encode("utf-8"))

        new_socket.close()

    def set_response_header(self, status, headers):
        self.status = status
        self.headers = [("server", "mini_web v8.8")]
        self.headers += headers

    def run_forever(self):
        while True:
            new_socket, client_addr = self.tcp_server_socket.accept()
            p = multiprocessing.Process(target=self.service_client, args=(new_socket,))
            p.start()
            new_socket.close()

        # 关闭监听套接字
        self.tcp_server_socket.close()

def main():
    if len(sys.argv) == 3:
        try:
            port = int(sys.argv[1])      # 7070
            
            frame_app_name = sys.argv[2] # frame:name
            print('11111111111111',port,'\t',frame_app_name)
            ret = re.match(r'([^:]+):(.*)', frame_app_name)
            if ret:
                frame_app_name = ret.group(1)
                app_name = ret.group(2)
                print('2'*10,frame_app_name,'\t',app_name)
            else:
                print('参数错误, 请按照以下方式运行', '\n', 'python xxx.py port frame:name')
                return
                
        except Exception as ret:
            print('端口输入错误')
            return
    else:
        print('长度不够, 请按照以下方式运行:','\n','python xxx.py port frame:name')
        return
        
    with open("./web_server.conf") as f:
        conf_info = eval(f.read())
    print(type(conf_info), conf_info)
    
    # "static_path":"./static",
    # "dynamic":"./dynamic"

    # 指定在dynamic里面找模块
    sys.path.append(conf_info['dynamic'])
    # 指定在dynamic里面找模块，指定按frame_app_name找
    frame = __import__(frame_app_name)   # 返回值标记着导入的这个模块
    app = getattr(frame, app_name)       # 此时app指向 dynamic/mini_frame模块中的函数
    wsgi_server = WSGIServer(port, app, conf_info['static_path'])
    wsgi_server.run_forever()

if __name__ == "__main__":
    main()
```
## Mini框架  
```Python
def index():
    with open('./templates/index.html','r',encoding='utf-8') as  f:
        return f.read()

def login():
    with open('./templates/center.html', 'r', encoding='utf-8') as  f:
        return f.read()

def application(env, start_response):
    start_response('200 OK', [('Content-Type', 'text/html;charset=utf-8')])
    file_name = env['PATH_INFO']
    # file_name = "/index.py"
    if file_name == "/index.py":
        return index()
    elif file_name == "/center.py":
        return login()
    else:
        return 'Hello World! kiss my lady 爱你三千零一遍'

```
