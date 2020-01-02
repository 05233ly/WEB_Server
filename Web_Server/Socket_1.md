基础版服务器  
=====
我们先来看一段代码: 
```Python
def service(new_socket):
    request = new_socket.recv(1024).splitlines()
    for i in request:
        print('浏览器请求头: ', i)
    response = 'HTTP/1.1 200 OK\r\n'
    response += "\r\n"
    response += '<h1>Hahahahahahaha<h1>'
    new_socket.send(response.encode('utf-8'))
    new_socket.close()
    
def main():
    tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
    tcp.bind(('', 8080))
    tcp.listen(128)
    while True:
        c, addr = tcp.accept()
        service(c)

if __name__ == '__main__':
    main()
```
这个代码分成了两个部分, `service`和`main`   
显然, `mian`提供了转接服务, 监听套接字, 如果有服务就让另外一个人处理, 提高了单线程堵塞效率  
## 第一段: `mian`  
```Python
tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
tcp.bind(('', 8080))
tcp.listen(128)
while True:
    c, addr = tcp.accept()
    service(c)
```
其中有几点细节:  
* 细节一:   
```Python
tcp.setsockopt(socket.SOCK_STREAM, socket.SO_REUSEADDR, 1)
```
这提供端口能够快速释放, 如果不写这句话, 断开连接后, 得等上一会儿才能用这个端口, 显然节约了调试的时间   
* 细节二:   
```Python
while True:
    c, addr = tcp.accept()
    service(c)
```
在关键位置, 程序最薄弱, 最耗时的环节, 我们加入了while, 保证一但有客户端连接, 能够快速转发并且重置执行等待   

## 第二段: `service`    
```Python
def service(new_socket):
    request = new_socket.recv(1024).splitlines()
    for i in request:
        print('浏览器请求头: ', i)
    response = 'HTTP/1.1 200 OK\r\n'
    response += "\r\n"
    response += '<h1>Hahahahahahaha<h1>'
    new_socket.send(response.encode('utf-8'))
    new_socket.close()
```
* 细节一:  
浏览器发过来的数据我们需要`splitlines`, 转换成字典遍历打印更直观, str形式挤在一起的信息人没法看  

* 细节二:  
```Python
response += "\r\n"
```
一个html页面, 需要遵守一定的规则才能解析, 而发送的字符串怎么让浏览器解析那是头, 哪是尾呢?   
这里, 加入`"\r\n"`后, 就可以区分头和尾了     
后面就是常规操作了,  发送的时候压缩下, 然后这个关闭套接字    
这时`service`函数就完成了任务    
