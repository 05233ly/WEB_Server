List仓库遍历
========
我们先看一段代码:   
```Python
import socket
import re, time

def service_client(request,  client_socket):
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
        print('这是路径', path)
        f = open(path, "rb")
    except:
        response = 'HTTP/1.1 404 NOT FOUND\r\n'
        response += "\r\n"
        response += "----Not found----"
        client_socket.send(response.encode('utf-8'))
    else:
        html_content = f.read()
        f.close()
        client_socket.send(response.encode('utf-8'))
        client_socket.send(html_content)
        client_socket.send(response.encode('utf-8'))
        client_socket.close()

def main():
    tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_server.bind(('', 8080))
    tcp_server.listen(128)
    tcp_server.setblocking(False)
    client_socket_list = list()

    while True:
        time.sleep(0.5)
        try:
            new_socket, new_addr = tcp_server.accept()
        except Exception as ret:
            print('没有来')
        else:
            print('来了新的')
            new_socket.setblocking(False)
            client_socket_list.append(new_socket)
        
        for client_socket in client_socket_list:
            print('有%s个连接' %(str(len(client_socket_list))))
            try:
                recv_data = client_socket.recv(1024).decode('utf-8')
            except Exception as ret:
                print("没有发来数据")
            else:
                if recv_data:
                    service_client(recv_data, client_socket)
                else:
                    client_socket.close()
                    client_socket_list.remove(client_socket)
                    
if __name__ == '__main__':
    main()
```
前面是一段常规代码, 我们来看看一点不一样的:  
```Python
client_socket_list = list()
tcp_server.setblocking(False)
while True:
    time.sleep(0.5)
    try:
        new_socket, new_addr = tcp_server.accept()
    except Exception as ret:
        print('没有来')
    else:
        print('来了新的')
        new_socket.setblocking(False)
        client_socket_list.append(new_socket)
```
设置tcp非堵塞, 显然, 当没有客户端到来的时候, 一定会报错  
如果不报错, 我们就认为来了新的客户端:  
于是:   
```Python
for client_socket in client_socket_list:
    print('有%s个连接' % (str(len(client_socket_list))))
    try:
        recv_data = client_socket.recv(1024).decode('utf-8')
    except Exception as ret:
        print("没有发来数据")
    else:
        if recv_data:
            service_client(recv_data, client_socket)
        else:
            client_socket.close()
            client_socket_list.remove(client_socket)
```
这里使用list当做仓库, 从仓库挨个查看是否有客户端通信, 显然, 这种方式效率还是很低下   



