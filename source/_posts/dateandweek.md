title: iOS开发tips之日期与星期的对应    
date: 2016-03-03 11:47:11    
category: iOS    
tags:

---

## 前言
前段时间练手模拟校园app写了个Demo，其中在写课程表的界面时，加上了当前日期，为了让日期与星期对应也是前后折腾了好几次，主要还是对日期的情况思考的不够全面吧（比如判断每月天数和月初月末的交替），不过最终还是完成需求。最后意外发现简便方法，写下来分享。    
* 对校园app的Demo有兴趣的话，请移驾GitHub:  [QLU-BlogDemo](https://github.com/gonghonglou/QLU-BlogDemo)

## 正文
首先看一下模样吧（如图中红框所示）
![课程表](http://7xn9bi.com1.z0.glb.clouddn.com/dateandweek%2Fdateweek.png)
最基本的：获取当前日期的方法

```
// 获取当前时间
NSDate *senddate = [NSDate date];
NSDateFormatter *dateformatter = [[NSDateFormatter alloc] init];
[dateformatter setDateFormat:@"yyy"];
NSString *yearString = [dateformatter stringFromDate:senddate];
[dateformatter setDateFormat:@"MM"];
NSString *monthString = [dateformatter stringFromDate:senddate];
[dateformatter setDateFormat:@"dd"];
NSString *dayString = [dateformatter stringFromDate:senddate];
[dateformatter setDateFormat:@"EEE"];
    
NSString *weekString = [dateformatter stringFromDate:senddate];
NSLog(@"-%@",weekString);
int year = [yearString intValue];
NSLog(@"-%d", year);
int month = [monthString intValue];
NSLog(@"--%d", month);
int day = [dayString intValue];
NSLog(@"---%d", day);
```

看打印结果：
![打印结果](http://7xn9bi.com1.z0.glb.clouddn.com/dateandweek%2F985173.png)
然后就是把获取到的日期放到对应星期上去了,需要格外注意的要判断的几点：
* 不同月的天数
* 月底时，显示下个月月初的日期
* 月初时，显示上个月月底的日期

首先获取不同月的天数,并抽离成类方法方便后边调用

```
// 获取某年某月总共多少天     
+ (int)getDaysInMonth:(int)year month:(int)imonth {
	// imonth == 0的情况是应对在CourseViewController里month-1的情况
	if((imonth == 0)||(imonth == 1)||(imonth == 3)||(imonth == 5)||(imonth == 7)||(imonth == 8)||(imonth == 10)||(imonth == 12))
		return 31;
	if((imonth == 4)||(imonth == 6)||(imonth == 9)||(imonth == 11))
		return 30;
	if((year%4 == 1)||(year%4 == 2)||(year%4 == 3))
		return 28;
	if(year%400 == 0)
		return 29;
	if(year%100 == 0)
		return 28;	
	return 29;    
}
```

放置label的方法，在label上写下日期

```
// 添加日期
- (void)setDateLabel:(CGRect)rect number:(NSString *)number{
	UILabel *label = [[UILabel alloc] initWithFrame:rect];
	label.textColor = [UIColor colorWithRed:100.0/255 green:	73.0/255 blue:250.0/255 alpha:1];
	label.textAlignment = NSTextAlignmentCenter;
	label.text = number;
	label.font = [UIFont systemFontOfSize:14];
	[self.view addSubview:label];
}
```
 
开始判断。拿到周一是几号后面的日数递增就OK了，注意月末月初交替的特殊情况（看机智如我，走起)

```
// 判断当前天是周几，从而计算出当周的周一是几号（负数表示上个月月末）
if ([weekString  isEqual: @"周一"]) {
    day = day - 1; // 因为下面有 day++;
} else if ([weekString isEqual:@"周二"]) {
    day = day - 2;
} else if ([weekString isEqual:@"周三"]) {
    day = day - 3;
} else if ([weekString isEqual:@"周四"]) {
    day = day - 4;
} else if ([weekString isEqual:@"周五"]) {
    day = day - 5;
} else if ([weekString isEqual:@"周六"]) {
    day = day - 6;
} else if ([weekString isEqual:@"周日"]) {
    day = day - 7;
}
    
if (day<0) { // 月初时显示上个月月末的日期
    for (int i = 0; i < 7; i++) {
        // 上个月末往后的月初数字
        int days = [GetDays getDaysInMonth:year month:month-1];
        day++;
        if (day > days) {
            day = 0;
            day++;
        }
        // 月初之前的月末数字
        switch (day) {
            case 0: day = days; break;
            case -1: day = days-1; break;
            case -2: day = days-2; break;
            case -3: day = days-3; break;
            case -4: day = days-4; break;
            case -5: day = days-5; break;
            default: break;
        }
        [self setDateLabel:CGRectMake(i*((self.view.frame.size.width-33.5)/7+0.5)+30.5, kTopY, (self.view.frame.size.width-33.5)/7, 15) number:[NSString stringWithFormat:@"%d", day]];
    }
} else { // 月末
    for (int i = 0; i < 7; i++) {
        int days = [GetDays getDaysInMonth:year month:month];
        day++;
        if (day > days) {
            day = 0;
            day++;
        }
        [self setDateLabel:CGRectMake(i*((self.view.frame.size.width-33.5)/7+0.5)+30.5, kTopY, (self.view.frame.size.width-33.5)/7, 15) number:[NSString stringWithFormat:@"%d", day]];
    }
}
```
劳心劳力，大功告成。接下来小L要告诉大家，上边的判断看看就好了（哭吧）我们还有so so so easy的方法（嘿）

方法推荐：获取几年几月几日后的日期,并抽离成类方法方便后边调用

```
/**
  *  获取几年几月几日后的日期，0表示当天，负数表示之前
  *  这里只要取到日就好了，年月置0，表示当年当月
  */
+ (int)getOneDay:(int)day {
    int year = 0, month = 0;
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    NSDateComponents *comps = nil;
    comps = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:[NSDate date]];
    NSDateComponents *adcomps = [[NSDateComponents alloc] init];
    [adcomps setYear:year];
    [adcomps setMonth:month];
    [adcomps setDay:day];
    NSDate *newdate = [calendar dateByAddingComponents:adcomps toDate:[NSDate date] options:0];
    
    NSDateFormatter  *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateFormat:@"dd"];
    NSString *  dayString = [formatter stringFromDate:newdate];
    return [dayString intValue];
}
```

重点来了：

```

// 判断当前天是周几，从而计算出当周的周一是几号（负数表示上个月月末）
if ([weekString  isEqual: @"周一"]) {
    day = 0; // 因为下面有 day++;
} else if ([weekString isEqual:@"周二"]) {
    day = -1;
} else if ([weekString isEqual:@"周三"]) {
    day = -2;
} else if ([weekString isEqual:@"周四"]) {
    day = -3;
} else if ([weekString isEqual:@"周五"]) {
    day = -4;
} else if ([weekString isEqual:@"周六"]) {
    day = -5;
} else if ([weekString isEqual:@"周日"]) {
    day = -6;
}
// 放置日期
for (int i = 0; i < 7; i++) {
	[self setDateLabel:CGRectMake(i*((self.view.frame.size.width-33.5)/7+0.5)+30.5, kTopY, (self.view.frame.size.width-33.5)/7, 15) number:[NSString stringWithFormat:@"%d", [GetOneDay getOneDay:day++]]];
}
```

如此才是万事大吉！
比上边的判断简单了许多，不过上边的判断也是一种思路，所以记录下来，给大家一个借鉴！

## 后记     
小白出手，请多指教。    
如言有误，还望斧正！

* 转载请保留原文地址：[http://gonghonglou.com/2016/03/03/dateandweek](http://gonghonglou.com/2016/03/03/dateandweek)
* 有兴趣的读者欢迎关注我的微博：[与佳期](http://weibo.com/gonghonglou)

**迁移文章 首发于2015-10-13**
