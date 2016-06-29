# 简单Scrapy爬虫

只是个小小的demo，自己测试了下，总共down了20几张图片

```Python
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
 
from scrapy.contrib.spiders import CrawlSpider,Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import Selector
from mylonly.items import wallpaperItem
from scrapy.http import Request
 
class wallpaper(CrawlSpider):
    name = "wallpaperSpider"
    allowed_domains = ['sj.zol.com.cn']
    start_urls = ['http://sj.zol.com.cn/']
    number = 0
    rules = (
    Rule(SgmlLinkExtractor(allow = ('detail_\d{4}_\d{5}\.html')),callback =     'parse_image',follow=True),)
    def parse_image(self,response):
        self.log('hi,this is an item page! %s' % response.url)
        sel = Selector(response)
        sites = sel.xpath("//div[@class='wrapper        mt15']//dd[@id='tagfbl']//a[@target='_blank']/@href").extract()
        for site in sites:
        url = 'http://sj.zol.com.cn%s' % (site)
        print 'one page:',url
        return Request(url,callback = self.parse_href)
    def parse_href(self,response):
        print 'I am in:',response.url
        sel = Selector(response)
        src = sel.xpath("//body//img/@src").extract()[0]
        self.download(src)
 
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


