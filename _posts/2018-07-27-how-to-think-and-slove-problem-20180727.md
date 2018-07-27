---
layout: post
title: 如何思考并编程解决一个常规问题？
categories: 解题的套路 
tags: python 闰年 日期 时间 软件工程 测试用例 测试 自动化测试 设计 
---

在业务软件的开发中，其实遇到的大部分业务需求场景都可以用常规的逻辑思考和解决，很少会遇到复杂的逻辑、算法、架构问题，这些都是比较偏底层的设计与实现的时候需要考虑的

那么面对大多数的业务需求、业务场景，我们应该怎么思考、怎么设计、怎么解决、怎么验证？

比如在这样一个真实的业务场景中，客户可以在任意一天订购我们的服务，订购周期可以是一个月、一个季度、半年或一年，在订购日后的这么多月后的凌晨 0 点订单到期

以下举例说明订购的过期计算方式：

* 在 2016-11-10 订购一个月，订购将在 2016-12-10 过期
* 在 2016-12-10 订购一个月，订购将在 2017-01-10 过期
* 在 2017-01-30 订购一个月，订购将在 2017-02-28 过期
* 在 2017-05-31 订购三个月，订购将在 2017-08-31 过期

## 哪天的到期订单最多？

假定我们每天可以收到4 个订单，其中一个是一个月周期的，一个是一个季度周期的，一个是半年周期的，一个是一年周期的。这样的订单从很久以前，比如五年前就开始了

请通过分析回答：在哪一天到期的订单会最多，有多少个？具体是哪几个订单？

分析过程如下：

>今天是2018-07-27，那么五年前就是2013-07-27

>从2013-07-27 开始下单，周期五年，那么最早一天有到期订单的是2013-08-27（2013-07-27 下的一个月周期的订单），最晚一天有到期订单的是 2019-07-27（2018-07-27 下的一年周期的订单）

>假定不考虑闰年、大小月，一个月固定30天，一季度固定90天，半年固定180天，一年固定360天，那么距离今天的天数是30、90、180、360 的最小公倍数，即360 倍数的天数会有4 个到期订单。以此类推，180 倍数（不包括360）的天数有3 个到期订单、90 倍数（不包括180、360）的天数有2 个到期订单、30 倍数（不包括360、180、90）的天数有1 个到期订单

>具体以2017-07-27 这天来说，有2017-06-27 下的一个月到期的订单、2017-04-27 下的三个月到期的订单、2017-01-27 下的六个月到期的订单、2016-07-27 下的十二个月到期的订单，所以2017-07-27 一共有四个到期订单

>如果是像上面分析的月份是均匀分布的，那么所有距今天数为360 倍数的日期有最多的到期订单数，也就是 4个

>但事实上月份并不是均匀分布的，1、3、5、7、8、10、12 月份是31 天；4、6、9、11 月份是30 天；2 月份闰年是 29天，非闰年是28天

>按照上面的说法，如果像2018-01-30 订购一个月，那么因为2月没有30 天，所以到 2018-02-28到期，所以2018-02-28 不光有2018-01-28、2017-11-28、2017-08-28、2017-02-28 时对应的订单，还有2018-01-29、2018-01-30、2018-01-31、2017-11-28、2017-11-29、2017-11-30、2017-08-28、2017-08-29、2017-08-30、2017-08-31 的订单

>如果上一年是闰年，还会多出一个当年的 02-29 下的一年单 到今年的 02-28

>2014、2015、2016、2017、2018、2019，上年是闰年的只有2017 年（上年是2016 年）

>所以在2017-02-28 那天到期的订单数最多，包括：

>2017-01-28、2017-01-29、2017-01-30、2017-01-31 下的一个月到期的订单

>2016-11-28、2016-11-29、2016-11-30 下的三个月到期的订单

>2016-08-28、2016-08-29、2016-08-30、2016-08-31 下的六个月到期的订单

>2016-02-28、2016-02-29 下的十二个月到期的订单

>一共13 个

## 求到期日

请实现一个方法，给定订购的年月日（year, month, day)，已知订购时长是一个月，返回订单的到期日。year, month, day 均为整数，保证是正确的日期

要求函数的接口是这样的

