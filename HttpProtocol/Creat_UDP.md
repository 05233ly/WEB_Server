## 网络通信入门  
### 设置网络通信

    大部分初学习的同学，可能都没有办法买上一台服务器，大多数都是开的虚拟机，充当服务器或客户端； 这里先讲解下怎么开启虚拟机和windows主机之间的通信。  
开启虚拟机的桥接模式：  
1、虚拟机设置为桥接模式；  
![桥接模式](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/brige.jpg)  
   坑点： 这里虚拟机有可能在你选择‘桥接模式’后，无法保存；可以直接联系我，137 3816972QQ；  
   
2、查看主机网络状态  
重要命令： ①、Linux： ifconfig  
          ②、windows： ipconfig  
         
3、查看是否可以通信  
方法： windows在cmd命令下，输入ping ip地址， 如果有显示，则是成功通信；
       Linux在base下， 输入ping ip地址，同上；
  
4、通信监测  
![网络助手](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/webhelper.jpg)  
 稍后写一段简单代码来测试。  
 
 
 ### 端口  
 解释：端口不是随便使用的，而是按照一定规定进行分配，端口的分配有好几种，这里说其中的重要的两种；  
 其中两种： 知名端口  
             众所周知的端口号，从0到1023， 不能随便使用，比如说里面的80端口分配给http服务， 21端口分配给FTP服务；  
           动态端口： 1024 - 65535；  
             系统随机分配；  
 
 ### Socket简介  
### - [ 1、不同的电脑进程间如何通信？ ]  
     IP地址 • 协议 • 端口  
### - [2、只有socket才能实现网络功能，socket是完成网络通信必备的东西；  ]  
 
 ### - [3、基本写法]  
  TCP = socket.socket(socket.AF_INFI, socket.SOCK_STREN)  
  UDP = socket.socket(socket.AF_INFI, socket.SOCK_DGRAM)  

### 接下来请看下文，基本收发信息

- [客户端与服务端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/UDP_1.md)
 
 
 
 
