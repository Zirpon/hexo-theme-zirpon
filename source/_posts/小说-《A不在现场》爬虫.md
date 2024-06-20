---
title: 小说 《A不在现场》爬虫
catalog: true
header-img: img/header_img/roman.png
subtitle: The quick brown fox jumps over the lazy dog
date: 2024-05-19 22:33:13
tags:
- 爬虫
- scrapy
catagories:
top: 9999
---

# scrapy 爬虫

## 爬取 小说正文 http://st.kanxshuo.com

保存为markdown 格式 [《A不在现场》前11章](kanxshuo-他不在现场-苏格拉夫顿-27846.txt)

### 源码

- [spider-kanxshuo.py](spider-kanxshuo.py) 
- [mylog.py](./utils/mylog.py)
- 依赖 [scrapy](https://docs.scrapy.org/en/latest/) [中文版](https://scrapy-16.readthedocs.io/zh-cn/)

#### spider-kanxshuo.py

```py
import scrapy
from scrapy.selector import Selector
import json

from utils import mylog
logger = mylog.mylogger(__name__)

outDict = []
bookCode = '27846'
website = 'st.kanxshuo.com/'
class kanxshuoSpider(scrapy.Spider):
    name = 'kanxshuo-'+str(bookCode)
    start_urls = [f'http://{website}/book-{bookCode}-1.html',]

    def parse(self, response):
        global outDict

        urltitle = response.css('div.bm_h::text').get()       
        
       #windows 文件名不能把包含以下字符
        urltitle = urltitle.replace('\\', '').replace('/', '').replace(':', '').replace('*', '')\
        .replace('?', '').replace('"', '').replace('<', '').replace('>', '').replace('|', '').replace('<br>', '')
        
        body = response.css('div.bookContent').get()
        body = body.replace('</div>', '').replace('<br>', '\n')\
            .replace('<div class="bookContent" id="fontzoom">', '')\
            .replace('<div id="a_d_4"><script type="text/javascript" src="/skin/a_728.js"></script>','')\
            .replace('<span id="a_d_1"> <script type="text/javascript" src="/skin/a_336.js"></script> </span>', '')\
            .replace('<span id="a_d_2"> <script type="text/javascript" src="/skin/a_728.js"></script> </span>', '')
        logger.info('当前页: {}'.format(urltitle))
        logger.debug('正文: {}'.format(body))
        outDict.append({
            '当前页': urltitle,
            '正文': body,
        })

        yield {
            '当前页': urltitle,
            '正文': body,
        }

        next_page = None
        for nextpage in response.css('div.bpages').css('a'):
            pagename = nextpage.css('a.pn::text').get()
            #logger.debug('{} '.format(pagename))
            if pagename == '下─页':
                next_page = nextpage.css('a::attr("href")').get()
                logger.debug(next_page)
                break
            #elif pagename != '尾页' and pagename != '首页' and pagename != '上一页' \
            #and int(pagename) and int(pagename) > 5000000:
            #    break
        if next_page is not None:
            yield response.follow(next_page, self.parse)
        else:
            outjsonName = "./txt-jsons/kanxshuo-%s-%s.json" % (urltitle, bookCode)
            #logger.debug("+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++")
            logger.debug(outDict)
            outMDName = "./txt-markdown/kanxshuo-%s-%s.md" % (urltitle, bookCode)
            with open(outMDName, 'w+', newline='', encoding='utf-8') as ff:
                for item in outDict:
                    title =  item['当前页']
                    body = item['正文']
                    md_item = f'##### {title}' + '\n'
                    md_item += f'{body}' + '\n'
                    md_item += '\n---\n\n'
                    logger.info(md_item)
                    ff.writelines(md_item)
            #with open(outjsonName, "w+", encoding='utf-8') as f:
            #    # json.dump(dict_, f)  # 写为一行
            #    json.dump(outDict, f, indent=2, sort_keys=False, ensure_ascii=False)  # 写为多行
```

#### ./utils/mylog.py

```py
import logging, coloredlogs

def mylogger(name):
    logger = logging.getLogger(name)

    level_styles = coloredlogs.DEFAULT_LEVEL_STYLES.copy()
    level_styles['debug'] = {'color': 'magenta'}
    level_styles['info'] = {'color': 'yellow'}
    level_styles['error'] = {'color': 'red'}
    level_styles['warning'] = {'color': 'blue'}
    coloredlogs.install(
        level="DEBUG",  # show only debug and above
        #fmt="%(asctime)s - %(hostname)s - %(name)s[%(process)d] -\
        #  %(pathname)s -%(filename)s - %(funcName)s - %(lineno)d - %(module)s - %(levelname)s - %(message)s",
        fmt="%(asctime)s - %(hostname)s - %(name)s[%(process)d] - %(filename)s::%(funcName)s::%(lineno)d - %(levelname)s - %(message)s",
 
        logger=logger,
        level_styles=level_styles,
    )
    return logger

    """
        logger.debug('Print log level：debug')
        logger.info('Print log level：info')
        logger.warning('Print log level：warning')
        logger.error('Print log level：error')
        logger.critical('Print log level：critical')
    """
```

### python 执行脚本

```sh
scrapy runspider spider-kanxshuo.py -o spider-kanxshuo.json -s FEED_EXPORT_ENCODING=UTF-8 -s LOG_FILE=spider-kanxshuo.log
```

## 下载B站 有声小说视频

使用剪映 识别视频字幕 保存为小说正文

### 使用vim 修改 视频字幕小说正文

#### vim 每 5行 合并为一行

```
nomarl 模式

输入 qa 进入录制模式

按5次 J 合并 5行

按j 跳下一行 

输入 q 退出录制模式

输入 3000@a 表示 重复执行3000次录制操作
```

#### vim 匹配 #### 数字数字. 并前后插入换行符

[& 表示使用匹配串](https://zhuanlan.zhihu.com/p/346058975)

> :%s/#### \d\d. /\r\r&\r\r\/
>

#### vim 每10行插入 3行空行

```
normal 模式下 yy辅助一行空行

输入 qa 进入录制模式

按10次 j 跳过10行

按3次 P 插入拷贝空行

输入q 退出录制模式

输入 150@a 表示重复执行 150次录制操作
```
[《不在现场》11章到最后一章](《不在现场》11章到最后一章.txt)

### 使用 bash脚本 分割大文本文件
[linux的history命令只显示最新10条](https://blog.csdn.net/u013541707/article/details/107359727)
```bash
  wc -l all.log
  split -l 35000 -d --verbose all.log all-log-split-
  for i in `ls | grep all-log-split-`;do a=`echo $i.txt`; mv $i $a;done
```

## 爬取 B站 《庆余年》 有声小说视频

[《庆余年》小说正文](《庆余年》_qinkan.net.txt)