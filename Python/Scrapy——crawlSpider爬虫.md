# Scrapy——crawlSpider爬虫

Scrapy中的BaseSpider爬虫类只能抓取start_urls中提供的链接，而利用Scrapy提供的crawlSpider类可以很方便的自动解析网页上符合要求的链接，从而达到爬虫自动抓取的功能。
 
要利用crawSpider和BaseSpider的区别在于crawSpider提供了一组Rule对象列表，这些Rule对象规定了爬虫抓取链接的行为，Rule规定的链接才会被抓取，交给相应的callback函数去处理。
  
在rules中通过SmglLinkExtractor提取希望获取的链接。
 
我此次的demo中rule只有一个，如下：

``` python
allowed_domains = ['sj.zol.com.cn']
start_urls = ['http://sj.zol.com.cn/bizhi/']
number = 0
rules = (
            Rule(SgmlLinkExtractor(allow = ('detail_\d{4}_\d{5}\.html')),callback = 'parse_image',follow=True),
            )
```

搜索起始链接下面符合allow中正则表达式的链接，并跟进解析，如果follow = False，则只会解析起始链接中找到的符合要求的链接
 
SmglLinkExtractor主要参数：
 
- allow：满足括号中“正则表达式”的值会被提取，如果为空，则全部匹配。
- deny：与这个正则表达式(或正则表达式列表)不匹配的URL一定不提取。
- allow_domains：会被提取的链接的domains。
- deny_domains：一定不会被提取链接的domains。
- restrict_xpaths：使用xpath表达式，和allow共同作用过滤链接。
 
下面的内容就是利用Selector解析获得的response并赋值个item就行了：

``` python
#coding: utf-8 #############################################################
# File Name: spiders/wallpaper.py
# Author: mylonly
# mail: tianxianggen@gmail.com
#Blog:www.mylonly.com
# Created Time: 2014年09月01日 星期一 14时20分07秒
#########################################################################
#!/usr/bin/python
import urllib2
import os
import re
 
from scrapy.contrib.spiders import CrawlSpider,Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import Selector
from mylonly.items import wallpaperItem
from scrapy.http import Request
 
class wallpaper(CrawlSpider):
        name = "wallpaperSpider"
        allowed_domains = ['sj.zol.com.cn']
        start_urls = ['http://sj.zol.com.cn/bizhi/']
        number = 0
        rules = (
                        Rule(SgmlLinkExtractor(allow = ('detail_\d{4}_\d{5}\.html')),callback = 'parse_image',follow=True),
                        )
        def parse_image(self,response):
                self.log('hi,this is an item page! %s' % response.url)
                sel = Selector(response)
                sites = sel.xpath("//div[@class='wrapper mt15']//dd[@id='tagfbl']//a[@target='_blank']/@href").extract()       
                for site in sites:
                        url = 'http://sj.zol.com.cn%s' % (site)
                        print 'one page:',url
                        item = wallpaperItem()
                        item['size'] = re.search('\d*x\d*',site).group()
                        item['altInfo'] = sel.xpath("//h1//a/text()").extract()[0]
                        return Request(url,meta = {'item':item},callback = self.parse_href)
        def parse_href(self,response):
                print 'I am in:',response.url
                item = response.meta['item']
                items = []
                sel = Selector(response)
                src = sel.xpath("//body//img/@src").extract()[0]
                item['imgSrc'] = src
                items.append(item)
                return items
                #self.download(src)
        def download(self,url):
                self.number += 1
                savePath = '/mnt/python_image/%d.jpg' % (self.number)
                print '正在下载...',url
                try:
                        u = urllib2.urlopen(url)
                        r = u.read()
                        downloadFile = open(savePath,'wb')
                        downloadFile.write(r)
                        u.close()
                        downloadFile.close()
                except:
                        print savePath,'can not download.'
```
 
 
 
接下来看看我的Item.py的代码：

```python
class wallpaperItem(scrapy.Item):
        size = scrapy.Field()
        altInfo = scrapy.Field()
        imgSrc = scrapy.Field()
```
 
还有pipilines.py:

```python
# -*- coding: utf-8 -*-
 
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html
import MySQLdb
 
 
class MylonlyPipeline(object):
    def process_item(self, item, spider):
        return item
 
class wallpaperPipeline(object):
        def process_item(self,item,spider):
                print 'imgSrc:',item['imgSrc']
                db = MySQLdb.connect("rdsauvva2auvva2.mysql.rds.aliyuncs.com","mylonly","703003659txg","wallpaper")
                cursor = db.cursor()
                db.set_character_set('utf8')
                cursor.execute('SET NAMES utf8;')
                cursor.execute('SET CHARACTER SET utf8;')
                cursor.execute('SET character_set_connection=utf8;')
 
                sql ="INSERT INTO mobile_download(wallpaper_size,wallpaper_info,wallpaper_src)\
                      VALUES('%s','%s','%s')"%(item['size'],item['altInfo'],item['imgSrc'])
                try:
                        print sql
                        cursor.execute(sql)
                        db.commit()
                except MySQLdb.Error,e:
                        print "Mysql Error %d: %s" % (e.args[0], e.args[1])
                db.close()
                return item
```
 
 
 
本例中我将传回的items数据存放到了数据库中，如果你不想这样，可以将我注释掉的self.download()取消注释，不反悔items，就可以将找到的所有图片链接全部下载下来，不过要找个大点的地方存储，因为总共有5W多条：
![2016-06-29_061712481319744.png](http://pic.mylonly.com/2016-06-29_061712481319744.png)

