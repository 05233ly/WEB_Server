长链接和短链接  
=====

## 长连接使用方法  
我们先看一段代码   
```python
# 实现网页标签返回数据
import socket, re
import time

def service_client(client_socket, recv_data):
    request_lines = recv_data.splitlines()
    ret = re.findall(r"(/[^ ]*)", request_lines[0])
    file_nema = ret[0]
    if file_nema:
        if file_nema == '/':
            file_nema = "/index.html"

    response = 'HTTP/1.1 200 OK\r\n'
    response += "Content-Length:%d\r\n" % len(response)
    response += "\r\n"
    try:
        path = r'D:/web' + file_nema
        f = open(path, "rb")
    except:
        response = 'HTTP/1.1 404 NOT FOUND\r\n'
        response += "\r\n"
        response += "----Not found----"
        client_socket.send(response.encode('utf-8'))
    else:
        response_body = f.read()
        f.close()
        response_header = 'HTTP/1.1 200 OK\r\n'
        response_header += "Content-Length:%s\r\n" % len(response_body)
        response_header += "\r\n"
        response = response_header.encode('utf-8')
        client_socket.send(response)

def main():
    tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_server.bind(('', 8080))
    tcp_server.listen(128)
    tcp_server.setblocking(False)
    client_socket_list = list()
    while True:
        time.sleep(0.5)
        try:
            new_socket, client_addr = tcp_server.accept()
        except Exception as ret:
            pass
        else:
            new_socket.setblocking(False)
            client_socket_list.append(new_socket)
            
        for client_socket in client_socket_list:
            try:
                recv_data = client_socket.recv(1024).decode('utf-8')
            except  Exception as ret:
                pass
            else:
                if recv_data:
                    print('套接字请求')
                    service_client(client_socket, recv_data)
                else:
                    client_socket.close()
                    client_socket_list.remove(client_socket)
    tcp_server.close()

if __name__ == '__main__':
    main()
```
细节部分:  
```python
header = 'HTTP/1.1 200 OK\r\n'
header += "Content-Length:%s\r\n" % len(response_body)
header += "\r\n"
```
浏览器会看一看网页一共有多长, `len(response_body)`, 文件都收到后, 那个圈圈就不会转了  
浏览器把全部文件收到后, 主动断开连接  


### 抽象图:  
![long_and_short1](https://github.com/KissMyLady/Web-of-Python/blob/master/Img/long_short.jpg)  


### 作用范围:  
  1、像一般游戏服务，就是使用的长链接，例如游戏穿越火线， 王者农药， LOL等。  
  2、HTTP服务: 浏览器与服务器的约定， 约定(默认)请求头1.0为短连接， 1.1为长链接。  

### HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接  
先看看经典的三次握手图  
![3times_hands](https://github.com/KissMyLady/Web-of-Python/blob/master/Img/3times_hands.png)  

经典的四次挥手图  
![4times_byby2](https://github.com/KissMyLady/Web-of-Python/blob/master/Img/4times_byby2.png)  


### 详细说明  
![long_and_short2](https://github.com/KissMyLady/Web-of-Python/blob/master/Img/long_and_short.jpg)  
#### 1、短链接  
短连接的操作步骤是：  
建立连接——数据传输——关闭连接...建立连接——数据传输——关闭连接  
##### 优点：`管理起来比较简单，存在的连接都是有用的连接，不需要额外的控制手段。 ` 
##### 缺点：如果客户请求频繁，将在TCP的建立和关闭操作上浪费时间和带宽。  


#### 2、长链接  
长连接的操作步骤是：  
建立连接——数据传输...（保持连接）...数据传输——关闭连接  
##### 优点：长连接可以省去较多的TCP建立和关闭的操作，减少浪费，节约时间。对于频繁请求资源的客户来说，较适用长连接；  
##### 缺点： 1、存活功能的探测周期太长；  2、它只是探测TCP连接的存活；(Client与server之间的连接如果一直不关闭的话，会存在一个问题，随着客户端连接越来越多，server早晚有扛不住的时候，这时候server端需要采取一些策略，## 如关闭一些长时间没有读写事件发生的连接，这样可 以避免一些恶意连接导致server端服务受损；如果条件再允许就可以以客户端机器为颗粒度，限制每个客户端的最大长连接数，这样可以完全避免某个蛋疼的客户端连累后端服务 ##)  


#### 3、如何权衡取舍？  
  长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况；例如：数据库的连接用长连接， 如果用短连接频繁的通信会造成socket错误，而且频繁的socket 创建也是对资源的浪费。  
http服务一般都用短链接；长连接对于服务端来说会耗费一定的资源，而像WEB网站这么频繁的成千上万甚至上亿客户端的连接用短连接会更省一些资源，如果用长连接，而且同时有成千上万的用户，如果每个用户都占用一个连接的话，那可想而知吧。所以并发量大，但每个用户无需频繁操作情况下需用短连好。   

