# Scrapy模拟登陆知乎
![2016-06-29_14645998164473.jpg](http://pic.mylonly.com/2016-06-29_14645998164473.jpg)
感谢简书作者`Andrew_liu`提供的思路,虽然知乎改版后，该文章上提供的方法已经失效[Python爬虫(七)--Scrapy模拟登录
](http://www.jianshu.com/p/b7f41df6202d)

利用Scrapy提供的cookie中间价很容易做到网页的模拟登陆,下面就来介绍怎么利用这个cookie中间件来登陆知乎。

###前期分析工作
* 打开https://www.zhihu.com,利用Chrome浏览器的Debug功能，定位到登陆框所在的位置，如下图所示
![2016-06-29_14645980407304.jpg](http://pic.mylonly.com/2016-06-29_14645980407304.jpg)
利用scrapy提供的xpath能很方便的获取到这个值(//div[@data-za-module="SignInForm"]//form//input[@name="_xsrf"]/@value')

* 选中Debug窗口的Network选项，同时在输完账号密码后点击登陆，获取登陆操作后的post请求，见下图:

![2016-06-29_14645982403637.jpg](http://pic.mylonly.com/2016-06-29_14645982403637.jpg)
点击这个链接，确认这个链接就是提交登陆的url
![2016-06-29_14645983258011.jpg](http://pic.mylonly.com/2016-06-29_14645983258011.jpg)
从FormData里面可以看到https://www.zhihu.com/email/login就是登陆POST请求的url，需要提交4个参数，其中_xsrf就是首页可以获取到的隐藏表单参数，remember_me是是否记住cookie的开关，email和password对应账号和密码
> 此处可能有不一样的地方，因为我的知乎账号是email注册的，根据这个url的特征推测别的账号类型可能存在不一样的Url

### 编写蜘蛛代码
1.继承CrawlSpider,并重写spider的start_request方法，让spier先访问登录页再去爬取start_urls中的链接，在start_requests方法中，让spider先去访问知乎首页，去获取隐藏的表单项`_xsrf`

```Python
def start_requests(self):
        return [Request("https://www.zhihu.com/",headers = self.headers,meta={"cookiejar":1},callback=self.post_login)]
```
其中header需要自定义，因为知乎对spider做了限制，应该是检测User-Agent，你可以在setting.py中更改spider的默认UserAgent,也可以像我这样自己自定义一个

```Pythoh
headers = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Accept-Encoding": "gzip,deflate",
    "Accept-Language": "en-US,en;q=0.8,zh-TW;q=0.6,zh;q=0.4",
    "Connection": "keep-alive",
    "Content-Type":" application/x-www-form-urlencoded; charset=UTF-8",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.97 Safari/537.36",
    "Referer": "http://www.zhihu.com"
}
```
meta中的cookiejar是Scrapy的Cookie中间件的关键字，具体可参考scrapy文档，这里因为只需要保存一个cookie，所以直接写1(注意:`并不是1个cookie才写的1，仅仅是个key，后面通过这个1这个key找到cookiejar中保存的cookie`)

2.解析首页内容，获取到_xsrf的值，同时提交登录请求:

```Python
def post_login(self,response):
        self.log("preparing login...")
        xsrf = Selector(response).xpath('//div[@data-za-module="SignInForm"]//form//input[@name="_xsrf"]/@value').extract()[0]
        self.log(xsrf)
        return FormRequest("https://www.zhihu.com/login/email",meta={'cookiejar':1},
                                          headers = self.headers,
                                          formdata = {
                                             '_xsrf':xsrf,
                                             'password':'xgBKQTx7VnVLK9tv',
                                             'email':'tianxianggen@gmail.com',
                                             'remember_me':'true',
                                          },
                                          callback = self.after_login,
                                          )
```

3. 将登录成功后获取到的cookie传递给每一个start_urls中链接的ruquest

```Python
def after_login(self,response):
        for url in self.start_urls:
            yield Request(url,meta={'cookiejar':1},headers = self.headers)
```

4.由于cookiejar中的cookie并不会自动发送给每个链接，因此在urls通过Rule获取到的连接，也是需要我们手动将cookie加上，通过Rule提供的process_request参数重新创建带cookie的Request

```Pythoh
rules = (
        Rule(SgmlLinkExtractor(allow=('/question/\d*')),process_request="request_question"),
    )
```
同时提供request_question函数

```Python
def request_question(self,request):
        return Request(request.url,meta={'cookiejar':1},headers = self.headers,callback=self.parse_question)
```

5.由于已经有了process_link ,Rule中的callback参数就不再起作用了，而是调用新构造的Request中的callback函数。

```Python
def parse_question(self,response):
        sel = Selector(response)
        item = zhihuItem()
        item['qestionTitle'] = sel.xpath("//div[@id='zh-question-title']//h2/text()").extract_first()
        item['image_urls'] = sel.xpath("//img[@class='origin_image zh-lightbox-thumb lazy']/@data-original").extract()
        return item
```
> 这个parse_question方法仅仅是获取问题名称和问题下面的所有图片链接。

### 完整代码

```Python
import urllib2
import os
import re
import codecs


from scrapy.contrib.spiders import CrawlSpider,Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import Selector
from MySpider.items import zhihuItem
from scrapy.http import Request
from scrapy.http import FormRequest
from scrapy.utils.response import open_in_browser


class zhihuSpider(CrawlSpider):
    name = "zhihu"
    allow_domians = ["zhihu.com"]
    start_urls = ["https://www.zhihu.com/collection/38624707"]
    rules = (
        Rule(SgmlLinkExtractor(allow=('/question/\d*')),process_request="request_question"),
    )
    
    headers = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Accept-Encoding": "gzip,deflate",
    "Accept-Language": "en-US,en;q=0.8,zh-TW;q=0.6,zh;q=0.4",
    "Connection": "keep-alive",
    "Content-Type":" application/x-www-form-urlencoded; charset=UTF-8",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.97 Safari/537.36",
    "Referer": "http://www.zhihu.com"
    }
    
    def start_requests(self):
        return [Request("https://www.zhihu.com/",headers = self.headers,meta={"cookiejar":1},callback=self.post_login)]
        
    def post_login(self,response):
        self.log("preparing login...")
        xsrf = Selector(response).xpath('//div[@data-za-module="SignInForm"]//form//input[@name="_xsrf"]/@value').extract()[0]
        self.log(xsrf)
        return FormRequest("https://www.zhihu.com/login/email",meta={'cookiejar':response.meta['cookiejar']},
                                          headers = self.headers,
                                          formdata = {
                                             '_xsrf':xsrf,
                                             'password':'差点就忘了删了',
                                             'email':'邮箱也不能暴露',
                                             'remember_me':'true',
                                          },
                                          callback = self.after_login,
                                          )
                                                      
    def after_login(self,response):
        for url in self.start_urls:
            yield Request(url,meta={'cookiejar':1},headers = self.headers)
   
    def request_question(self,request):
        return Request(request.url,meta={'cookiejar':1},headers = self.headers,callback=self.parse_question)
        
    def parse_question(self,response):
        sel = Selector(response)
        item = zhihuItem()
        item['qestionTitle'] = sel.xpath("//div[@id='zh-question-title']//h2/text()").extract_first()
        item['image_urls'] = sel.xpath("//img[@class='origin_image zh-lightbox-thumb lazy']/@data-original").extract()
        return item
```


