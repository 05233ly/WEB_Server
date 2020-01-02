Web和Socket通信--单线程--返回指定网页
======
我们先来看一段代码:   
```Python
import socket
import re, os

def service(new_socket):
    request = new_socket.recv(1024).decode('utf-8')
    request_lines = request.splitlines()
    ret = re.findall(r"[^/]+(/[^ ]*)", request_lines[0])
    file_nema = ret[0]
    file_nema = ret_lv
    if file_nema == '/':
        file_nema = "/index.html"

    response = 'HTTP/1.1 200 OK\r\n'
    response += "\r\n"
    
    try:
        path = r'D:/web服务器' + file_nema
        f = open(path, "rb")
        
    except:
        response = 'HTTP/1.1 404 NOT FOUND\r\n'
        response += "\r\n"
        response += "----Not found----"
        new_socket.send(response.encode('utf-8'))

    else:
        html_content = f.read()
        f.close()
        new_socket.send(response.encode('utf-8'))
        new_socket.send(html_content)
        new_socket.send(response.encode('utf-8'))
        new_socket.close()

def main():
    print('开始连接')
    tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp.bind(('', 8080))
    tcp.listen(128)
    while True:
        new_socket, client_addr = tcp.accept()
        service(new_socket)

if __name__ == '__main__':
    main()
```
可以看到, 我们的代码分成了`main`和`service`两部分   
下面我们来分析分析两个代码块:  

## 函数`main`  
```Python
def main():
    print('开始连接')
    tcp = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp.bind(('', 8080))
    tcp.listen(128)
    while True:
        new_socket, client_addr = tcp.accept()
        service(new_socket)
```
`main`函数是我们熟悉的, 通过while控制转发, 将耗时任务外包给while    
让while帮我们完成解堵塞   

## 函数`service` 
```Python          
def service(s):
    request = s.recv(1024).decode()
    lines = request.splitlines()
    ret = re.findall(r"[^/]+(/[^ ]*)", lines[0])
    file_nema = ret[0]
    if file_nema == '/':
        file_nema = "/index.html"
    response = 'HTTP/1.1 200 OK\r\n'
    response += "\r\n"
    
    try:
        path = r'D:/web服务器' + file_nema
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
```
这段代码看起来比较多, 实际上逻辑关系并不难理解:   
### 1. 解析浏览器请求的数据   
```Python     
request = s.recv(1024).decode()
lines = request.splitlines()
ret = re.findall(r"[^/]+(/[^ ]*)", lines[0])
file_nema = ret[0]
```
### 2. 判断浏览器请求的什么        
```Python    
if file_nema == '/':
    file_nema = "/index.html"
```
#### 细节一:   
这里使用判断, 如果浏览器什么都没写, 就返回固定页面  
像我们浏览网站, 你肯定不会在域名上敲点什么再去访问, 所以这个是必要的  

### 3. 接下来   
```Python  
response = 'HTTP/1.1 200 OK\r\n'
response += "\r\n"
try:
    path = r'D:/web服务器' + file_nema
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
```
#### 细节一:   
使用了try增加系统容错率, 电脑不比人, 找不到文件或是文件打开错误搞不好整个系统就崩溃掉了  
所以我们这里使用try包一下   

#### 细节二:  
```Python
html_content = f.read()
```
一个网页才几kb的样子, 所以可以直接读取, 不用担心  

#### 细节三:  
```Python
html_content = f.read()
f.close()
new_socket.send(html_content)
```
网页通过read读出来就是二进制文件了, 不用再需要encode了   
另外, 读取文件后要记得关闭   