```python
def getExpirationDate(year, month, day):
    next_year = 0
    next_month = 0
    next_day = 0

    # 后续完善逻辑

    return [next_year, next_month, next_day]
```

**首先去分析所有可能的场景**

1) 考虑输入数据是否合法的场景，使用者正常都会传入正确的日期，但不能保证其不传入非法的日期，比如传入2018.100.20、2018.-1.33 这样的，那么就需要在程序中先检查输入是否正确

因此设计一个异常类，以及一个检查输入的函数

```python
def dateToStr(date):
    return str(date[0]) + '.' + str(date[1]) + '.' + str(date[2])

class LegalDateException(Exception):
    def __init__(self, year, month, day):
        error = 'Legal Date: ' + dateToStr([year, month, day])
        Exception.__init__(self, error)

def judgeLegalDate(year, month, day):
    if (year <= 0) or (month <= 0) or (day <= 0):
        raise LegalDateException(year, month, day)
    # 其他非法情况看后续代码实现
```

2) 考虑一些特殊的日期

* 1、3、5、7、8、10、12 每个月31 天
* 4、6、9、11 每个月30 天
* 2 月，闰年29 天，非闰年28 天
* 跨年的情况，从去年的12 月到今年的1 月
    * 如果现在是12 月，那么 next_year = year+1; next_month = 1
    * 如果现在不是12 月，那么 next_year = year; next_month = month + 1

所以可能就要设计判断闰年的函数、针对31/30/29/28 天的月份进行分别处理、考虑跨年的情况

```python
def isLeapYear(year):
    pass

def getExpirationDate(year, month, day):
    judgeLegalDate(year, month, day)

    next_year = 0
    next_month = 0
    next_day = 0

    # 后续完善逻辑

    return [next_year, next_month, next_day]
```

**设计测试用例**

接下来就是在很对上面考虑到的可能场景，设计测试用例

简单的编码就是这样的

```python
# simple test function
def testGetExpirationDate(date, expect_date, is_exception = False):
    try:
        year = date[0]
        month = date[1]
        day = date[2]
        next_date = getExpirationDate(year, month, day)
        if (next_date == expect_date):
            print('[Success]   ' + dateToStr(date) + ' -> ' + dateToStr(next_date))
        else:
            print('[Error]     ' + dateToStr(date) + ' -> ' + dateToStr(next_date) + ', not ' + dateToStr(expect_date))
    except LegalDateException as e:
        if (is_exception):
            print('[Success]   ' + str(e))
        else:
            print('[Exception] ' + str(e))


if (__name__ == '__main__'):
    # 非法日期
    testGetExpirationDate([2000, -2, 10], [0, 0, 0], True)
    testGetExpirationDate([-2000, 2, 10], [0, 0, 0], True)
    testGetExpirationDate([2000, 2, -10], [0, 0, 0], True)
    testGetExpirationDate([2000, 2, 30], [0, 0, 0], True)
    testGetExpirationDate([2001, 2, 29], [0, 0, 0], True)
    testGetExpirationDate([2001, 2, 29], [0, 0, 0], True)

    # 普通日期
    testGetExpirationDate([2018, 1, 10], [2018, 2, 10])
    # 闰年 1->2 月
    testGetExpirationDate([2000, 1, 31], [2000, 2, 29])
    # 非闰年 1->2 月
    testGetExpirationDate([2001, 1, 31], [2001, 2, 28])
    # 跨年 12 -> 1 月
    testGetExpirationDate([2000, 12, 31], [2001, 1, 31])
    # 跨年 12 -> 1 月
    testGetExpirationDate([2000, 12, 20], [2001, 1, 20])
    # 本月31 -> 下月30
    testGetExpirationDate([2000, 3, 31], [2000, 4, 30])
```

**现在开始完成编码实现**

经过上面的场景分析、简单解耦、设计测试用例等步骤之后，下面给出完整的代码实现！

