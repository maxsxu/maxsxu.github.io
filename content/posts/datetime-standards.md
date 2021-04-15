+++ 
draft = false
date = 2021-02-03T15:52:46+08:00
title = "日期时间规范"
description = ""
slug = "" 
tags = ["Datetime"]
categories = ["Develop"]
externalLink = ""
series = []
+++

日期时间格式规范的目标是作为统一的标准，来规范和统一不同时区、不同语言、不同国家对同一个时刻的表示。

# 概念
- 时区

时区（Time Zone) 是地球上的区域使用同一个时间定义。1884 年在华盛顿召开国际经度会议时，为了克服时间上的混乱，规定将全球划分为 24 个时区。 在中国采用首都北京所在地东八区的时间为全国统一使用时间。

- 时差

世界各国位于地球不同位置上，因此不同国家，特别是东西跨度大的国家日出、日落时间必有偏差。这些偏差就是时差。

- 夏令时

夏令时（Daylight Saving Time, DST），又称 “日光节约时制” 和 “夏令时间”，是一种为节约能源而人为规定地方时间的制度。一般在天亮早的夏季人为将时间调快一小时，可以使人早起早睡，减少照明量，以充分利用光照资源，从而节约照明用电。
1986 年到 1991 年期间，中国曾实行夏令时制度。

- 冬令时

冬令时通常是指当地使用的标准时间。在使用夏令时的地区，夏天时钟拨快一小时，冬天再拨回标准时间。

- 时间戳

时间戳是一个数字，定义为格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒 (北京时间 1970 年 01 月 01 日 08 时 00 分 00 秒) 起至现在的总秒数。注意，同一时刻不同时区获得的时间戳是相同的。

- 本地时间

本地时间只包括当前的时间，不包含任何时区信息。同一时刻，东八区的本地时间比零时区的本地时间快了 8 个小时。

- 标准时间

在不同时区之间交换时间数据，除了用纯数字的时间戳，还有一种更方便人类阅读的表示方式：标准时间的偏移量表示方法。

- GMT

GMT（Greenwich Mean Time，格林威治平均时间）是时区时间。指位于英国伦敦郊区的皇家格林尼治天文台当地的平太阳时，因为本初子午线被定义为通过那里的经线。
GMT 已被 UTC 取代。

- UTC

UTC（Coordinated Universal Time, 协调世界时）是标准时间。是目前通用的世界时间标准, UTC 作为一个标准时间和各个时区时间换算。

```
GMT: UTC +0    =    GMT: GMT +0
CST: UTC +8    =    CST: GMT +8
PST: UTC -8    =    PST: GMT -8
```

- PST

PST (Pacific Standard Time, 太平洋时间)

- CST

CST (China Standard Time, 北京时间)

# 标准
## ISO 8601
> 国际标准。

|           Name           |                         Description                          |
| :----------------------: | :----------------------------------------------------------: |
|      ISO 8601:1988       | Data elements and interchange formats — Information interchange — Representation of dates and times |
| ISO 8601:1988/COR 1:1991 | Data elements and interchange formats — Information interchange — Representation of dates and times — Technical Corrigendum 1 |
|      ISO 8601:2000       | Data elements and interchange formats — Information interchange — Representation of dates and times |
|      ISO 8601:2004       | Data elements and interchange formats — Information interchange — Representation of dates and times |
|     ISO 8601-1:2019      | Date and time — Representations for information interchange — Part 1: Basic rules |
|     ISO 8601-2:2019      | Date and time — Representations for information interchange — Part 2: Extensions |



## RFC 3339
> 国际标准。

RFC3339 详细定义了互联网上日期 / 时间的偏移量表示：

```
2017-12-08T00:00:00.00Z
```

这个代表了 UTC 时间的 2017 年 12 月 08 日零时

```
2017-12-08T00:08:00.00+08:00
```

这个代表了同一时刻的，东八区北京时间（CST）表示的方法

上面两个时间的时间戳是等价的。两个的区别，就是在本地时间后面增加了时区信息。Z 表示零时区。+08:00 表示 UTC 时间增加 8 小时。

这种表示方式容易让人疑惑的点是从标准时间换算 UTC 时间。以 CST 转换 UTC 为例，没有看文档的情况下，根据 +08:00 的结尾，很容易根据直觉在本地时间再加上 8 小时。正确的计算方法是本地时间减去多增加的 8 小时。+08:00 减去 8 小时才是 UTC 时间，-08:00 加上 8 小时才是 UTC 时间。



## GB/T 7408-2005
> 国家标准。

国家标准《数据元和交换格式 信息交换 日期和时间表示法》 由 TC83（全国电子业务标准化技术委员会）归口上报及执行 ，主管部门为 国家标准化管理委员会。

本标准等同采用 ISO 国际标准：ISO 8601:2000。

# 参考
- ISO 8601, https://www.iso.org/iso-8601-date-and-time-format.html
- RFC 3339, https://www.ietf.org/rfc/rfc3339.txt
- GB/T 7408-2005, http://std.samr.gov.cn/gb/search/gbDetailed?id=71F772D77FD1D3A7E05397BE0A0AB82A
- Time Zone Abbreviations, https://www.timeanddate.com/time/zones/
- Wikipedia ISO 8601, https://en.wikipedia.org/wiki/ISO_8601