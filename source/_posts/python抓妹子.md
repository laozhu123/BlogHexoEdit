---
title: python爬虫初涉
date: 2018-02-06 14:17:44
categories: "python"
tags:
     - python
     - 爬虫
     - 技术
---

![enter image description here](http://op0dvu7tu.bkt.clouddn.com/helo004.jpg)

### 文章简介

**本文将介绍如何使用python对www.mzitu.com中所有的图片的爬取以及存储到本地最后我们会得到如下图1所示**


![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206142745.png)

是不是很心动，接下来就让我们开始python爬虫之旅，本期内容将会从单线程使用到多线程，从自己写python到使用scrapy包，以及分布式redis的使用，当然这些都是后话了，当前我们的目标是要爬去这个网站上的美图。

### 环境介绍

废话少说接下来就是我们的基础环境部分
**1.Python -- 3.6.1**（这个版本其实要区别的就是python2和python3啦，我使用的python3）

**2.Requests** （看名字就知道这个是用来进行网络请求用的，这里的request是rllib包中的，之后我们要学的scrapy中也有自己想要的request，到时候两个不要搞混了）

**3.beautifulsoup** （当然数据获取到了之后，我们要对数据进行提取，解析就是通过beautifulsoup来进行，当然我们自己也可通过正则表达式来对数据进行过滤，如果你的正则水平不错的情况下） 

**4.LXML** 一个HTML解析包 用于辅助beautifulsoup解析网页  

--------------
**上面的模块需要 单独安装，下面几个就不用啦。**

--------------
**5.OS 系统内置模块** （这个玩意是系统内置的，在本文中我们通过它来将图片存储在本地）

**6.PyCharm**   一个草鸡好用的PythonIDE工具 、真滴。


### 模块的安装

再使用pip指令之前我们需要安装python。（如果我们安装python时选择**不将**python写入环境变量的话，那么我们还需要将“文件夹\python36”和“文件夹\python36\Scripts”写入path中，这样我们才能在控制台中使用python和pip指令）

**接下来就用指令安装模块**
    
     pip install requests   
     pip install beautifulsoup4  
     pip install lxml

大致结果就如下图所示
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206150606.png)

当python + 3个模块 + PyCharm安装完成之后我们就可以开始我们本次抓取的代码编写了。

### 爬虫编写
**首先我们先整理一下爬图片的步骤：**

1.我们需要一个目标网站（www.mzitu.com）
2.我们需要从一个网页中找出接下来要去的网页的链接地址，通过beautifulsoup来获取
3.获取网页中的图片地址
4.下载图片到本地


**步骤1：**
www.mzitu.com网站的截图如下，这里选择了www.mzitu.com/all作为起始网页，因为在该网页中包含了网站中所有图片组图的链接链接地址
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206160047.png)


**步骤2：**
推荐使用chrome浏览器来进行网页源码的查看
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206160522.png)

通过观察我们可以发现接下来的链接地址在所在的**href**是被**div class=all**所包含着的，所有我们可以使用这一点来找到所有的url地址

**步骤3：**
接下来我们就进入到某个链接里面去看图了
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206161232.png)
相应图片的地址被包含在**div class='main-image'**中


注意着个有一套图（一般她们拍写真都有很多张的），所以呢我们就看下接下来的这张图
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180206161358.png)
着张图里的**href**就是套图里其他图片所在网页的地址了，有了这个我们就可以获取接下来的图片了，其中通过观察可以发现**href**被包在**div class=‘pagenavi’**中

**步骤4：**
**开始代码编写**

我们需要在代码中引入相应的模块

    import requests
	from bs4 import BeautifulSoup
	import os

获取起始网页内容

    html = self.request(url)
    
这里的request是下面的这个方法

        def request(self, url):  ##这个函数获取网页的response 然后返回
	        headers = {
            'User-Agent': "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1"}
	        content = requests.get(url, headers=headers)

	        return content

获取步骤1中内容（也就是所有套图的地址），并进一步发送请求获取到套图首张图片所在网页

    all_a = BeautifulSoup(html.text, 'lxml').find('div', class_='all').find_all('a')
    for a in all_a:       
	    title = a.get_text() #取出a标签的文本       
	    href = a['href'] #取出a标签的href 属性，也就是套图地址
	    self.html(href, "e:\\pic\\" + path)
	
相应的html方法

        def html(self, href, path):  ##这个函数是处理套图地址获得图片的页面地址
	        html = self.request(href)
	        helo = BeautifulSoup(html.text, 'lxml').find_all('span')
	        if len(helo) < 10:
	            return
	        max_span = helo[10].get_text()
	        for page in range(1, int(max_span) + 1):
	            page_url = href + '/' + str(page)
	            self.img(page_url, path)  ##调用img函数

上面的html方法中的后半截是执行了步骤3中的后半部分，也就是遍历了套图中其他图片所在的网页，相应的img方法如下

        def img(self, page_url, path):  ##这个函数处理图片页面地址获得图片的实际地址
	        img_html = self.request(page_url)
	        helo = BeautifulSoup(img_html.text, 'lxml').find('div', class_='main-image')
	        if helo is not None:
	            helo1 = helo.find('img')
	            if helo1 is not None:
	                # do some thing you need
	                img_url = helo1['src']
	                self.saveImg(img_url, path)

        def saveImg(self, url, path):
	        getHeaders = {
	            'Host': 'i.meizitu.net',
	            'Connection': 'Keep-Alive',
	            'Accept': 'image/webp,image/apng,image/*,*/*;q=0.8',
	            'Upgrade-Insecure-Requests': '1',
	            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36',
	            'Referer': 'http://www.mzitu.com/',
	            'Accept-Encoding': 'gzip, deflate',
	            'Accept-Language': 'zh-CN,zh;q=0.9'
	        }
	        response = requests.get(url, headers=getHeaders)
	        
	        if (response.status_code == 404):  # 若404错误，递归get，尝试非重定向方式获取
	            response = requests.get(url, headers=getHeaders, allow_redirects=False)
	            if (response.status_code == 302):  # 302表示访问对象已被移动到新位置，但仍按照原地址进行访问（造成404错误）。
	                name = url[-9:-4]
	                redirectUrl = response.headers['location']  # 因此需在响应头文件中获取重定向后地址
	                response = requests.get(redirectUrl)
	                fp = open(name + ".jpg", 'ab')
	                fp.write(response.content)
	                fp.close()
	        else:
	            os.chdir(path)
	            name = url[-9:-4]
	            fp = open(name + ".jpg", 'ab')
	            fp.write(response.content)
	            fp.close()
	            self.picNum = self.picNum + 1
	            print(self.picNum)
	            
