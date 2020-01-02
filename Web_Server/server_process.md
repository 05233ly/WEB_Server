Web和Socket通信--开启多进程--继承实现网页标签返回数据  
=====
我们先来看一段代码:  
```Python
import multiprocessing
import socket, re

def service_client(new_socket):
    request = new_socket.recv(1024).decode()
    request_lines = request.splitlines()
    print(request_lines[0])
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
        new_socket.send(response.encode())
    
    else:
        html_content = f.read()
        f.close()
        new_socket.send(response.encode())
        new_socket.send(html_content)
        new_socket.send(response.encode())
        new_socket.close()

def main():
    tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_server.bind(('', 8080))
    tcp_server.listen(128)
    while True:
        s, client_addr = tcp_server.accept()
        p = multiprocessing.Process(target=service_client, args=(s,))
        p.start()
        new_socket.close()
    tcp_server.close()

if __name__ == '__main__':
    main()
```
承接[上文的-网页标签返回数据](https://github.com/KissMyLady/WEB_Server/blob/master/Web_Server/server_one.md), 通过上面的方法, 我们就可以得到一个服务器, 但浏览器访问时, 就可以让服务器做到自由访问了   
但是, 可以更进一步, 提高效率吗?   
这里我们采用了多进程,  但是这种效果并不好, 进程占用资源大, 开辟空间慢, 访问数量多了还是会挂掉  

先看下多进程怎么实现的:   
```Python
def main():
    tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_server.bind(('', 8080))
    tcp_server.listen(128)
    while True:
        s, client_addr = tcp_server.accept()
        p = multiprocessing.Process(target=service_client, args=(s,))
        p.start()
        new_socket.close()
        
    tcp_server.close()
```
还是老样子, 通过while方法转发服务, 把accpet堵塞这种耗时的东西外包   
这里的细节就来了     
细节一:   
```Python
p = multiprocessing.Process(target=service_client, args=(s,))
```
启动进程方式, 线程启动方式也是大同小异, 不过是换成了`threading.Thread`   
请注意: 后面的args是元祖, 函数不要带括号, 这是新手容易犯的错误   

细节二:  
```
tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
....
while True:
	.....
    tcp.close()
```
这的tcp需要再关闭一次, 为什么?  进程是写实copy, 把这个资源, 环境递归复制一份运行了  
进程就相当于一块独立的空间, 独立的空间关闭了, 这里本土的空间还没关闭, 需要关闭   

我们平常说的多核cpu, 就是多个cpu一起运行, 提高效率, 多进程就可以最大限度利用多核cpu的效率    

最后一点, 单就多进程实现服务器来说, 访问量一大, 肯定会挂掉
