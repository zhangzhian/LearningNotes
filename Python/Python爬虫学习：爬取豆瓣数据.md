# Python爬虫学习：爬取豆瓣数据
Python的学习起源于帮助他人找bug，现阶段可能会做一些不同爬虫相关的Demo，后续如果有时间继续深入学习，近期没有时间，现不列于计划之内。
学习主要途径和内容：[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
学习过程中的一些demo：[我的GitHub](https://github.com/zhangzhian/learn-python3)

--------

现在开始总结[豆瓣电影 Top 250](https://movie.douban.com/top250) 爬取数据的过程
豆瓣电影 Top 250 url：https://movie.douban.com/top250
获取的数据包括排名，电影名称，导演，年代，评分
数据保存到Excel中
## 1.分析网站
以下是基于Mac下Safari浏览器，其余浏览器类似。
首页如下：
![豆瓣Top250首页](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E8%B1%86%E7%93%A3%E7%94%B5%E5%BD%B1Top250%E9%A6%96%E9%A1%B5.png?raw=true)
### 首页我们来确定查询的url
爬取的首页 https://movie.douban.com/top250
观察豆瓣首页，只有25条数据，而页面底部有分页
![底部分页](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E8%B1%86%E7%93%A3%E5%88%86%E9%A1%B5.png?raw=true)
我们选择第二页，然后观察url变化

首页加上了start和filter提交数据

filter参数内容为空，暂不考虑

start=25，可以推测为从第25条数据开始，改变数字，测试一下，我们会发现推测是正确的

确定url为 https://movie.douban.com/top250?start= 

start后加开始获取第一条数据的编号
![url](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E5%88%86%E9%A1%B5url.png?raw=true)
### 查看源文件
在该网页鼠标右键，选择显示页面源文件
![显示页面源文件](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E6%98%BE%E7%A4%BA%E9%A1%B5%E9%9D%A2%E6%BA%90%E6%96%87%E4%BB%B6.png?raw=true)

页面源文件

![页面源文件显示](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E9%A1%B5%E9%9D%A2%E6%BA%90%E6%96%87%E4%BB%B6%E6%98%BE%E7%A4%BA.png?raw=true)
如上图即为该页面的源文件
接下来我们分析源文件
### 分析HTML元素
我们以如下一个模块做参考进行分析：
![单个例子](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E5%8D%95%E4%B8%AA%E4%BE%8B%E5%AD%90.png?raw=true)
寻找对应的源码
![源码](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E5%8D%95%E4%B8%AA%E4%BE%8B%E5%AD%90%E8%AF%A6%E7%AE%80.jpeg?raw=true)

如上图所示

- 首先，我们在右上侧搜索名称，然后回车，可以在左边看到搜索结构

- 然后我们来分析HTML源码

	如图1，2，3这三个部分
	
	第一部分：“hd”的div中包含了电影名称
	
	第二部分：“bd”的div中包含了导演，时间，主演等
	
	第三部分：“star”的div中包含了评分和评价数

- 再来看我们需要获取的数据

	排名：根据数据的顺序
	
	电影名称：div(hd)->a->span
	
	导演：div(bd)->p
	
	年代：div(bd)->p
	
	评分：div(star)->span

- 获取到数据后则需要解析和保存数据

分析完这几部分，即可写代码了
### 代码


```python

import requests
import xlwt
from bs4 import BeautifulSoup
# https://movie.douban.com/top250
# 豆瓣电影 Top 250 爬取

# 存储爬去到的数据
movie_list = []
director_list = []
time_list = []
star_list = []

for i in range(0, 10):

    link = 'https://movie.douban.com/top250?start=' + str(i*25)
    res = requests.get(link, timeout=10)
    # 该网站不需要使用headers也可以
    # res = requests.get(link, headers=headers, timeout=10)
    soup = BeautifulSoup(res.text, "lxml")
    # 寻找div class为hd的标签
    div_list = soup.find_all('div', class_='hd')
    div1_list = soup.find_all('div', class_='bd')
    div2_list = soup.find_all('div', class_='star')
    # 解析数据
    for each in div_list:
        movie = each.a.span.text.strip()
        movie_list.append(movie)

    for each in div1_list:
        info = each.p.text.strip()
        if len(info) < 3:
            continue
        time_start = info.find('20')
        if time_start < 0:
            time_start = info.find('19')
        end = info.find('...')
        gap_value = time_start - end
        time = info[end + gap_value:end + gap_value + 4]
        time_list.append(time)

        end = info.find('主')
        director = info[4:end - 3]
        director_list.append(director)

    for each in div2_list:
        info = each.text.strip()
        star = info[0:3]
        star_list.append(star)

# 保存到xls文件中
file = xlwt.Workbook()

table = file.add_sheet('sheet name')
table.write(0, 0, "排名")
table.write(0, 1, "电影")
table.write(0, 2, "时间")
table.write(0, 3, "导演")
table.write(0, 4, "评分")

for i in range(len(star_list)):
    table.write(i + 1, 0, i + 1)
    table.write(i + 1, 1, movie_list[i])
    table.write(i + 1, 2, time_list[i])
    table.write(i + 1, 3, director_list[i])
    table.write(i + 1, 4, star_list[i])

file.save('data_douban_1.xls')

```
### 抓取到的数据
![抓取到的数据](https://github.com/zhangzhian/learn-python3/blob/master/res/PythonCrawler/%E6%8A%93%E5%8F%96%E7%9A%84%E6%95%B0%E6%8D%AE.png?raw=true)

[Demo 地址](https://github.com/zhangzhian/learn-python3/tree/master/PythonCrawler)

---
> 最后，说一句题外话，在修改bug的时候，debug是很重要的手段，单步分析存在问题的部分，很容易发现bug，遇到bug不要慌，debug走一波。