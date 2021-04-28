
目录
=================

  * [基金分析项目](#基金分析项目)
    * [概述](#概述)
    * [1.基金数据爬取](#1-基金数据爬取)
    * [2.定投分析](#2-定投分析)
    * [3.定投利润计算](#3-定投利润计算)
  * [开发日志](#开发日志)

# 基金分析项目

## 概述
这是一个自己使用的分析基金的项目，目前主要实现了目前只有一个需求，就是可以分析一个基金的定投情况

懒得架数据库，都用文件存储了

此repo有一个关联项目:[基金分析](https://github.com/users/piginzoo/projects/1)，这个项目是github项目，里面提供了需求看板，方便需求更新。

另外，还创建了一个gitbook：[我的金融工具箱](https://finbook.piginzoo.com/)，用来编写我对金融知识的积累。

## 1 基金数据爬取

###实现思路

这个功能，实现了单独爬取一个基金的数据，或者批量爬取所有的基金数据。

### 如何运行

运行爬虫，只需要运行run.sh即可：

```
    bin/run.sh
```
运行这个命令，即可运行爬虫，目前有一下运行选项：
- bin/run.sh all 爬取所有的基金，支持增量爬取，即可以继续爬取自上次爬取后最新的数据
- bin/run.sh --code xxxxx 指定爬取一只基金，xxxxx为基金代码


### 详细实现思路

先实现单个基金的爬取，使用request包，拼出来爬取的url，
然后对返回的结果进行解析，形成pandas的DateFrame，
最终利用dataframe的to_csv，保存到 data/db文件夹，
数据文件的命名方式为 <code>.csv。

【细节问题】
- 如何保存数据？
  没有用数据库，而是采用了最直白的文本csv文件，用dataframe.to_csv，形成csv文件。
- 如何保证爬取后，不再爬取？
  每次爬取，都从dataframe的数据文件中，找到最后的一次爬取日期，然后从当日到这个最后一次爬取日期之间的数据，都进行爬取。
- 如何实现增量爬取
  上面已经解释过，会只爬取最后的日期到今天的数据了，所以增量爬取应该很快
- 爬取参数有哪些？
  爬取参数，包含了每页爬几条（目前是40条），爬取第几页，日期是从什么时候到什么时候   
- 日期排序
  爬取完一页是不保存的，而是等都爬取完，才保存，保存之前，会按照日期排序，日期是index，方便检索和排序
- 爬取频率
  为了防止被封，目前的爬取频次是0~1秒之间的间歇，而且是一个进程
  
这个项目，主要参考了：<https://github.com/weibycn/fund>

使用到的web数据接口是[天天基金](https://fund.eastmoney.com/)的api：
```
http://fund.eastmoney.com/f10/F10DataApi.aspx?type=lsjz&code=377240&page=1&per=20&sdate=2017-03-01&edate=2017-03-01
```

### 爬取结果

按照0~1的间隔持续爬取了40个小时，终于爬下来所有的数据，大概是11000个基金，300M的大小，压缩后是55M，由于太大，我没有放到git中，
而是放到百度云盘上，可以通过

数据: 请解压缩数据到 <data/db> 文件夹中，
链接: <https://pan.baidu.com/s/134xIJdd2vVmnP7msa7ujcw> 提取码: **3e6d** 


## 2 定投分析

可以跑一下一个基金，在某个时期内，按照某种频次定投的收益率是多少。

如何运行：

```
bin/invest.sh --code <基金代码> --start <定投开始日期> --end <定投结束日期> --period <day|week|month> --day <第几日>"
```

需要输入一下参数：
 - code     基金代码
 - start    定投开始日期 
 - end      定投结束日期
 - period <day|week|month>，分别代表是每天都定投、每周都定投、每个月都定投 
 - day      第几日，意思是每周第几天、每个月第几天，当period为day时候此参数无效 

样例：

```
bin/invest.sh --code 519778 --start 2020-01-01 --end 2021-04-22 --period week --day 1

INFO : 加载了[data/db/519778.csv]数据，行数：748
DEBUG : 数据经过[week]过滤,剩余64条
DEBUG : 最后一天[2021-04-20]的价格为：3.20
INFO : 代码[519778] 按[周]定投 64 次, [2020-01-01] -> [2021-04-22] 定投收益率: 121.328%, 耗时: 0.09
```

## 3 定投利润计算 

用来计算真实的投机计划的利润情况。

### 编写定投文件

你需要编撰一个投资计划文件，文本格式的，格式如下：
`<date>,<amount>`

其中，date是yyyy-mm-dd格式的，必须是这样哈，别的不支持。
如果是定投的话，可以用一下格式表示：
- month_xx，每月第xx日定投，如果遭遇周末或节假日，向后顺延
- week_xx，每周第xx日定投，如果遭遇周末或节假日，向后顺延

下面是一个例子：

```
2020-1-7,5000
2020-1-13,500
2020-2-11,500
2020-2-24,-3231.14
2020-3-2,3300
month_11,1000
```

### 运行利润计算

```
bin/invest.sh

bin/invest.sh --code <基金代码> --start <定投开始日期> --end <定投结束日期> --period <day|week|month> --day <第几日>"
```

需要输入一下参数：
 - code     基金代码
 - start    定投开始日期 
 - end      定投结束日期
 - period <day|week|month>，分别代表是每天都定投、每周都定投、每个月都定投 
 - day      第几日，意思是每周第几天、每个月第几天，当period为day时候此参数无效 

样例：

```
bin/profit.sh --code 001938 --plan data/plan/jq_001938.txt

INFO :
基金代码:	 001938 ,
投资计划获利:	22.6489% ，
总投资金额:	20068.86 ,
总账面资产价值:	24614.23 ，
合计获利金额:	4545.37 ，
合计投资天数:	19 天
```


# 开发日志

## 2021.4.28
实现了真实投资计划利润计算

## 2021.4.25
修复了月度的时候定投日是节假日，无法顺延的bug

## 2021.4.23
实现了一个定投的利润率计算，可以跑一个基金从某日到某日的投资收益率。

## 2021.4.21
实现了所有的基金的爬取，基金的列表是取自：<http://fund.eastmoney.com/js/fundcode_search.js>

格式：

```
["000001","HXCZ","华夏成长","混合型","HUAXIACHENGZHANG"]
```
然后不再每次爬取都保存，而是整体一个基金爬取完，才保存。
另外，支持断点续传，如果爬取过的，就不在爬取。
具体实现是，获取爬取的最后的日期，做起起点，一直爬取到昨天（今天的数据没有）
未来可以用这个机制，实现自动的全体的增量爬取。

解决了排序问题，原来是set_index(inplace=True)，要替掉默认的索引。

## 2021.4.20
实现了一个基金的爬取，可以爬取成功了，
也实现了从开始日到节结束日的爬取，
他们的api支持sdate~edate，就是支持日期段爬取的。
并且从中解析出剩余页数，用于循环用。
