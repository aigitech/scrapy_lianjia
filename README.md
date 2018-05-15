# lianjia_scrapy
基于Scrapy+Mysql的链家二手房爬虫
**对济南链家二手房以行政区和小区名称进行房源的爬取**

    class Lianjia_spider(Spider):
    name = 'lianjia'
    allowed_domains = ['jn.lianjia.com']
    regions = {'lixia':'历下',
               'shizhong':'市中',
               'tianqiao':'天桥',
               'licheng': '历城',
               'huaiyin': '槐荫'
    }

**小区设置**

    def start_requests(self):
        for region in list(self.regions.keys()):
            url = "https://jn.lianjia.com/xiaoqu/" + region + "/"
            yield Request(url=url, callback=self.parse, meta={'region':region}) #用来获取页码

    def parse(self, response):
        region = response.meta['region']
        selector = etree.HTML(response.text)
        sel = selector.xpath("//div[@class='page-box house-lst-page-box']/@page-data")[0]
        sel = json.loads(sel)  # 转化为字典
        total_pages = sel.get("totalPage")

        for i in range(int(total_pages)):
            url_page = "https://jn.lianjia.com/xiaoqu/{}/pg{}/".format(region, str(i + 1))
            yield Request(url=url_page, callback=self.parse_xiaoqu, meta={'region':region})


**获取数据结构**

    class LianjiaItem(Item):
     table = 'lianjia'
    # define the fields for your item here like:
    # name = scrapy.Field()
     region = Field()      #行政区域
     href = Field()        #房源链接
     house_name = Field()        #房源名称
     style = Field()       #房源结构
     area = Field()           #小区
     orientation = Field()    #朝向
     decoration = Field()     #装修
     elevator = Field()       #电梯
     floor = Field()          #楼层高度
     build_year = Field()     #建造时间
     sign_time = Field()      #签约时间
     unit_price = Field()     #每平米单价
     total_price = Field()    #总价
     fangchan_class = Field()   #房产类型
     school = Field()         #周边学校
     subway = Field()         #周边地铁

**存储到mysql**
