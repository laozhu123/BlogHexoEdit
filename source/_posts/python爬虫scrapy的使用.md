---
title: python爬虫scrapy的使用
date: 2018-02-06 17:13:44
categories: "python"
tags:
     - python
     - 爬虫
     - 技术
---

![enter image description here](http://op0dvu7tu.bkt.clouddn.com/helo005.jpg)

### 简介
**本文将记录一次使用scrapy进行网页数据爬取的经历。**

### 环境与安装
**环境**

python -- 3.6.1（区别python2和python3就行了，两者的语法在有些地方有区别）
scrapy -- 1.5.0 （这个是根据你的python版本来选择的）
twisted
wheel
pywin32


**安装python**
这里就不再赘述了，无非就是到python的官方网站下载相应安装包安装。如果要看的话，可以看blog里的另外一篇文章，也就是《python爬妹子》这一篇，或者可以看这个网址[python安装](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001374738150500472fd5785c194ebea336061163a8a974000)

**安装scrapy**
一般使用windows电脑安装时会出现安装失败的情况，故而我们需要到[这个网站](https://pypi.python.org/pypi/Scrapy/1.5.0)下载相应的版本，来进行安装在安装之前我们还需要先安装wheel只需要在cmd中敲入：

    pip install wheel

![|center](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207153010.png)

而后我们到https://pypi.python.org/pypi/Scrapy/1.5.0下载相应的scrapy的.whl文件
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207153247.png)

![|center](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207153547.png)
经过上面两部我们就将scrapy安装到了本地，但是有没有发现还是无法运行，因为还有**pywin32 和twisted没有安装**故而我们继续到https://pypi.python.org/pypi/pywin32/222下载相应的的.whl文件进行安装
![pywin32](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207153939.png)
![|center](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207154852.png)
同样使用命令行进行安装

    pip install pywin32-222-cp36-cp36m-win_amd64.whl
    pip install Twisted-17.9.0-cp27-cp27m-win_amd64.whl


安装好这些之后我们就可以来看下scrapy了
![|center](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207155127.png)

### scrapy初涉
之前没有接触过的小伙伴可以先看下这个网站的内容https://doc.scrapy.org/en/latest/intro/tutorial.html

我们先通过下面命令行创建一个scrapy项目

    scrapy startproject heloScrapy

相应的文件结构如下图所示

![|center](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207161524.png)

而后我们在spiders文件夹下创建一个demo.py文件，之后的主要代码都将在该文件内完成。
引入scrapy以及相应的request，并命名为demo

    import scrapy
	from tutorial.items import TutorialItem
	from scrapy.http import Request
	
	class DmozSpider(scrapy.Spider):
	    name = "demo"
	    allowed_domains = ["blog.csdn.net", "baidu.com"]
	    start_urls = [
        "http://blog.csdn.net/u013687632/article/details/57075514"
	    ]

此处在start_urls中设置了相应的初始网址http://blog.csdn.net/u013687632/article/details/57075514，也就是我们将从该网址出发来爬取相应网页内容，同时我们对于allowed_domains 的设置使我们只爬取这个域名内的网页。

在设置好以上内容之后，scrapy会将start_urls 网页的内容以response的形式传递给parse函数，下面我们就将对parse函数进行定义

        def parse(self, response):
	        filename = response.url.split("/")[-2]
	        with open(filename, 'wb') as f:
	            f.write(response.body)
	        for sel in response.xpath('//ul/li'):
	            item = TutorialItem()
	            item['title'] = sel.xpath('a/text()').extract()
	            item['link'] = sel.xpath('a/@href').extract()
	            item['desc'] = sel.xpath('text()').extract()
	            yield item

	            if sel.xpath('a/@href').extract() == '':
	                print('empty')
	            else:
	                if len(sel.xpath('a/@href').extract()) > 0:
	                    self.num = self.num + 1

	                    print('helo%s' % (self.num))
	                    yield Request(response.urljoin(sel.xpath('a/@href').extract()[0]), callback=self.parse)

从以上内容可以看出我们的是对网页内人title、link、desc进行了抽取，同时根据link中的内容来进行接下去的网页爬取，其中需要注意的方法有一下几个：

        with open(filename, 'wb') as f:
            f.write(response.body)

with as的语法是对于有_enter_()和_exit_()方法的对象使用的，这样可以减少我们代码的书写，不让像上面的内容我们就要写出如下的形式：

    file = open("/tmp/foo.txt")
	try:
	    data = file.read()
	finally:
	    file.close()

然后就是TutorialItem这个类了，该类我们定义在items.py中

    import scrapy
    
	class TutorialItem(scrapy.Item):
	    # define the fields for your item here like:
	    # name = scrapy.Field()
	    title = scrapy.Field()
	    link = scrapy.Field()
	    desc = scrapy.Field()
	    pass

至于sel.xpath('a/text()')就是用来过滤出我们需要的xml对象了，这个方法是lxml包中的，这个包我们在上一篇文章中已经安装过滤，也就是pip instll lxml，而该方法中的表达式该如何写，这个就要靠自己了，人总是要靠自己的。当然我们也可以看下这篇blog的内容https://www.cnblogs.com/lei0213/p/7506130.html

最后你是不是对yield这个语法很困惑，这个就和生成器相关了，详细内容可以看这个blog http://python.jobbole.com/83610/，简单来将呢，yield就是一个关键词，类似return, 不同之处在于，yield返回的是一个生成器。


最后的最后就是下面这段代码了

     yield Request(response.urljoin(sel.xpath('a/@href').extract()[0]), callback=self.parse)

它发起了对新的url的请求，并将返回的内容传递给parse进行处理，这就实现了新url的爬取效果。

最后我们可以通过cmd输入一些命令来运行程序

    scrapy crawl demo


运行结果如下
![enter image description here](http://op0dvu7tu.bkt.clouddn.com/%E4%BA%91%E4%B9%8B%E5%AE%B6%E5%9B%BE%E7%89%8720180207164936.png)


### 总结
使用scrapy能够为我们提供很大的便利，如待爬取队里的建立，以及url去重等都不需要我们去做了，当然最棒的是它有一个扩展包scrapy-redis，通过使用这个包我们可以实现分布式爬取，到时候我们的爬取速度就能够有指数级的提升了（在有多台硬件设备的情况下），而后通过这大量的数据我们就可以进行一些如数据挖掘、机器学习的工作了，是不是很心动。