```python
def dateToStr(date):
    return str(date[0]) + '.' + str(date[1]) + '.' + str(date[2])

def isLeapYear(year):
    if (year % 400 == 0) or ((year % 4 == 0) and (year % 100 != 0)):
        return True
    else:
        return False


class LegalDateException(Exception):
    def __init__(self, year, month, day):
        error = 'Legal Date: ' + dateToStr([year, month, day])
        Exception.__init__(self, error)

def judgeLegalDate(year, month, day):
    if (year <= 0) or (month <= 0) or (day <= 0):
        raise LegalDateException(year, month, day)
    if (month > 12):
        raise LegalDateException(year, month, day)
    if (month in [1, 3, 5, 7, 8, 10, 12]) and (day > 31):
        raise LegalDateException(year, month, day)
    if (month in [4, 6, 9, 11]) and (day > 30):
        raise LegalDateException(year, month, day)
    if (2 == month):
        if (isLeapYear(year)) and (day > 29):
            raise LegalDateException(year, month, day)
        if (not isLeapYear(year)) and (day > 28):
            raise LegalDateException(year, month, day)


def getExpirationDate(year, month, day):
    judgeLegalDate(year, month, day)

    next_year = 0
    next_month = 0
    next_day = 0

    # year and month
    next_year = year
    next_month = month + 1
    if 12 == month:
        next_year = year + 1
        next_month = 1

    # day
    next_day = day
    if (month in [3, 5, 7, 8, 10]) and (31 == day):
        next_day = 30
    elif (1 == month):
        if (isLeapYear(year)) and (day >= 29):
            next_day = 29
        elif (not isLeapYear(year)) and (day >= 28):
            next_day = 28

    return [next_year, next_month, next_day]


# simple test function
def testGetExpirationDate(date, expect_date, is_exception = False):
    try:
        year = date[0]
        month = date[1]
        day = date[2]
        next_date = getExpirationDate(year, month, day)
        if (next_date == expect_date):
            print('[Success]   ' + dateToStr(date) + ' -> ' + dateToStr(next_date))
        else:
            print('[Error]     ' + dateToStr(date) + ' -> ' + dateToStr(next_date) + ', not ' + dateToStr(expect_date))
    except LegalDateException as e:
        if (is_exception):
            print('[Success]   ' + str(e))
        else:
            print('[Exception] ' + str(e))


if (__name__ == '__main__'):
    # 非法日期
    testGetExpirationDate([2000, -2, 10], [0, 0, 0], True)
    testGetExpirationDate([-2000, 2, 10], [0, 0, 0], True)
    testGetExpirationDate([2000, 2, -10], [0, 0, 0], True)
    testGetExpirationDate([2000, 2, 30], [0, 0, 0], True)
    testGetExpirationDate([2001, 2, 29], [0, 0, 0], True)
    testGetExpirationDate([2001, 2, 29], [0, 0, 0], True)

    # 普通日期
    testGetExpirationDate([2018, 1, 10], [2018, 2, 10])
    # 闰年 1->2 月
    testGetExpirationDate([2000, 1, 31], [2000, 2, 29])
    # 非闰年 1->2 月
    testGetExpirationDate([2001, 1, 31], [2001, 2, 28])
    # 跨年 12 -> 1 月
    testGetExpirationDate([2000, 12, 31], [2001, 1, 31])
    # 跨年 12 -> 1 月
    testGetExpirationDate([2000, 12, 20], [2001, 1, 20])
    # 本月31 -> 下月30
    testGetExpirationDate([2000, 3, 31], [2000, 4, 30])
```

## 简单总结

以上通过一个极其简单的例子、简单的业务需求，详细列出思考到解决这个问题的步骤

* 思考所有可能出现的场景：正常场景、异常场景
* 针对以上列举出的所有可能场景，逐个思考对应的处理方案
* 针对以上列举出的所有可能场景，分别设计一或多个测试用例
* 编写代码实现逻辑
* 执行测试用例，检查代码逻辑是否正确
* 上面没有提到这样几点，也很重要
    * code review，看看哪些公共逻辑可以解耦出来作为单独的函数或类……
    * 扩展性，以上只要求实现获取一个月后的日期，那如果要求扩展功能，获取2、3、4个月……之后的日期呢？

对于平时遇到的很多平常问题都可以通过这个套路去搞定

当然本文只是我闲来无事的瞎想而已，无甚大意义、无甚技术含量！只是提供一个思考和解决常规问题的套路而已！
