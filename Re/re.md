### Re学习  
Re是正则的简称，写上一定的规则就能从万千文本数据中提取想要的内容， 可以看做是筛子，只要能经过筛选的内容。  

Re学习分为两个主要知识点，第一个match，search，findall等几个高级函数的学习，这是非常简单的；  
第二个学习点就是正则表达式本身了，包含特殊符号和数字字母的组合，通过这些组合，就可以定制我们的"数据网"了，筛选出想要的数据；  

## match方法  
从头开始匹配，要是不符合，直接返回None;  
![match](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/match1.jpg)  
请注意，↑ 这里的resule如果有结果，且不加上.group()的话， 打印的结果会将是一个 <re.Match object; span=(0, 5), match='hello'>类型；  

如果匹配不到数据，返回None  
![match2](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/match2.jpg)  

## search方法  
全局匹配第一个符合的字符串;  
![search](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/search1.jpg)  

## findall方法  
找到所有匹配的字符串，返回一个列表，如果没有找到匹配的，则返回空列表;  
![findall](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/findall1.jpg)  

### compile方法  
预先把表达式编译好，这样可以节省不少时间;  
![compile](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/compile1.jpg)  


## 补充  
这些方法都有三个参数，其中前面两个是正常参数，第三个是可选模式(修正参数)，用来控制匹配的模式；  
![re_3](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/img/re_3.jpg)  

### 接下来
- [re表达式](https://github.com/KissMyLady/Web-of-Python/blob/master/Re/re_text.md)  
