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

# scrape 爬虫

## 爬取 小说正文 http://st.kanxshuo.com

保存为markdown 格式 [A不在现场](kanxshuo-他不在现场-苏格拉夫顿-27846.txt)


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

[视频字幕版](《不在现场》11章到最后一章.txt)