方法saveImg执行了步骤4，也就是将图片保存在了本地。



### 总结

本片文字是在之前微信上看到的一篇文章改写的，当时照着那篇文字将代码写了一遍之后发现，保存在本地的图片都被篡改了，也就是被防盗链了，而后就只能修改request中的一些参数来确保准确性。在上面这些完成后，发现对爬虫有了一点的了解之后想着是不是可以使用多线程的方式来爬去，就又看了多线程的写法，下面贴出多线程的代码，这里使用的线程池。

**多线程代码**

    import requests
	from bs4 import BeautifulSoup
	import os
	import threading
	import threadpool
	
	
	class mzitu():
	    num = 0
	    picNum = 0

	    def all_url(self, url):
	        self.cv = threading.Condition()
	        html = self.request(url)  ##调用request函数把套图地址传进去会返回给我们一个response
	        all_a = BeautifulSoup(html.text, 'lxml').find('div', class_='all').find_all('a')
	
	        task_pool = threadpool.ThreadPool(50)
	        requests = threadpool.makeRequests(self.nice, all_a)
	        for req in requests:
	            task_pool.putRequest(req)
	        task_pool.wait()

	    def nice(self, a):
	        title = a.get_text()
	        path = str(title).replace("?", '_')  ##我注意到有个标题带有 ？  这个符号Windows系统是不能创建文件夹的所以要替换掉
	        if self.mkdir(path):  ##调用mkdir函数创建文件夹！这儿path代表的是标题title哦！！！！！不要糊涂了哦！
	            os.chdir("e:\\pic\\" + path)  ##切换到目录
	            href = a['href']
	            self.html(href, "e:\\pic\\" + path)  ##调用html函数把href参数传递过去！href是啥还记的吧？ 就是套图的地址哦！！不要迷糊了哦！

	    def html(self, href, path):  ##这个函数是处理套图地址获得图片的页面地址
	        html = self.request(href)
	        helo = BeautifulSoup(html.text, 'lxml').find_all('span')
	        if len(helo) < 10:
	            return
	        max_span = helo[10].get_text()
	        for page in range(1, int(max_span) + 1):
	            page_url = href + '/' + str(page)
	            self.img(page_url, path)  ##调用img函数

	    def img(self, page_url, path):  ##这个函数处理图片页面地址获得图片的实际地址
	        img_html = self.request(page_url)
	        helo = BeautifulSoup(img_html.text, 'lxml').find('div', class_='main-image')
	        if helo is not None:
	            helo1 = helo.find('img')
	            if helo1 is not None:
	                # do some thing you need
	                img_url = helo1['src']
	                self.saveImg(img_url, path)
	
	    def mkdir(self, path):  ##这个函数创建文件夹
	        path = path.strip()
	        if path.__contains__("妲己"):
	            return False
	        isExists = os.path.exists(os.path.join("e:\\pic\\", path))
	        if not isExists:
	            os.makedirs(os.path.join("e:\\pic\\", path))
	            return True
	        else:
	            return False
	
	    def request(self, url):  ##这个函数获取网页的response 然后返回
	        headers = {
	            'User-Agent': "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 (KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1"}
	        content = requests.get(url, headers=headers)
	
	        return content
	
	    def saveImg(self, url, path):
	        getHeaders = {
	            'Host': 'i.meizitu.net',
	            'Connection': 'Keep-Alive',
	            'Accept': 'image/webp,image/apng,image/*,*/*;q=0.8',
	            'Upgrade-Insecure-Requests': '1',
	            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36',
	            'Referer': 'http://www.mzitu.com/',
	            'Accept-Encoding': 'gzip, deflate',
	            'Accept-Language': 'zh-CN,zh;q=0.9'
	        }
	
	        response = requests.get(url, headers=getHeaders)
	        if (response.status_code == 404):  # 若404错误，递归get，尝试非重定向方式获取
	            response = requests.get(url, headers=getHeaders, allow_redirects=False)
	            if (response.status_code == 302):  # 302表示访问对象已被移动到新位置，但仍按照原地址进行访问（造成404错误）。
	                name = url[-9:-4]
	                redirectUrl = response.headers['location']  # 因此需在响应头文件中获取重定向后地址
	                response = requests.get(redirectUrl)
	                fp = open(name + ".jpg", 'ab')
	                fp.write(response.content)
	                fp.close()
	        else:
	            os.chdir(path)
	            name = url[-9:-4]
	            fp = open(name + ".jpg", 'ab')
	            fp.write(response.content)
	            fp.close()
	            self.picNum = self.picNum + 1
	            print(self.picNum)
	            
	Mzitu = mzitu()  ##实例化
	Mzitu.all_url('http://www.mzitu.com/all')  ##给函数all_url传入参数  你可以当作启动爬虫（就是入口）







       




