# chinese-calendar-golang

[![Run Tests](https://github.com/Lofanmi/chinese-calendar-golang/actions/workflows/runtest.yml/badge.svg?branch=master)](https://github.com/Lofanmi/chinese-calendar-golang/actions?query=branch%3Amaster)
[![codecov](https://codecov.io/gh/Lofanmi/chinese-calendar-golang/branch/master/graph/badge.svg)](https://codecov.io/gh/Lofanmi/chinese-calendar-golang)
[![Go Report Card](https://goreportcard.com/badge/github.com/Lofanmi/chinese-calendar-golang)](https://goreportcard.com/report/github.com/Lofanmi/chinese-calendar-golang)
[![Go Reference](https://pkg.go.dev/badge/github.com/Lofanmi/chinese-calendar-golang?status.svg)](https://pkg.go.dev/github.com/Lofanmi/chinese-calendar-golang?tab=doc)
[![Sourcegraph](https://sourcegraph.com/github.com/Lofanmi/chinese-calendar-golang/-/badge.svg)](https://sourcegraph.com/github.com/Lofanmi/chinese-calendar-golang?badge)
[![Open Source Helpers](https://www.codetriage.com/lofanmi/chinese-calendar-golang/badges/users.svg)](https://www.codetriage.com/Lofanmi/chinese-calendar-golang)

公历, 农历, 干支历转换包, 提供精确的日历转换.

使用 `Go` 编写, 觉得好用的给 `Star` 呗~

觉得**好用**, 而不是觉得**有用**. 如果不好用, 欢迎向我提 issue, 我会抽空不断改进它!

真希望有人打钱打钱打钱给我啊哈哈哈哈!!!

# 如何安装

新版本已支持 Go Module, 可以直接引入。

```bash
go mod tidy
go mod download -x
```

如果您还在使用 GOPATH, 别忘了 `go get`:
```bash
go get -u -v github.com/Lofanmi/chinese-calendar-golang/calendar@latest
```

# 用法

```bash
# 确保使用的是东八区(北京时间)
export TZ=PRC

# 查看时间
date -R
```

```go
package main

import (
	"time"

	"github.com/Lofanmi/chinese-calendar-golang/calendar"
)

func main() {
    t := time.Now()
    // 1. ByTimestamp
    // 时间戳
    c := calendar.ByTimestamp(t.Unix())
    // 2. BySolar
    // 公历
    c := calendar.BySolar(year, month, day, hour, minute, second)
    // 3. ByLunar
    // 农历(最后一个参数表示是否闰月)
    c := calendar.ByLunar(year, month, day, hour, minute, second, false)

    bytes, err := c.ToJSON()

    fmt.Println(string(bytes))
}
```

# 原理

1. 公历的计算比较容易, 生肖, 星座等的转换见源码, 不啰嗦哈;

2. 农历的数据和算法参考了[chinese-calendar](https://github.com/overtrue/chinese-calendar)的实现, 还有 [1900年至2100年公历、农历互转Js代码 - 晶晶的博客](http://blog.jjonline.cn/userInterFace/173.html), 节气的舍去了, 因为在干支历和生肖的计算上不够精确;

3. 干支历的转换是最繁琐的, 其计算依据是二十四节气. ~~我结合了网上的文章, 用天文方法计算了 1904-2024 的二十四节气时间.~~ 目前使用的是寿星天文历的时间, 支持 1904-3000 年, 精确到秒, 并与[香港天文台](http://data.weather.gov.hk/gts/astron2018/Solar_Term_2018_uc.htm)进行比较, 误差基本在 `1秒` 以内, 大家如果有以往的数据, 可以看下源码比对一下.

# 使用须知

1. 干支历的时间范围是 1904-3000 年, ~~主要也是因为 1904 年之前的时间戳, 在32位的 `PHP` 下会识别不出(溢出), 而且再早的时间, 其实意义也不大,~~因为之前遗留的问题开放了 `NewSolarterm` 接口, 为保证兼容性, `index` 不能随便改, ~~2024之后...好像也不会用到吧. 研究二十四节气这个算法的时间花了很长很长, 反而撸代码的时间不算太多~~;

2. ~~实际上, 农历的时间也是可以根据天文方法计算出来的. 计算的过程, 也需要先计算二十四节气和每月的正月初一的日期(日月合朔), 才能知道闰月的信息(所以农历是阴阳历). 不过已经有了数据, 我也就懒了, 直接拿来用...所以农历的算法我还没有实现~~(我在2025年实现了!时隔7年!);

3. 农历的时间范围是 1900-3000 年, 数据来自寿星万年历算法. [晶晶的博客](https://github.com/jjonline/calendar.js) 注意只支持到 2100 年, 并且 2057 年存在错误(2057-09-28、2057-10-27), 后续我给个 PR 到那边仓库, 后面全网大家都可以拿到最准确的数据了~

# 公历(阳历) - 字段说明

```
{
    // 生肖, 以立春分界
    //     如 2018-02-04 05:28:26 立春
    //        2018-02-04 04:59:59 属鸡年
    //        2018-02-04 05:00:00 属狗年
    "animal": "鼠",

    // 星座
    "constellation": "天秤",

    // 今年是否为闰年
    "is_leep": true,

    // 年
    "year": 2020,
    // 月
    "month": 9,
    // 日
    "day": 20,

    // 时
    "hour": 5,
    // 分
    "minute": 15,
    // 秒
    "second": 26,
    // 纳秒
    "nanosecond": 0,

    // 星期日
    "week_alias": "日",
    // 星期序数, 0表示周日
    "week_number": 0,
}
```

# 农历(阴阳历) - 字段说明

```
{
    // 生肖, 以每年正月初一分界
    "animal": "鼠",

    // 年
    "year": 2020,
    // 年(汉字)
    "year_alias": "二零二零",
    // 月
    "month": 8,
    // 月(汉字)
    "month_alias": "八月",
    // 日
    "day": 4,
    // 日(汉字)
    "day_alias": "初四",

    // 是否闰年
    "is_leap": true,
    
    // 这个月是否为闰月
    "is_leap_month": false,

    // 今年闰四月
    "leap_month": 4,
}
```

# 干支历 - 字段说明

```
{
    // 生肖, 以立春分界
    //     如 2018-02-04 05:28:26 立春
    //        2018-02-03 05:28:25 属鸡年
    //        2018-02-04 05:28:26 属狗年
    "animal": "鼠",

    // 年干支
    "year": "庚子",
    // 年干支六十甲子序数
    "year_order": 37,

    // 月干支
    "month": "乙酉",
    // 月干支六十甲子序数
    "month_order": 22,

    // 日干支
    "day": "丙寅",
    // 日干支六十甲子序数
    "day_order": 3,

    // 时干支
    "hour": "辛卯",
    // 时干支六十甲子序数
    "hour_order": 28,
}
```

# 输出示例

```json
{
    "ganzhi": {
        "animal": "鼠",
        "day": "丙寅",
        "day_order": 3,
        "hour": "辛卯",
        "hour_order": 28,
        "month": "乙酉",
        "month_order": 22,
        "year": "庚子",
        "year_order": 37
    },
    "lunar": {
        "animal": "鼠",
        "day": 4,
        "day_alias": "初四",
        "is_leap": true,
        "is_leap_month": false,
        "leap_month": 4,
        "month": 8,
        "month_alias": "八月",
        "year": 2020,
        "year_alias": "二零二零"
    },
    "solar": {
        "animal": "鼠",
        "constellation": "天秤",
        "day": 20,
        "hour": 5,
        "is_leep": true,
        "minute": 15,
        "month": 9,
        "nanosecond": 0,
        "second": 26,
        "week_alias": "日",
        "week_number": 0,
        "year": 2020
    }
}
```

# 输出日历

```
# calendar/calendar_test.go:192

公历日期   周   农历日期         干支               节气
----------------------------------------------------------------------------
2025-01-01 周三 2024龙年腊月初二 甲辰年丙子月庚午日
2025-01-02 周四 2024龙年腊月初三 甲辰年丙子月辛未日
2025-01-03 周五 2024龙年腊月初四 甲辰年丙子月壬申日
2025-01-04 周六 2024龙年腊月初五 甲辰年丙子月癸酉日
2025-01-05 周日 2024龙年腊月初六 甲辰年丙子月甲戌日 小寒 2025-01-05 10:32:31
2025-01-06 周一 2024龙年腊月初七 甲辰年丁丑月乙亥日
2025-01-07 周二 2024龙年腊月初八 甲辰年丁丑月丙子日
2025-01-08 周三 2024龙年腊月初九 甲辰年丁丑月丁丑日
2025-01-09 周四 2024龙年腊月初十 甲辰年丁丑月戊寅日
2025-01-10 周五 2024龙年腊月十一 甲辰年丁丑月己卯日
2025-01-11 周六 2024龙年腊月十二 甲辰年丁丑月庚辰日
2025-01-12 周日 2024龙年腊月十三 甲辰年丁丑月辛巳日
2025-01-13 周一 2024龙年腊月十四 甲辰年丁丑月壬午日
2025-01-14 周二 2024龙年腊月十五 甲辰年丁丑月癸未日
2025-01-15 周三 2024龙年腊月十六 甲辰年丁丑月甲申日
2025-01-16 周四 2024龙年腊月十七 甲辰年丁丑月乙酉日
2025-01-17 周五 2024龙年腊月十八 甲辰年丁丑月丙戌日
2025-01-18 周六 2024龙年腊月十九 甲辰年丁丑月丁亥日
2025-01-19 周日 2024龙年腊月二十 甲辰年丁丑月戊子日
2025-01-20 周一 2024龙年腊月廿一 甲辰年丁丑月己丑日 大寒 2025-01-20 03:59:52
2025-01-21 周二 2024龙年腊月廿二 甲辰年丁丑月庚寅日
2025-01-22 周三 2024龙年腊月廿三 甲辰年丁丑月辛卯日
2025-01-23 周四 2024龙年腊月廿四 甲辰年丁丑月壬辰日
2025-01-24 周五 2024龙年腊月廿五 甲辰年丁丑月癸巳日
2025-01-25 周六 2024龙年腊月廿六 甲辰年丁丑月甲午日
2025-01-26 周日 2024龙年腊月廿七 甲辰年丁丑月乙未日
2025-01-27 周一 2024龙年腊月廿八 甲辰年丁丑月丙申日
2025-01-28 周二 2024龙年腊月廿九 甲辰年丁丑月丁酉日
2025-01-29 周三 2025蛇年正月初一 甲辰年丁丑月戊戌日
2025-01-30 周四 2025蛇年正月初二 甲辰年丁丑月己亥日
2025-01-31 周五 2025蛇年正月初三 甲辰年丁丑月庚子日
2025-02-01 周六 2025蛇年正月初四 甲辰年丁丑月辛丑日
2025-02-02 周日 2025蛇年正月初五 甲辰年丁丑月壬寅日
2025-02-03 周一 2025蛇年正月初六 甲辰年丁丑月癸卯日 立春 2025-02-03 22:10:12
2025-02-04 周二 2025蛇年正月初七 乙巳年戊寅月甲辰日
2025-02-05 周三 2025蛇年正月初八 乙巳年戊寅月乙巳日
2025-02-06 周四 2025蛇年正月初九 乙巳年戊寅月丙午日
2025-02-07 周五 2025蛇年正月初十 乙巳年戊寅月丁未日
2025-02-08 周六 2025蛇年正月十一 乙巳年戊寅月戊申日
2025-02-09 周日 2025蛇年正月十二 乙巳年戊寅月己酉日
2025-02-10 周一 2025蛇年正月十三 乙巳年戊寅月庚戌日
2025-02-11 周二 2025蛇年正月十四 乙巳年戊寅月辛亥日
2025-02-12 周三 2025蛇年正月十五 乙巳年戊寅月壬子日
2025-02-13 周四 2025蛇年正月十六 乙巳年戊寅月癸丑日
2025-02-14 周五 2025蛇年正月十七 乙巳年戊寅月甲寅日
2025-02-15 周六 2025蛇年正月十八 乙巳年戊寅月乙卯日
2025-02-16 周日 2025蛇年正月十九 乙巳年戊寅月丙辰日
2025-02-17 周一 2025蛇年正月二十 乙巳年戊寅月丁巳日
2025-02-18 周二 2025蛇年正月廿一 乙巳年戊寅月戊午日 雨水 2025-02-18 18:06:18
2025-02-19 周三 2025蛇年正月廿二 乙巳年戊寅月己未日
2025-02-20 周四 2025蛇年正月廿三 乙巳年戊寅月庚申日
2025-02-21 周五 2025蛇年正月廿四 乙巳年戊寅月辛酉日
2025-02-22 周六 2025蛇年正月廿五 乙巳年戊寅月壬戌日
2025-02-23 周日 2025蛇年正月廿六 乙巳年戊寅月癸亥日
2025-02-24 周一 2025蛇年正月廿七 乙巳年戊寅月甲子日
2025-02-25 周二 2025蛇年正月廿八 乙巳年戊寅月乙丑日
2025-02-26 周三 2025蛇年正月廿九 乙巳年戊寅月丙寅日
2025-02-27 周四 2025蛇年正月三十 乙巳年戊寅月丁卯日
2025-02-28 周五 2025蛇年二月初一 乙巳年戊寅月戊辰日
2025-03-01 周六 2025蛇年二月初二 乙巳年戊寅月己巳日
2025-03-02 周日 2025蛇年二月初三 乙巳年戊寅月庚午日
2025-03-03 周一 2025蛇年二月初四 乙巳年戊寅月辛未日
2025-03-04 周二 2025蛇年二月初五 乙巳年戊寅月壬申日
2025-03-05 周三 2025蛇年二月初六 乙巳年戊寅月癸酉日 惊蛰 2025-03-05 16:07:02
2025-03-06 周四 2025蛇年二月初七 乙巳年己卯月甲戌日
2025-03-07 周五 2025蛇年二月初八 乙巳年己卯月乙亥日
2025-03-08 周六 2025蛇年二月初九 乙巳年己卯月丙子日
2025-03-09 周日 2025蛇年二月初十 乙巳年己卯月丁丑日
2025-03-10 周一 2025蛇年二月十一 乙巳年己卯月戊寅日
2025-03-11 周二 2025蛇年二月十二 乙巳年己卯月己卯日
2025-03-12 周三 2025蛇年二月十三 乙巳年己卯月庚辰日
2025-03-13 周四 2025蛇年二月十四 乙巳年己卯月辛巳日
2025-03-14 周五 2025蛇年二月十五 乙巳年己卯月壬午日
2025-03-15 周六 2025蛇年二月十六 乙巳年己卯月癸未日
2025-03-16 周日 2025蛇年二月十七 乙巳年己卯月甲申日
2025-03-17 周一 2025蛇年二月十八 乙巳年己卯月乙酉日
2025-03-18 周二 2025蛇年二月十九 乙巳年己卯月丙戌日
2025-03-19 周三 2025蛇年二月二十 乙巳年己卯月丁亥日
2025-03-20 周四 2025蛇年二月廿一 乙巳年己卯月戊子日 春分 2025-03-20 17:01:13
2025-03-21 周五 2025蛇年二月廿二 乙巳年己卯月己丑日
2025-03-22 周六 2025蛇年二月廿三 乙巳年己卯月庚寅日
2025-03-23 周日 2025蛇年二月廿四 乙巳年己卯月辛卯日
2025-03-24 周一 2025蛇年二月廿五 乙巳年己卯月壬辰日
2025-03-25 周二 2025蛇年二月廿六 乙巳年己卯月癸巳日
2025-03-26 周三 2025蛇年二月廿七 乙巳年己卯月甲午日
2025-03-27 周四 2025蛇年二月廿八 乙巳年己卯月乙未日
2025-03-28 周五 2025蛇年二月廿九 乙巳年己卯月丙申日
2025-03-29 周六 2025蛇年三月初一 乙巳年己卯月丁酉日
2025-03-30 周日 2025蛇年三月初二 乙巳年己卯月戊戌日
2025-03-31 周一 2025蛇年三月初三 乙巳年己卯月己亥日
2025-04-01 周二 2025蛇年三月初四 乙巳年己卯月庚子日
2025-04-02 周三 2025蛇年三月初五 乙巳年己卯月辛丑日
2025-04-03 周四 2025蛇年三月初六 乙巳年己卯月壬寅日
2025-04-04 周五 2025蛇年三月初七 乙巳年己卯月癸卯日 清明 2025-04-04 20:48:20
2025-04-05 周六 2025蛇年三月初八 乙巳年庚辰月甲辰日
2025-04-06 周日 2025蛇年三月初九 乙巳年庚辰月乙巳日
2025-04-07 周一 2025蛇年三月初十 乙巳年庚辰月丙午日
2025-04-08 周二 2025蛇年三月十一 乙巳年庚辰月丁未日
2025-04-09 周三 2025蛇年三月十二 乙巳年庚辰月戊申日
2025-04-10 周四 2025蛇年三月十三 乙巳年庚辰月己酉日
2025-04-11 周五 2025蛇年三月十四 乙巳年庚辰月庚戌日
2025-04-12 周六 2025蛇年三月十五 乙巳年庚辰月辛亥日
2025-04-13 周日 2025蛇年三月十六 乙巳年庚辰月壬子日
2025-04-14 周一 2025蛇年三月十七 乙巳年庚辰月癸丑日
2025-04-15 周二 2025蛇年三月十八 乙巳年庚辰月甲寅日
2025-04-16 周三 2025蛇年三月十九 乙巳年庚辰月乙卯日
2025-04-17 周四 2025蛇年三月二十 乙巳年庚辰月丙辰日
2025-04-18 周五 2025蛇年三月廿一 乙巳年庚辰月丁巳日
2025-04-19 周六 2025蛇年三月廿二 乙巳年庚辰月戊午日
2025-04-20 周日 2025蛇年三月廿三 乙巳年庚辰月己未日 谷雨 2025-04-20 03:55:45
2025-04-21 周一 2025蛇年三月廿四 乙巳年庚辰月庚申日
2025-04-22 周二 2025蛇年三月廿五 乙巳年庚辰月辛酉日
2025-04-23 周三 2025蛇年三月廿六 乙巳年庚辰月壬戌日
2025-04-24 周四 2025蛇年三月廿七 乙巳年庚辰月癸亥日
2025-04-25 周五 2025蛇年三月廿八 乙巳年庚辰月甲子日
2025-04-26 周六 2025蛇年三月廿九 乙巳年庚辰月乙丑日
2025-04-27 周日 2025蛇年三月三十 乙巳年庚辰月丙寅日
2025-04-28 周一 2025蛇年四月初一 乙巳年庚辰月丁卯日
2025-04-29 周二 2025蛇年四月初二 乙巳年庚辰月戊辰日
2025-04-30 周三 2025蛇年四月初三 乙巳年庚辰月己巳日
2025-05-01 周四 2025蛇年四月初四 乙巳年庚辰月庚午日
2025-05-02 周五 2025蛇年四月初五 乙巳年庚辰月辛未日
2025-05-03 周六 2025蛇年四月初六 乙巳年庚辰月壬申日
2025-05-04 周日 2025蛇年四月初七 乙巳年庚辰月癸酉日
2025-05-05 周一 2025蛇年四月初八 乙巳年庚辰月甲戌日 立夏 2025-05-05 13:56:57
2025-05-06 周二 2025蛇年四月初九 乙巳年辛巳月乙亥日
2025-05-07 周三 2025蛇年四月初十 乙巳年辛巳月丙子日
2025-05-08 周四 2025蛇年四月十一 乙巳年辛巳月丁丑日
2025-05-09 周五 2025蛇年四月十二 乙巳年辛巳月戊寅日
2025-05-10 周六 2025蛇年四月十三 乙巳年辛巳月己卯日
2025-05-11 周日 2025蛇年四月十四 乙巳年辛巳月庚辰日
2025-05-12 周一 2025蛇年四月十五 乙巳年辛巳月辛巳日
2025-05-13 周二 2025蛇年四月十六 乙巳年辛巳月壬午日
2025-05-14 周三 2025蛇年四月十七 乙巳年辛巳月癸未日
2025-05-15 周四 2025蛇年四月十八 乙巳年辛巳月甲申日
2025-05-16 周五 2025蛇年四月十九 乙巳年辛巳月乙酉日
2025-05-17 周六 2025蛇年四月二十 乙巳年辛巳月丙戌日
2025-05-18 周日 2025蛇年四月廿一 乙巳年辛巳月丁亥日
2025-05-19 周一 2025蛇年四月廿二 乙巳年辛巳月戊子日
2025-05-20 周二 2025蛇年四月廿三 乙巳年辛巳月己丑日
2025-05-21 周三 2025蛇年四月廿四 乙巳年辛巳月庚寅日 小满 2025-05-21 02:54:22
2025-05-22 周四 2025蛇年四月廿五 乙巳年辛巳月辛卯日
2025-05-23 周五 2025蛇年四月廿六 乙巳年辛巳月壬辰日
2025-05-24 周六 2025蛇年四月廿七 乙巳年辛巳月癸巳日
2025-05-25 周日 2025蛇年四月廿八 乙巳年辛巳月甲午日
2025-05-26 周一 2025蛇年四月廿九 乙巳年辛巳月乙未日
2025-05-27 周二 2025蛇年五月初一 乙巳年辛巳月丙申日
2025-05-28 周三 2025蛇年五月初二 乙巳年辛巳月丁酉日
2025-05-29 周四 2025蛇年五月初三 乙巳年辛巳月戊戌日
2025-05-30 周五 2025蛇年五月初四 乙巳年辛巳月己亥日
2025-05-31 周六 2025蛇年五月初五 乙巳年辛巳月庚子日
2025-06-01 周日 2025蛇年五月初六 乙巳年辛巳月辛丑日
2025-06-02 周一 2025蛇年五月初七 乙巳年辛巳月壬寅日
2025-06-03 周二 2025蛇年五月初八 乙巳年辛巳月癸卯日
2025-06-04 周三 2025蛇年五月初九 乙巳年辛巳月甲辰日
2025-06-05 周四 2025蛇年五月初十 乙巳年辛巳月乙巳日 芒种 2025-06-05 17:56:16
2025-06-06 周五 2025蛇年五月十一 乙巳年壬午月丙午日
2025-06-07 周六 2025蛇年五月十二 乙巳年壬午月丁未日
2025-06-08 周日 2025蛇年五月十三 乙巳年壬午月戊申日
2025-06-09 周一 2025蛇年五月十四 乙巳年壬午月己酉日
2025-06-10 周二 2025蛇年五月十五 乙巳年壬午月庚戌日
2025-06-11 周三 2025蛇年五月十六 乙巳年壬午月辛亥日
2025-06-12 周四 2025蛇年五月十七 乙巳年壬午月壬子日
2025-06-13 周五 2025蛇年五月十八 乙巳年壬午月癸丑日
2025-06-14 周六 2025蛇年五月十九 乙巳年壬午月甲寅日
2025-06-15 周日 2025蛇年五月二十 乙巳年壬午月乙卯日
2025-06-16 周一 2025蛇年五月廿一 乙巳年壬午月丙辰日
2025-06-17 周二 2025蛇年五月廿二 乙巳年壬午月丁巳日
2025-06-18 周三 2025蛇年五月廿三 乙巳年壬午月戊午日
2025-06-19 周四 2025蛇年五月廿四 乙巳年壬午月己未日
2025-06-20 周五 2025蛇年五月廿五 乙巳年壬午月庚申日
2025-06-21 周六 2025蛇年五月廿六 乙巳年壬午月辛酉日 夏至 2025-06-21 10:41:59
2025-06-22 周日 2025蛇年五月廿七 乙巳年壬午月壬戌日
2025-06-23 周一 2025蛇年五月廿八 乙巳年壬午月癸亥日
2025-06-24 周二 2025蛇年五月廿九 乙巳年壬午月甲子日
2025-06-25 周三 2025蛇年六月初一 乙巳年壬午月乙丑日
2025-06-26 周四 2025蛇年六月初二 乙巳年壬午月丙寅日
2025-06-27 周五 2025蛇年六月初三 乙巳年壬午月丁卯日
2025-06-28 周六 2025蛇年六月初四 乙巳年壬午月戊辰日
2025-06-29 周日 2025蛇年六月初五 乙巳年壬午月己巳日
2025-06-30 周一 2025蛇年六月初六 乙巳年壬午月庚午日
2025-07-01 周二 2025蛇年六月初七 乙巳年壬午月辛未日
2025-07-02 周三 2025蛇年六月初八 乙巳年壬午月壬申日
2025-07-03 周四 2025蛇年六月初九 乙巳年壬午月癸酉日
2025-07-04 周五 2025蛇年六月初十 乙巳年壬午月甲戌日
2025-07-05 周六 2025蛇年六月十一 乙巳年壬午月乙亥日
2025-07-06 周日 2025蛇年六月十二 乙巳年壬午月丙子日
2025-07-07 周一 2025蛇年六月十三 乙巳年壬午月丁丑日 小暑 2025-07-07 04:04:43
2025-07-08 周二 2025蛇年六月十四 乙巳年癸未月戊寅日
2025-07-09 周三 2025蛇年六月十五 乙巳年癸未月己卯日
2025-07-10 周四 2025蛇年六月十六 乙巳年癸未月庚辰日
2025-07-11 周五 2025蛇年六月十七 乙巳年癸未月辛巳日
2025-07-12 周六 2025蛇年六月十八 乙巳年癸未月壬午日
2025-07-13 周日 2025蛇年六月十九 乙巳年癸未月癸未日
2025-07-14 周一 2025蛇年六月二十 乙巳年癸未月甲申日
2025-07-15 周二 2025蛇年六月廿一 乙巳年癸未月乙酉日
2025-07-16 周三 2025蛇年六月廿二 乙巳年癸未月丙戌日
2025-07-17 周四 2025蛇年六月廿三 乙巳年癸未月丁亥日
2025-07-18 周五 2025蛇年六月廿四 乙巳年癸未月戊子日
2025-07-19 周六 2025蛇年六月廿五 乙巳年癸未月己丑日
2025-07-20 周日 2025蛇年六月廿六 乙巳年癸未月庚寅日
2025-07-21 周一 2025蛇年六月廿七 乙巳年癸未月辛卯日
2025-07-22 周二 2025蛇年六月廿八 乙巳年癸未月壬辰日 大暑 2025-07-22 21:29:10
2025-07-23 周三 2025蛇年六月廿九 乙巳年癸未月癸巳日
2025-07-24 周四 2025蛇年六月三十 乙巳年癸未月甲午日
2025-07-25 周五 2025蛇年闰六月初一 乙巳年癸未月乙未日
2025-07-26 周六 2025蛇年闰六月初二 乙巳年癸未月丙申日
2025-07-27 周日 2025蛇年闰六月初三 乙巳年癸未月丁酉日
2025-07-28 周一 2025蛇年闰六月初四 乙巳年癸未月戊戌日
2025-07-29 周二 2025蛇年闰六月初五 乙巳年癸未月己亥日
2025-07-30 周三 2025蛇年闰六月初六 乙巳年癸未月庚子日
2025-07-31 周四 2025蛇年闰六月初七 乙巳年癸未月辛丑日
2025-08-01 周五 2025蛇年闰六月初八 乙巳年癸未月壬寅日
2025-08-02 周六 2025蛇年闰六月初九 乙巳年癸未月癸卯日
2025-08-03 周日 2025蛇年闰六月初十 乙巳年癸未月甲辰日
2025-08-04 周一 2025蛇年闰六月十一 乙巳年癸未月乙巳日
2025-08-05 周二 2025蛇年闰六月十二 乙巳年癸未月丙午日
2025-08-06 周三 2025蛇年闰六月十三 乙巳年癸未月丁未日
2025-08-07 周四 2025蛇年闰六月十四 乙巳年癸未月戊申日 立秋 2025-08-07 13:51:18
2025-08-08 周五 2025蛇年闰六月十五 乙巳年甲申月己酉日
2025-08-09 周六 2025蛇年闰六月十六 乙巳年甲申月庚戌日
2025-08-10 周日 2025蛇年闰六月十七 乙巳年甲申月辛亥日
2025-08-11 周一 2025蛇年闰六月十八 乙巳年甲申月壬子日
2025-08-12 周二 2025蛇年闰六月十九 乙巳年甲申月癸丑日
2025-08-13 周三 2025蛇年闰六月二十 乙巳年甲申月甲寅日
2025-08-14 周四 2025蛇年闰六月廿一 乙巳年甲申月乙卯日
2025-08-15 周五 2025蛇年闰六月廿二 乙巳年甲申月丙辰日
2025-08-16 周六 2025蛇年闰六月廿三 乙巳年甲申月丁巳日
2025-08-17 周日 2025蛇年闰六月廿四 乙巳年甲申月戊午日
2025-08-18 周一 2025蛇年闰六月廿五 乙巳年甲申月己未日
2025-08-19 周二 2025蛇年闰六月廿六 乙巳年甲申月庚申日
2025-08-20 周三 2025蛇年闰六月廿七 乙巳年甲申月辛酉日
2025-08-21 周四 2025蛇年闰六月廿八 乙巳年甲申月壬戌日
2025-08-22 周五 2025蛇年闰六月廿九 乙巳年甲申月癸亥日
2025-08-23 周六 2025蛇年七月初一 乙巳年甲申月甲子日 处暑 2025-08-23 04:33:35
2025-08-24 周日 2025蛇年七月初二 乙巳年甲申月乙丑日
2025-08-25 周一 2025蛇年七月初三 乙巳年甲申月丙寅日
2025-08-26 周二 2025蛇年七月初四 乙巳年甲申月丁卯日
2025-08-27 周三 2025蛇年七月初五 乙巳年甲申月戊辰日
2025-08-28 周四 2025蛇年七月初六 乙巳年甲申月己巳日
2025-08-29 周五 2025蛇年七月初七 乙巳年甲申月庚午日
2025-08-30 周六 2025蛇年七月初八 乙巳年甲申月辛未日
2025-08-31 周日 2025蛇年七月初九 乙巳年甲申月壬申日
2025-09-01 周一 2025蛇年七月初十 乙巳年甲申月癸酉日
2025-09-02 周二 2025蛇年七月十一 乙巳年甲申月甲戌日
2025-09-03 周三 2025蛇年七月十二 乙巳年甲申月乙亥日
2025-09-04 周四 2025蛇年七月十三 乙巳年甲申月丙子日
2025-09-05 周五 2025蛇年七月十四 乙巳年甲申月丁丑日
2025-09-06 周六 2025蛇年七月十五 乙巳年甲申月戊寅日
2025-09-07 周日 2025蛇年七月十六 乙巳年甲申月己卯日 白露 2025-09-07 16:51:41
2025-09-08 周一 2025蛇年七月十七 乙巳年乙酉月庚辰日
2025-09-09 周二 2025蛇年七月十八 乙巳年乙酉月辛巳日
2025-09-10 周三 2025蛇年七月十九 乙巳年乙酉月壬午日
2025-09-11 周四 2025蛇年七月二十 乙巳年乙酉月癸未日
2025-09-12 周五 2025蛇年七月廿一 乙巳年乙酉月甲申日
2025-09-13 周六 2025蛇年七月廿二 乙巳年乙酉月乙酉日
2025-09-14 周日 2025蛇年七月廿三 乙巳年乙酉月丙戌日
2025-09-15 周一 2025蛇年七月廿四 乙巳年乙酉月丁亥日
2025-09-16 周二 2025蛇年七月廿五 乙巳年乙酉月戊子日
2025-09-17 周三 2025蛇年七月廿六 乙巳年乙酉月己丑日
2025-09-18 周四 2025蛇年七月廿七 乙巳年乙酉月庚寅日
2025-09-19 周五 2025蛇年七月廿八 乙巳年乙酉月辛卯日
2025-09-20 周六 2025蛇年七月廿九 乙巳年乙酉月壬辰日
2025-09-21 周日 2025蛇年七月三十 乙巳年乙酉月癸巳日
2025-09-22 周一 2025蛇年八月初一 乙巳年乙酉月甲午日
2025-09-23 周二 2025蛇年八月初二 乙巳年乙酉月乙未日 秋分 2025-09-23 02:19:04
2025-09-24 周三 2025蛇年八月初三 乙巳年乙酉月丙申日
2025-09-25 周四 2025蛇年八月初四 乙巳年乙酉月丁酉日
2025-09-26 周五 2025蛇年八月初五 乙巳年乙酉月戊戌日
2025-09-27 周六 2025蛇年八月初六 乙巳年乙酉月己亥日
2025-09-28 周日 2025蛇年八月初七 乙巳年乙酉月庚子日
2025-09-29 周一 2025蛇年八月初八 乙巳年乙酉月辛丑日
2025-09-30 周二 2025蛇年八月初九 乙巳年乙酉月壬寅日
2025-10-01 周三 2025蛇年八月初十 乙巳年乙酉月癸卯日
2025-10-02 周四 2025蛇年八月十一 乙巳年乙酉月甲辰日
2025-10-03 周五 2025蛇年八月十二 乙巳年乙酉月乙巳日
2025-10-04 周六 2025蛇年八月十三 乙巳年乙酉月丙午日
2025-10-05 周日 2025蛇年八月十四 乙巳年乙酉月丁未日
2025-10-06 周一 2025蛇年八月十五 乙巳年乙酉月戊申日
2025-10-07 周二 2025蛇年八月十六 乙巳年乙酉月己酉日
2025-10-08 周三 2025蛇年八月十七 乙巳年乙酉月庚戌日 寒露 2025-10-08 08:40:56
2025-10-09 周四 2025蛇年八月十八 乙巳年丙戌月辛亥日
2025-10-10 周五 2025蛇年八月十九 乙巳年丙戌月壬子日
2025-10-11 周六 2025蛇年八月二十 乙巳年丙戌月癸丑日
2025-10-12 周日 2025蛇年八月廿一 乙巳年丙戌月甲寅日
2025-10-13 周一 2025蛇年八月廿二 乙巳年丙戌月乙卯日
2025-10-14 周二 2025蛇年八月廿三 乙巳年丙戌月丙辰日
2025-10-15 周三 2025蛇年八月廿四 乙巳年丙戌月丁巳日
2025-10-16 周四 2025蛇年八月廿五 乙巳年丙戌月戊午日
2025-10-17 周五 2025蛇年八月廿六 乙巳年丙戌月己未日
2025-10-18 周六 2025蛇年八月廿七 乙巳年丙戌月庚申日
2025-10-19 周日 2025蛇年八月廿八 乙巳年丙戌月辛酉日
2025-10-20 周一 2025蛇年八月廿九 乙巳年丙戌月壬戌日
2025-10-21 周二 2025蛇年九月初一 乙巳年丙戌月癸亥日
2025-10-22 周三 2025蛇年九月初二 乙巳年丙戌月甲子日
2025-10-23 周四 2025蛇年九月初三 乙巳年丙戌月乙丑日 霜降 2025-10-23 11:50:39
2025-10-24 周五 2025蛇年九月初四 乙巳年丙戌月丙寅日
2025-10-25 周六 2025蛇年九月初五 乙巳年丙戌月丁卯日
2025-10-26 周日 2025蛇年九月初六 乙巳年丙戌月戊辰日
2025-10-27 周一 2025蛇年九月初七 乙巳年丙戌月己巳日
2025-10-28 周二 2025蛇年九月初八 乙巳年丙戌月庚午日
2025-10-29 周三 2025蛇年九月初九 乙巳年丙戌月辛未日
2025-10-30 周四 2025蛇年九月初十 乙巳年丙戌月壬申日
2025-10-31 周五 2025蛇年九月十一 乙巳年丙戌月癸酉日
2025-11-01 周六 2025蛇年九月十二 乙巳年丙戌月甲戌日
2025-11-02 周日 2025蛇年九月十三 乙巳年丙戌月乙亥日
2025-11-03 周一 2025蛇年九月十四 乙巳年丙戌月丙子日
2025-11-04 周二 2025蛇年九月十五 乙巳年丙戌月丁丑日
2025-11-05 周三 2025蛇年九月十六 乙巳年丙戌月戊寅日
2025-11-06 周四 2025蛇年九月十七 乙巳年丙戌月己卯日
2025-11-07 周五 2025蛇年九月十八 乙巳年丙戌月庚辰日 立冬 2025-11-07 12:03:47
2025-11-08 周六 2025蛇年九月十九 乙巳年丁亥月辛巳日
2025-11-09 周日 2025蛇年九月二十 乙巳年丁亥月壬午日
2025-11-10 周一 2025蛇年九月廿一 乙巳年丁亥月癸未日
2025-11-11 周二 2025蛇年九月廿二 乙巳年丁亥月甲申日
2025-11-12 周三 2025蛇年九月廿三 乙巳年丁亥月乙酉日
2025-11-13 周四 2025蛇年九月廿四 乙巳年丁亥月丙戌日
2025-11-14 周五 2025蛇年九月廿五 乙巳年丁亥月丁亥日
2025-11-15 周六 2025蛇年九月廿六 乙巳年丁亥月戊子日
2025-11-16 周日 2025蛇年九月廿七 乙巳年丁亥月己丑日
2025-11-17 周一 2025蛇年九月廿八 乙巳年丁亥月庚寅日
2025-11-18 周二 2025蛇年九月廿九 乙巳年丁亥月辛卯日
2025-11-19 周三 2025蛇年九月三十 乙巳年丁亥月壬辰日
2025-11-20 周四 2025蛇年十月初一 乙巳年丁亥月癸巳日
2025-11-21 周五 2025蛇年十月初二 乙巳年丁亥月甲午日
2025-11-22 周六 2025蛇年十月初三 乙巳年丁亥月乙未日 小雪 2025-11-22 09:35:18
2025-11-23 周日 2025蛇年十月初四 乙巳年丁亥月丙申日
2025-11-24 周一 2025蛇年十月初五 乙巳年丁亥月丁酉日
2025-11-25 周二 2025蛇年十月初六 乙巳年丁亥月戊戌日
2025-11-26 周三 2025蛇年十月初七 乙巳年丁亥月己亥日
2025-11-27 周四 2025蛇年十月初八 乙巳年丁亥月庚子日
2025-11-28 周五 2025蛇年十月初九 乙巳年丁亥月辛丑日
2025-11-29 周六 2025蛇年十月初十 乙巳年丁亥月壬寅日
2025-11-30 周日 2025蛇年十月十一 乙巳年丁亥月癸卯日
2025-12-01 周一 2025蛇年十月十二 乙巳年丁亥月甲辰日
2025-12-02 周二 2025蛇年十月十三 乙巳年丁亥月乙巳日
2025-12-03 周三 2025蛇年十月十四 乙巳年丁亥月丙午日
2025-12-04 周四 2025蛇年十月十五 乙巳年丁亥月丁未日
2025-12-05 周五 2025蛇年十月十六 乙巳年丁亥月戊申日
2025-12-06 周六 2025蛇年十月十七 乙巳年丁亥月己酉日
2025-12-07 周日 2025蛇年十月十八 乙巳年丁亥月庚戌日 大雪 2025-12-07 05:04:19
2025-12-08 周一 2025蛇年十月十九 乙巳年戊子月辛亥日
2025-12-09 周二 2025蛇年十月二十 乙巳年戊子月壬子日
2025-12-10 周三 2025蛇年十月廿一 乙巳年戊子月癸丑日
2025-12-11 周四 2025蛇年十月廿二 乙巳年戊子月甲寅日
2025-12-12 周五 2025蛇年十月廿三 乙巳年戊子月乙卯日
2025-12-13 周六 2025蛇年十月廿四 乙巳年戊子月丙辰日
2025-12-14 周日 2025蛇年十月廿五 乙巳年戊子月丁巳日
2025-12-15 周一 2025蛇年十月廿六 乙巳年戊子月戊午日
2025-12-16 周二 2025蛇年十月廿七 乙巳年戊子月己未日
2025-12-17 周三 2025蛇年十月廿八 乙巳年戊子月庚申日
2025-12-18 周四 2025蛇年十月廿九 乙巳年戊子月辛酉日
2025-12-19 周五 2025蛇年十月三十 乙巳年戊子月壬戌日
2025-12-20 周六 2025蛇年冬月初一 乙巳年戊子月癸亥日
2025-12-21 周日 2025蛇年冬月初二 乙巳年戊子月甲子日 冬至 2025-12-21 23:02:48
2025-12-22 周一 2025蛇年冬月初三 乙巳年戊子月乙丑日
2025-12-23 周二 2025蛇年冬月初四 乙巳年戊子月丙寅日
2025-12-24 周三 2025蛇年冬月初五 乙巳年戊子月丁卯日
2025-12-25 周四 2025蛇年冬月初六 乙巳年戊子月戊辰日
2025-12-26 周五 2025蛇年冬月初七 乙巳年戊子月己巳日
2025-12-27 周六 2025蛇年冬月初八 乙巳年戊子月庚午日
2025-12-28 周日 2025蛇年冬月初九 乙巳年戊子月辛未日
2025-12-29 周一 2025蛇年冬月初十 乙巳年戊子月壬申日
2025-12-30 周二 2025蛇年冬月十一 乙巳年戊子月癸酉日
2025-12-31 周三 2025蛇年冬月十二 乙巳年戊子月甲戌日

公历日期   周   农历日期         干支               节气
----------------------------------------------------------------------------
3000-01-01 周三 2999羊年腊月初四 己未年丙子月辛巳日
3000-01-02 周四 2999羊年腊月初五 己未年丙子月壬午日
3000-01-03 周五 2999羊年腊月初六 己未年丙子月癸未日
3000-01-04 周六 2999羊年腊月初七 己未年丙子月甲申日
3000-01-05 周日 2999羊年腊月初八 己未年丙子月乙酉日
3000-01-06 周一 2999羊年腊月初九 己未年丙子月丙戌日 小寒 3000-01-06 00:47:25
3000-01-07 周二 2999羊年腊月初十 己未年丁丑月丁亥日
3000-01-08 周三 2999羊年腊月十一 己未年丁丑月戊子日
3000-01-09 周四 2999羊年腊月十二 己未年丁丑月己丑日
3000-01-10 周五 2999羊年腊月十三 己未年丁丑月庚寅日
3000-01-11 周六 2999羊年腊月十四 己未年丁丑月辛卯日
3000-01-12 周日 2999羊年腊月十五 己未年丁丑月壬辰日
3000-01-13 周一 2999羊年腊月十六 己未年丁丑月癸巳日
3000-01-14 周二 2999羊年腊月十七 己未年丁丑月甲午日
3000-01-15 周三 2999羊年腊月十八 己未年丁丑月乙未日
3000-01-16 周四 2999羊年腊月十九 己未年丁丑月丙申日
3000-01-17 周五 2999羊年腊月二十 己未年丁丑月丁酉日
3000-01-18 周六 2999羊年腊月廿一 己未年丁丑月戊戌日
3000-01-19 周日 2999羊年腊月廿二 己未年丁丑月己亥日
3000-01-20 周一 2999羊年腊月廿三 己未年丁丑月庚子日 大寒 3000-01-20 18:30:18
3000-01-21 周二 2999羊年腊月廿四 己未年丁丑月辛丑日
3000-01-22 周三 2999羊年腊月廿五 己未年丁丑月壬寅日
3000-01-23 周四 2999羊年腊月廿六 己未年丁丑月癸卯日
3000-01-24 周五 2999羊年腊月廿七 己未年丁丑月甲辰日
3000-01-25 周六 2999羊年腊月廿八 己未年丁丑月乙巳日
3000-01-26 周日 2999羊年腊月廿九 己未年丁丑月丙午日
3000-01-27 周一 2999羊年腊月三十 己未年丁丑月丁未日
3000-01-28 周二 3000猴年正月初一 己未年丁丑月戊申日
3000-01-29 周三 3000猴年正月初二 己未年丁丑月己酉日
3000-01-30 周四 3000猴年正月初三 己未年丁丑月庚戌日
3000-01-31 周五 3000猴年正月初四 己未年丁丑月辛亥日
3000-02-01 周六 3000猴年正月初五 己未年丁丑月壬子日
3000-02-02 周日 3000猴年正月初六 己未年丁丑月癸丑日
3000-02-03 周一 3000猴年正月初七 己未年丁丑月甲寅日
3000-02-04 周二 3000猴年正月初八 己未年丁丑月乙卯日 立春 3000-02-04 12:02:17
3000-02-05 周三 3000猴年正月初九 庚申年戊寅月丙辰日
3000-02-06 周四 3000猴年正月初十 庚申年戊寅月丁巳日
3000-02-07 周五 3000猴年正月十一 庚申年戊寅月戊午日
3000-02-08 周六 3000猴年正月十二 庚申年戊寅月己未日
3000-02-09 周日 3000猴年正月十三 庚申年戊寅月庚申日
3000-02-10 周一 3000猴年正月十四 庚申年戊寅月辛酉日
3000-02-11 周二 3000猴年正月十五 庚申年戊寅月壬戌日
3000-02-12 周三 3000猴年正月十六 庚申年戊寅月癸亥日
3000-02-13 周四 3000猴年正月十七 庚申年戊寅月甲子日
3000-02-14 周五 3000猴年正月十八 庚申年戊寅月乙丑日
3000-02-15 周六 3000猴年正月十九 庚申年戊寅月丙寅日
3000-02-16 周日 3000猴年正月二十 庚申年戊寅月丁卯日
3000-02-17 周一 3000猴年正月廿一 庚申年戊寅月戊辰日
3000-02-18 周二 3000猴年正月廿二 庚申年戊寅月己巳日
3000-02-19 周三 3000猴年正月廿三 庚申年戊寅月庚午日 雨水 3000-02-19 06:29:44
3000-02-20 周四 3000猴年正月廿四 庚申年戊寅月辛未日
3000-02-21 周五 3000猴年正月廿五 庚申年戊寅月壬申日
3000-02-22 周六 3000猴年正月廿六 庚申年戊寅月癸酉日
3000-02-23 周日 3000猴年正月廿七 庚申年戊寅月甲戌日
3000-02-24 周一 3000猴年正月廿八 庚申年戊寅月乙亥日
3000-02-25 周二 3000猴年正月廿九 庚申年戊寅月丙子日
3000-02-26 周三 3000猴年二月初一 庚申年戊寅月丁丑日
3000-02-27 周四 3000猴年二月初二 庚申年戊寅月戊寅日
3000-02-28 周五 3000猴年二月初三 庚申年戊寅月己卯日
3000-03-01 周六 3000猴年二月初四 庚申年戊寅月庚辰日
3000-03-02 周日 3000猴年二月初五 庚申年戊寅月辛巳日
3000-03-03 周一 3000猴年二月初六 庚申年戊寅月壬午日
3000-03-04 周二 3000猴年二月初七 庚申年戊寅月癸未日
3000-03-05 周三 3000猴年二月初八 庚申年戊寅月甲申日
3000-03-06 周四 3000猴年二月初九 庚申年戊寅月乙酉日 惊蛰 3000-03-06 02:15:30
3000-03-07 周五 3000猴年二月初十 庚申年己卯月丙戌日
3000-03-08 周六 3000猴年二月十一 庚申年己卯月丁亥日
3000-03-09 周日 3000猴年二月十二 庚申年己卯月戊子日
3000-03-10 周一 3000猴年二月十三 庚申年己卯月己丑日
3000-03-11 周二 3000猴年二月十四 庚申年己卯月庚寅日
3000-03-12 周三 3000猴年二月十五 庚申年己卯月辛卯日
3000-03-13 周四 3000猴年二月十六 庚申年己卯月壬辰日
3000-03-14 周五 3000猴年二月十七 庚申年己卯月癸巳日
3000-03-15 周六 3000猴年二月十八 庚申年己卯月甲午日
3000-03-16 周日 3000猴年二月十九 庚申年己卯月乙未日
3000-03-17 周一 3000猴年二月二十 庚申年己卯月丙申日
3000-03-18 周二 3000猴年二月廿一 庚申年己卯月丁酉日
3000-03-19 周三 3000猴年二月廿二 庚申年己卯月戊戌日
3000-03-20 周四 3000猴年二月廿三 庚申年己卯月己亥日
3000-03-21 周五 3000猴年二月廿四 庚申年己卯月庚子日 春分 3000-03-21 00:18:08
3000-03-22 周六 3000猴年二月廿五 庚申年己卯月辛丑日
3000-03-23 周日 3000猴年二月廿六 庚申年己卯月壬寅日
3000-03-24 周一 3000猴年二月廿七 庚申年己卯月癸卯日
3000-03-25 周二 3000猴年二月廿八 庚申年己卯月甲辰日
3000-03-26 周三 3000猴年二月廿九 庚申年己卯月乙巳日
3000-03-27 周四 3000猴年二月三十 庚申年己卯月丙午日
3000-03-28 周五 3000猴年三月初一 庚申年己卯月丁未日
3000-03-29 周六 3000猴年三月初二 庚申年己卯月戊申日
3000-03-30 周日 3000猴年三月初三 庚申年己卯月己酉日
3000-03-31 周一 3000猴年三月初四 庚申年己卯月庚戌日
3000-04-01 周二 3000猴年三月初五 庚申年己卯月辛亥日
3000-04-02 周三 3000猴年三月初六 庚申年己卯月壬子日
3000-04-03 周四 3000猴年三月初七 庚申年己卯月癸丑日
3000-04-04 周五 3000猴年三月初八 庚申年己卯月甲寅日
3000-04-05 周六 3000猴年三月初九 庚申年己卯月乙卯日 清明 3000-04-05 00:47:14
3000-04-06 周日 3000猴年三月初十 庚申年庚辰月丙辰日
3000-04-07 周一 3000猴年三月十一 庚申年庚辰月丁巳日
3000-04-08 周二 3000猴年三月十二 庚申年庚辰月戊午日
3000-04-09 周三 3000猴年三月十三 庚申年庚辰月己未日
3000-04-10 周四 3000猴年三月十四 庚申年庚辰月庚申日
3000-04-11 周五 3000猴年三月十五 庚申年庚辰月辛酉日
3000-04-12 周六 3000猴年三月十六 庚申年庚辰月壬戌日
3000-04-13 周日 3000猴年三月十七 庚申年庚辰月癸亥日
3000-04-14 周一 3000猴年三月十八 庚申年庚辰月甲子日
3000-04-15 周二 3000猴年三月十九 庚申年庚辰月乙丑日
3000-04-16 周三 3000猴年三月二十 庚申年庚辰月丙寅日
3000-04-17 周四 3000猴年三月廿一 庚申年庚辰月丁卯日
3000-04-18 周五 3000猴年三月廿二 庚申年庚辰月戊辰日
3000-04-19 周六 3000猴年三月廿三 庚申年庚辰月己巳日
3000-04-20 周日 3000猴年三月廿四 庚申年庚辰月庚午日 谷雨 3000-04-20 04:23:52
3000-04-21 周一 3000猴年三月廿五 庚申年庚辰月辛未日
3000-04-22 周二 3000猴年三月廿六 庚申年庚辰月壬申日
3000-04-23 周三 3000猴年三月廿七 庚申年庚辰月癸酉日
3000-04-24 周四 3000猴年三月廿八 庚申年庚辰月甲戌日
3000-04-25 周五 3000猴年三月廿九 庚申年庚辰月乙亥日
3000-04-26 周六 3000猴年四月初一 庚申年庚辰月丙子日
3000-04-27 周日 3000猴年四月初二 庚申年庚辰月丁丑日
3000-04-28 周一 3000猴年四月初三 庚申年庚辰月戊寅日
3000-04-29 周二 3000猴年四月初四 庚申年庚辰月己卯日
3000-04-30 周三 3000猴年四月初五 庚申年庚辰月庚辰日
3000-05-01 周四 3000猴年四月初六 庚申年庚辰月辛巳日
3000-05-02 周五 3000猴年四月初七 庚申年庚辰月壬午日
3000-05-03 周六 3000猴年四月初八 庚申年庚辰月癸未日
3000-05-04 周日 3000猴年四月初九 庚申年庚辰月甲申日
3000-05-05 周一 3000猴年四月初十 庚申年庚辰月乙酉日 立夏 3000-05-05 10:56:07
3000-05-06 周二 3000猴年四月十一 庚申年辛巳月丙戌日
3000-05-07 周三 3000猴年四月十二 庚申年辛巳月丁亥日
3000-05-08 周四 3000猴年四月十三 庚申年辛巳月戊子日
3000-05-09 周五 3000猴年四月十四 庚申年辛巳月己丑日
3000-05-10 周六 3000猴年四月十五 庚申年辛巳月庚寅日
3000-05-11 周日 3000猴年四月十六 庚申年辛巳月辛卯日
3000-05-12 周一 3000猴年四月十七 庚申年辛巳月壬辰日
3000-05-13 周二 3000猴年四月十八 庚申年辛巳月癸巳日
3000-05-14 周三 3000猴年四月十九 庚申年辛巳月甲午日
3000-05-15 周四 3000猴年四月二十 庚申年辛巳月乙未日
3000-05-16 周五 3000猴年四月廿一 庚申年辛巳月丙申日
3000-05-17 周六 3000猴年四月廿二 庚申年辛巳月丁酉日
3000-05-18 周日 3000猴年四月廿三 庚申年辛巳月戊戌日
3000-05-19 周一 3000猴年四月廿四 庚申年辛巳月己亥日
3000-05-20 周二 3000猴年四月廿五 庚申年辛巳月庚子日 小满 3000-05-20 20:40:12
3000-05-21 周三 3000猴年四月廿六 庚申年辛巳月辛丑日
3000-05-22 周四 3000猴年四月廿七 庚申年辛巳月壬寅日
3000-05-23 周五 3000猴年四月廿八 庚申年辛巳月癸卯日
3000-05-24 周六 3000猴年四月廿九 庚申年辛巳月甲辰日
3000-05-25 周日 3000猴年四月三十 庚申年辛巳月乙巳日
3000-05-26 周一 3000猴年五月初一 庚申年辛巳月丙午日
3000-05-27 周二 3000猴年五月初二 庚申年辛巳月丁未日
3000-05-28 周三 3000猴年五月初三 庚申年辛巳月戊申日
3000-05-29 周四 3000猴年五月初四 庚申年辛巳月己酉日
3000-05-30 周五 3000猴年五月初五 庚申年辛巳月庚戌日
3000-05-31 周六 3000猴年五月初六 庚申年辛巳月辛亥日
3000-06-01 周日 3000猴年五月初七 庚申年辛巳月壬子日
3000-06-02 周一 3000猴年五月初八 庚申年辛巳月癸丑日
3000-06-03 周二 3000猴年五月初九 庚申年辛巳月甲寅日
3000-06-04 周三 3000猴年五月初十 庚申年辛巳月乙卯日
3000-06-05 周四 3000猴年五月十一 庚申年辛巳月丙辰日 芒种 3000-06-05 08:59:56
3000-06-06 周五 3000猴年五月十二 庚申年壬午月丁巳日
3000-06-07 周六 3000猴年五月十三 庚申年壬午月戊午日
3000-06-08 周日 3000猴年五月十四 庚申年壬午月己未日
3000-06-09 周一 3000猴年五月十五 庚申年壬午月庚申日
3000-06-10 周二 3000猴年五月十六 庚申年壬午月辛酉日
3000-06-11 周三 3000猴年五月十七 庚申年壬午月壬戌日
3000-06-12 周四 3000猴年五月十八 庚申年壬午月癸亥日
3000-06-13 周五 3000猴年五月十九 庚申年壬午月甲子日
3000-06-14 周六 3000猴年五月二十 庚申年壬午月乙丑日
3000-06-15 周日 3000猴年五月廿一 庚申年壬午月丙寅日
3000-06-16 周一 3000猴年五月廿二 庚申年壬午月丁卯日
3000-06-17 周二 3000猴年五月廿三 庚申年壬午月戊辰日
3000-06-18 周三 3000猴年五月廿四 庚申年壬午月己巳日
3000-06-19 周四 3000猴年五月廿五 庚申年壬午月庚午日
3000-06-20 周五 3000猴年五月廿六 庚申年壬午月辛未日 夏至 3000-06-20 23:44:14
3000-06-21 周六 3000猴年五月廿七 庚申年壬午月壬申日
3000-06-22 周日 3000猴年五月廿八 庚申年壬午月癸酉日
3000-06-23 周一 3000猴年五月廿九 庚申年壬午月甲戌日
3000-06-24 周二 3000猴年六月初一 庚申年壬午月乙亥日
3000-06-25 周三 3000猴年六月初二 庚申年壬午月丙子日
3000-06-26 周四 3000猴年六月初三 庚申年壬午月丁丑日
3000-06-27 周五 3000猴年六月初四 庚申年壬午月戊寅日
3000-06-28 周六 3000猴年六月初五 庚申年壬午月己卯日
3000-06-29 周日 3000猴年六月初六 庚申年壬午月庚辰日
3000-06-30 周一 3000猴年六月初七 庚申年壬午月辛巳日
3000-07-01 周二 3000猴年六月初八 庚申年壬午月壬午日
3000-07-02 周三 3000猴年六月初九 庚申年壬午月癸未日
3000-07-03 周四 3000猴年六月初十 庚申年壬午月甲申日
3000-07-04 周五 3000猴年六月十一 庚申年壬午月乙酉日
3000-07-05 周六 3000猴年六月十二 庚申年壬午月丙戌日
3000-07-06 周日 3000猴年六月十三 庚申年壬午月丁亥日 小暑 3000-07-06 15:58:11
3000-07-07 周一 3000猴年六月十四 庚申年癸未月戊子日
3000-07-08 周二 3000猴年六月十五 庚申年癸未月己丑日
3000-07-09 周三 3000猴年六月十六 庚申年癸未月庚寅日
3000-07-10 周四 3000猴年六月十七 庚申年癸未月辛卯日
3000-07-11 周五 3000猴年六月十八 庚申年癸未月壬辰日
3000-07-12 周六 3000猴年六月十九 庚申年癸未月癸巳日
3000-07-13 周日 3000猴年六月二十 庚申年癸未月甲午日
3000-07-14 周一 3000猴年六月廿一 庚申年癸未月乙未日
3000-07-15 周二 3000猴年六月廿二 庚申年癸未月丙申日
3000-07-16 周三 3000猴年六月廿三 庚申年癸未月丁酉日
3000-07-17 周四 3000猴年六月廿四 庚申年癸未月戊戌日
3000-07-18 周五 3000猴年六月廿五 庚申年癸未月己亥日
3000-07-19 周六 3000猴年六月廿六 庚申年癸未月庚子日
3000-07-20 周日 3000猴年六月廿七 庚申年癸未月辛丑日
3000-07-21 周一 3000猴年六月廿八 庚申年癸未月壬寅日
3000-07-22 周二 3000猴年六月廿九 庚申年癸未月癸卯日 大暑 3000-07-22 09:08:09
3000-07-23 周三 3000猴年闰六月初一 庚申年癸未月甲辰日
3000-07-24 周四 3000猴年闰六月初二 庚申年癸未月乙巳日
3000-07-25 周五 3000猴年闰六月初三 庚申年癸未月丙午日
3000-07-26 周六 3000猴年闰六月初四 庚申年癸未月丁未日
3000-07-27 周日 3000猴年闰六月初五 庚申年癸未月戊申日
3000-07-28 周一 3000猴年闰六月初六 庚申年癸未月己酉日
3000-07-29 周二 3000猴年闰六月初七 庚申年癸未月庚戌日
3000-07-30 周三 3000猴年闰六月初八 庚申年癸未月辛亥日
3000-07-31 周四 3000猴年闰六月初九 庚申年癸未月壬子日
3000-08-01 周五 3000猴年闰六月初十 庚申年癸未月癸丑日
3000-08-02 周六 3000猴年闰六月十一 庚申年癸未月甲寅日
3000-08-03 周日 3000猴年闰六月十二 庚申年癸未月乙卯日
3000-08-04 周一 3000猴年闰六月十三 庚申年癸未月丙辰日
3000-08-05 周二 3000猴年闰六月十四 庚申年癸未月丁巳日
3000-08-06 周三 3000猴年闰六月十五 庚申年癸未月戊午日
3000-08-07 周四 3000猴年闰六月十六 庚申年癸未月己未日 立秋 3000-08-07 02:14:52
3000-08-08 周五 3000猴年闰六月十七 庚申年甲申月庚申日
3000-08-09 周六 3000猴年闰六月十八 庚申年甲申月辛酉日
3000-08-10 周日 3000猴年闰六月十九 庚申年甲申月壬戌日
3000-08-11 周一 3000猴年闰六月二十 庚申年甲申月癸亥日
3000-08-12 周二 3000猴年闰六月廿一 庚申年甲申月甲子日
3000-08-13 周三 3000猴年闰六月廿二 庚申年甲申月乙丑日
3000-08-14 周四 3000猴年闰六月廿三 庚申年甲申月丙寅日
3000-08-15 周五 3000猴年闰六月廿四 庚申年甲申月丁卯日
3000-08-16 周六 3000猴年闰六月廿五 庚申年甲申月戊辰日
3000-08-17 周日 3000猴年闰六月廿六 庚申年甲申月己巳日
3000-08-18 周一 3000猴年闰六月廿七 庚申年甲申月庚午日
3000-08-19 周二 3000猴年闰六月廿八 庚申年甲申月辛未日
3000-08-20 周三 3000猴年闰六月廿九 庚申年甲申月壬申日
3000-08-21 周四 3000猴年闰六月三十 庚申年甲申月癸酉日
3000-08-22 周五 3000猴年七月初一 庚申年甲申月甲戌日 处暑 3000-08-22 18:32:49
3000-08-23 周六 3000猴年七月初二 庚申年甲申月乙亥日
3000-08-24 周日 3000猴年七月初三 庚申年甲申月丙子日
3000-08-25 周一 3000猴年七月初四 庚申年甲申月丁丑日
3000-08-26 周二 3000猴年七月初五 庚申年甲申月戊寅日
3000-08-27 周三 3000猴年七月初六 庚申年甲申月己卯日
3000-08-28 周四 3000猴年七月初七 庚申年甲申月庚辰日
3000-08-29 周五 3000猴年七月初八 庚申年甲申月辛巳日
3000-08-30 周六 3000猴年七月初九 庚申年甲申月壬午日
3000-08-31 周日 3000猴年七月初十 庚申年甲申月癸未日
3000-09-01 周一 3000猴年七月十一 庚申年甲申月甲申日
3000-09-02 周二 3000猴年七月十二 庚申年甲申月乙酉日
3000-09-03 周三 3000猴年七月十三 庚申年甲申月丙戌日
3000-09-04 周四 3000猴年七月十四 庚申年甲申月丁亥日
3000-09-05 周五 3000猴年七月十五 庚申年甲申月戊子日
3000-09-06 周六 3000猴年七月十六 庚申年甲申月己丑日
3000-09-07 周日 3000猴年七月十七 庚申年甲申月庚寅日 白露 3000-09-07 09:15:15
3000-09-08 周一 3000猴年七月十八 庚申年乙酉月辛卯日
3000-09-09 周二 3000猴年七月十九 庚申年乙酉月壬辰日
3000-09-10 周三 3000猴年七月二十 庚申年乙酉月癸巳日
3000-09-11 周四 3000猴年七月廿一 庚申年乙酉月甲午日
3000-09-12 周五 3000猴年七月廿二 庚申年乙酉月乙未日
3000-09-13 周六 3000猴年七月廿三 庚申年乙酉月丙申日
3000-09-14 周日 3000猴年七月廿四 庚申年乙酉月丁酉日
3000-09-15 周一 3000猴年七月廿五 庚申年乙酉月戊戌日
3000-09-16 周二 3000猴年七月廿六 庚申年乙酉月己亥日
3000-09-17 周三 3000猴年七月廿七 庚申年乙酉月庚子日
3000-09-18 周四 3000猴年七月廿八 庚申年乙酉月辛丑日
3000-09-19 周五 3000猴年七月廿九 庚申年乙酉月壬寅日
3000-09-20 周六 3000猴年八月初一 庚申年乙酉月癸卯日
3000-09-21 周日 3000猴年八月初二 庚申年乙酉月甲辰日
3000-09-22 周一 3000猴年八月初三 庚申年乙酉月乙巳日 秋分 3000-09-22 21:39:17
3000-09-23 周二 3000猴年八月初四 庚申年乙酉月丙午日
3000-09-24 周三 3000猴年八月初五 庚申年乙酉月丁未日
3000-09-25 周四 3000猴年八月初六 庚申年乙酉月戊申日
3000-09-26 周五 3000猴年八月初七 庚申年乙酉月己酉日
3000-09-27 周六 3000猴年八月初八 庚申年乙酉月庚戌日
3000-09-28 周日 3000猴年八月初九 庚申年乙酉月辛亥日
3000-09-29 周一 3000猴年八月初十 庚申年乙酉月壬子日
3000-09-30 周二 3000猴年八月十一 庚申年乙酉月癸丑日
3000-10-01 周三 3000猴年八月十二 庚申年乙酉月甲寅日
3000-10-02 周四 3000猴年八月十三 庚申年乙酉月乙卯日
3000-10-03 周五 3000猴年八月十四 庚申年乙酉月丙辰日
3000-10-04 周六 3000猴年八月十五 庚申年乙酉月丁巳日
3000-10-05 周日 3000猴年八月十六 庚申年乙酉月戊午日
3000-10-06 周一 3000猴年八月十七 庚申年乙酉月己未日
3000-10-07 周二 3000猴年八月十八 庚申年乙酉月庚申日
3000-10-08 周三 3000猴年八月十九 庚申年乙酉月辛酉日 寒露 3000-10-08 07:23:23
3000-10-09 周四 3000猴年八月二十 庚申年丙戌月壬戌日
3000-10-10 周五 3000猴年八月廿一 庚申年丙戌月癸亥日
3000-10-11 周六 3000猴年八月廿二 庚申年丙戌月甲子日
3000-10-12 周日 3000猴年八月廿三 庚申年丙戌月乙丑日
3000-10-13 周一 3000猴年八月廿四 庚申年丙戌月丙寅日
3000-10-14 周二 3000猴年八月廿五 庚申年丙戌月丁卯日
3000-10-15 周三 3000猴年八月廿六 庚申年丙戌月戊辰日
3000-10-16 周四 3000猴年八月廿七 庚申年丙戌月己巳日
3000-10-17 周五 3000猴年八月廿八 庚申年丙戌月庚午日
3000-10-18 周六 3000猴年八月廿九 庚申年丙戌月辛未日
3000-10-19 周日 3000猴年九月初一 庚申年丙戌月壬申日
3000-10-20 周一 3000猴年九月初二 庚申年丙戌月癸酉日
3000-10-21 周二 3000猴年九月初三 庚申年丙戌月甲戌日
3000-10-22 周三 3000猴年九月初四 庚申年丙戌月乙亥日
3000-10-23 周四 3000猴年九月初五 庚申年丙戌月丙子日 霜降 3000-10-23 14:00:19
3000-10-24 周五 3000猴年九月初六 庚申年丙戌月丁丑日
3000-10-25 周六 3000猴年九月初七 庚申年丙戌月戊寅日
3000-10-26 周日 3000猴年九月初八 庚申年丙戌月己卯日
3000-10-27 周一 3000猴年九月初九 庚申年丙戌月庚辰日
3000-10-28 周二 3000猴年九月初十 庚申年丙戌月辛巳日
3000-10-29 周三 3000猴年九月十一 庚申年丙戌月壬午日
3000-10-30 周四 3000猴年九月十二 庚申年丙戌月癸未日
3000-10-31 周五 3000猴年九月十三 庚申年丙戌月甲申日
3000-11-01 周六 3000猴年九月十四 庚申年丙戌月乙酉日
3000-11-02 周日 3000猴年九月十五 庚申年丙戌月丙戌日
3000-11-03 周一 3000猴年九月十六 庚申年丙戌月丁亥日
3000-11-04 周二 3000猴年九月十七 庚申年丙戌月戊子日
3000-11-05 周三 3000猴年九月十八 庚申年丙戌月己丑日
3000-11-06 周四 3000猴年九月十九 庚申年丙戌月庚寅日
3000-11-07 周五 3000猴年九月二十 庚申年丙戌月辛卯日 立冬 3000-11-07 17:38:30
3000-11-08 周六 3000猴年九月廿一 庚申年丁亥月壬辰日
3000-11-09 周日 3000猴年九月廿二 庚申年丁亥月癸巳日
3000-11-10 周一 3000猴年九月廿三 庚申年丁亥月甲午日
3000-11-11 周二 3000猴年九月廿四 庚申年丁亥月乙未日
3000-11-12 周三 3000猴年九月廿五 庚申年丁亥月丙申日
3000-11-13 周四 3000猴年九月廿六 庚申年丁亥月丁酉日
3000-11-14 周五 3000猴年九月廿七 庚申年丁亥月戊戌日
3000-11-15 周六 3000猴年九月廿八 庚申年丁亥月己亥日
3000-11-16 周日 3000猴年九月廿九 庚申年丁亥月庚子日
3000-11-17 周一 3000猴年九月三十 庚申年丁亥月辛丑日
3000-11-18 周二 3000猴年十月初一 庚申年丁亥月壬寅日
3000-11-19 周三 3000猴年十月初二 庚申年丁亥月癸卯日
3000-11-20 周四 3000猴年十月初三 庚申年丁亥月甲辰日
3000-11-21 周五 3000猴年十月初四 庚申年丁亥月乙巳日
3000-11-22 周六 3000猴年十月初五 庚申年丁亥月丙午日 小雪 3000-11-22 18:12:26
3000-11-23 周日 3000猴年十月初六 庚申年丁亥月丁未日
3000-11-24 周一 3000猴年十月初七 庚申年丁亥月戊申日
3000-11-25 周二 3000猴年十月初八 庚申年丁亥月己酉日
3000-11-26 周三 3000猴年十月初九 庚申年丁亥月庚戌日
3000-11-27 周四 3000猴年十月初十 庚申年丁亥月辛亥日
3000-11-28 周五 3000猴年十月十一 庚申年丁亥月壬子日
3000-11-29 周六 3000猴年十月十二 庚申年丁亥月癸丑日
3000-11-30 周日 3000猴年十月十三 庚申年丁亥月甲寅日
3000-12-01 周一 3000猴年十月十四 庚申年丁亥月乙卯日
3000-12-02 周二 3000猴年十月十五 庚申年丁亥月丙辰日
3000-12-03 周三 3000猴年十月十六 庚申年丁亥月丁巳日
3000-12-04 周四 3000猴年十月十七 庚申年丁亥月戊午日
3000-12-05 周五 3000猴年十月十八 庚申年丁亥月己未日
3000-12-06 周六 3000猴年十月十九 庚申年丁亥月庚申日
3000-12-07 周日 3000猴年十月二十 庚申年丁亥月辛酉日 大雪 3000-12-07 16:17:06
3000-12-08 周一 3000猴年十月廿一 庚申年戊子月壬戌日
3000-12-09 周二 3000猴年十月廿二 庚申年戊子月癸亥日
3000-12-10 周三 3000猴年十月廿三 庚申年戊子月甲子日
3000-12-11 周四 3000猴年十月廿四 庚申年戊子月乙丑日
3000-12-12 周五 3000猴年十月廿五 庚申年戊子月丙寅日
3000-12-13 周六 3000猴年十月廿六 庚申年戊子月丁卯日
3000-12-14 周日 3000猴年十月廿七 庚申年戊子月戊辰日
3000-12-15 周一 3000猴年十月廿八 庚申年戊子月己巳日
3000-12-16 周二 3000猴年十月廿九 庚申年戊子月庚午日
3000-12-17 周三 3000猴年十月三十 庚申年戊子月辛未日
3000-12-18 周四 3000猴年冬月初一 庚申年戊子月壬申日
3000-12-19 周五 3000猴年冬月初二 庚申年戊子月癸酉日
3000-12-20 周六 3000猴年冬月初三 庚申年戊子月甲戌日
3000-12-21 周日 3000猴年冬月初四 庚申年戊子月乙亥日
3000-12-22 周一 3000猴年冬月初五 庚申年戊子月丙子日 冬至 3000-12-22 12:07:10
3000-12-23 周二 3000猴年冬月初六 庚申年戊子月丁丑日
3000-12-24 周三 3000猴年冬月初七 庚申年戊子月戊寅日
3000-12-25 周四 3000猴年冬月初八 庚申年戊子月己卯日
3000-12-26 周五 3000猴年冬月初九 庚申年戊子月庚辰日
3000-12-27 周六 3000猴年冬月初十 庚申年戊子月辛巳日
3000-12-28 周日 3000猴年冬月十一 庚申年戊子月壬午日
3000-12-29 周一 3000猴年冬月十二 庚申年戊子月癸未日
3000-12-30 周二 3000猴年冬月十三 庚申年戊子月甲申日
3000-12-31 周三 3000猴年冬月十四 庚申年戊子月乙酉日
```

# 计算节气时间间隔

```
# solarterm/solarterm_test.go:305
2025年 [立春]~[雨水] 相差 14.830625 天(1281366秒) [2025-02-03 22:10:12]~[2025-02-18 18:06:18]
2025年 [雨水]~[惊蛰] 相差 14.917176 天(1288844秒) [2025-02-18 18:06:18]~[2025-03-05 16:07:02]
2025年 [惊蛰]~[春分] 相差 15.037627 天(1299251秒) [2025-03-05 16:07:02]~[2025-03-20 17:01:13]
2025年 [春分]~[清明] 相差 15.157720 天(1309627秒) [2025-03-20 17:01:13]~[2025-04-04 20:48:20]
2025年 [清明]~[谷雨] 相差 15.296817 天(1321645秒) [2025-04-04 20:48:20]~[2025-04-20 03:55:45]
2025年 [谷雨]~[立夏] 相差 15.417500 天(1332072秒) [2025-04-20 03:55:45]~[2025-05-05 13:56:57]
2025年 [立夏]~[小满] 相差 15.539873 天(1342645秒) [2025-05-05 13:56:57]~[2025-05-21 02:54:22]
2025年 [小满]~[芒种] 相差 15.626319 天(1350114秒) [2025-05-21 02:54:22]~[2025-06-05 17:56:16]
2025年 [芒种]~[夏至] 相差 15.698414 天(1356343秒) [2025-06-05 17:56:16]~[2025-06-21 10:41:59]
2025年 [夏至]~[小暑] 相差 15.724120 天(1358564秒) [2025-06-21 10:41:59]~[2025-07-07 04:04:43]
2025年 [小暑]~[大暑] 相差 15.725312 天(1358667秒) [2025-07-07 04:04:43]~[2025-07-22 21:29:10]
2025年 [大暑]~[立秋] 相差 15.682037 天(1354928秒) [2025-07-22 21:29:10]~[2025-08-07 13:51:18]
2025年 [立秋]~[处暑] 相差 15.612697 天(1348937秒) [2025-08-07 13:51:18]~[2025-08-23 04:33:35]
2025年 [处暑]~[白露] 相差 15.512569 天(1340286秒) [2025-08-23 04:33:35]~[2025-09-07 16:51:41]
2025年 [白露]~[秋分] 相差 15.394016 天(1330043秒) [2025-09-07 16:51:41]~[2025-09-23 02:19:04]
2025年 [秋分]~[寒露] 相差 15.265185 天(1318912秒) [2025-09-23 02:19:04]~[2025-10-08 08:40:56]
2025年 [寒露]~[霜降] 相差 15.131748 天(1307383秒) [2025-10-08 08:40:56]~[2025-10-23 11:50:39]
2025年 [霜降]~[立冬] 相差 15.009120 天(1296788秒) [2025-10-23 11:50:39]~[2025-11-07 12:03:47]
2025年 [立冬]~[小雪] 相差 14.896887 天(1287091秒) [2025-11-07 12:03:47]~[2025-11-22 09:35:18]
2025年 [小雪]~[大雪] 相差 14.811817 天(1279741秒) [2025-11-22 09:35:18]~[2025-12-07 05:04:19]
2025年 [大雪]~[冬至] 相差 14.748947 天(1274309秒) [2025-12-07 05:04:19]~[2025-12-21 23:02:48]
2025年 [冬至]~[小寒] 相差 14.722280 天(1272005秒) [2025-12-21 23:02:48]~[2026-01-05 16:22:53]
2025年 [小寒]~[大寒] 相差 14.723449 天(1272106秒) [2026-01-05 16:22:53]~[2026-01-20 09:44:39]
```

# TODO

1. ~~完善单元测试~~
2. ~~完善注释~~
3. ~~完善文档~~
4. ~~支持更大范围的时间~~
5. ~~把农历的算法实现一下~~(我在2025年实现了!时隔7年!)
6. ~~支持 go module~~

# 参考资料

- [算法系列之十八：用天文方法计算二十四节气（上）](https://blog.csdn.net/orbit/article/details/7910220)
- [算法系列之十八：用天文方法计算二十四节气（下）](https://blog.csdn.net/orbit/article/details/7944248)
- [overtrue/chinese-calendar](https://github.com/overtrue/chinese-calendar)
- [1900年至2100年公历、农历互转Js代码 - 晶晶的博客](http://blog.jjonline.cn/userInterFace/173.html)
- [香港天文台](http://data.weather.gov.hk/)
- [五虎遁元](https://baike.baidu.com/item/%E4%BA%94%E8%99%8E%E9%81%81%E5%85%83/5471492)
- [五鼠遁元](https://baike.baidu.com/item/%E4%BA%94%E9%BC%A0%E9%81%81%E5%85%83/5471935)
- [NASA](https://eclipse.gsfc.nasa.gov/)

# License

MIT


# Stargazers over time

[![Stargazers over time](https://starchart.cc/Lofanmi/chinese-calendar-golang.svg)](https://starchart.cc/Lofanmi/chinese-calendar-golang)
