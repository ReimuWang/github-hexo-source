---
title: Java 基础-日期和时间
date: 2017-09-29 15:29:49
tags: [Java,日期,时间]
categories: Java 基础
---

简单来说，Java的时间API可归结为以下3个类：

- Date:时间API的核心类。最初很强大，几乎包含了时间API的所有功能。后来被拆分，只保留与核心long类型毫秒数相关的功能，其余功能均已标记为废弃方法。

- DateFormat:负责进行Date与格式化字符串之间的转换。主要用于将Date展示为各种人类易于阅读的格式，基本不承载具体的功能。

- Calendar:字面含义是日历类。是Java时间API中功能最强大的类。可以这么理解，除了Date及DateFormat所实现的那一点功能之外，其余的功能都在Calendar中。

# Date

一个java.util.Date类的实例表示一个精确的时刻，单位为毫秒。核心字段用long类型存储，记录自标准纪元1970年1月1日0时0分0秒起到该特定时刻的毫秒数。

若有如下代码：

```
Date date = new Date();
```

<!-- more -->

该无参构造函数的源码为：

```
public Date() {
    this(System.currentTimeMillis());
}
```

即默认以当前时刻创建了Date实例。当然我们也可以指定时刻：

```
Date date = new Date(666);
date.setTime(1000);
System.out.println(date.getTime());
System.out.println(date.toString());    // 当前系统环境(中国)时区
System.out.println(date.toGMTString());    // 默认时区(仅用于举例，已废弃，不推荐使用)
```

输出：

```
1000
Thu Jan 01 08:00:01 CST 1970
1 Jan 1970 00:00:01 GMT
```

时刻前后比较：

```
Date date1 = new Date(1);
Date date2 = new Date(2);
System.out.println(date1.before(date2));
System.out.println(date1.after(date2));
```

输出：

```
true
false
```

比较2：

```
Date date1 = new Date(1);
Date date2 = new Date(1);
System.out.println(date1.before(date2));
System.out.println(date1.after(date2));
System.out.println(date1.equals(date2));
```

输出：

```
false
false
true
```

实际上，Date的比较就是在比其内部的毫秒数的先后。

# DateFormat

利用java.text.DataFormat的子类SimpleDateFormat可将Date与特定格式的字符串进行相互转化：

- Date-->特定格式的字符串：format方法。

- 特定格式的字符串-->Date：parse方法。

**Date转字符串**

```
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test {

    public static void main(String[] args) {
        String dateFormat = new SimpleDateFormat("yyyy/MM/dd-HH:mm:ss").format(new Date());
        System.out.println(dateFormat);
    }
}
```

输出：

```
2017/09/29-15:06:03
```

其中"yyyy/MM/dd-HH:mm:ss"被称为格式化字符串，全字符含义如下图所示：

![0.jpg](/images/blog_pic/Java 基础/日期和时间/0.jpg)

---

**字符串转Date**

```
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test {

    public static void main(String[] args) throws ParseException {
        String dateStr = "1990年06月05日 07:07:07";
        DateFormat dateFormat1 = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        Date date = dateFormat1.parse(dateStr);
        System.out.println(date);
    }
}
```

输出：

```
Tue Jun 05 07:07:07 CDT 1990
```

# Calendar

在上文已叙述的两个类中，DateFormat只用于格式转换，不承载实际的逻辑。Date中的时间是以1个long类型的整数存储的。这种存储格式是给计算机看的，更关心时间流逝的本质，因此基本计数单位也仅采用毫秒一种，人类几乎无法理解。

为扩展时间API的功能，设计者又添加了日历类Calendar，表示日期的概念(其实也能精确到时分秒)，较之Date类，其更关心人类在历法上的逻辑。功能为进行某一特定时刻与各种日期(年月周日时分秒)之间的转化。

需要注意的是，Calendar这一族的类中的月份是从0开始的，即月份的取值范围为[0,11](欧美国家是不采用数字表示月份的，因此从0开始他们并不在乎。中国人用数字表示月份，因此从0开始看着会比较别扭)。而星期则是从1开始的，即周日是1，周一是2...周六是7。为了避免开发人员记忆过多这种纯规则性的东西，Calendar内部以不可变int型类变量的方式提供了全部日期类型：

```
public final static int YEAR = 1;
public final static int SUNDAY = 1;
public final static int FEBRUARY = 1;
...
```

这里我故意抽取了3个值一样的类变量。这是不会产生歧义的，因为它们3个的使用场景不同。

Calendar本身是抽象类，GregorianCalendar是Calendar的一个子类，表示世界上大部分国家及地区均采用的标准日历系统(即所谓的公历)。我们在日常开发中用到的也就是这个类。

```
import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;

public class Test {

    public static void main(String[] args) {
        Calendar calendar = new GregorianCalendar();
        calendar.set(1990, Calendar.JUNE, 5, 7, 7, 7);    // 1990年6月5日 07:07:07
        System.out.println(calendar.getTimeInMillis());    // 1970年1月1日0时0分0秒到calendar对应时刻的毫秒数
        Date date = calendar.getTime();    // Date类是时间API中各组件间的桥梁
        System.out.println(date);
    }
}
```

输出：

```
644537227223
Tue Jun 05 07:07:07 CDT 1990
```

除此之外，也常采用以下方法设置Calendar：

```
import java.util.Calendar;
import java.util.GregorianCalendar;

public class Test {

    public static void main(String[] args) {
        Calendar calendar = new GregorianCalendar();
        calendar.set(Calendar.YEAR, 1990);
        System.out.println(calendar.get(Calendar.YEAR));
        System.out.println(calendar.getTime());
    }
}
```

输出：

```
1990
Wed Dec 05 23:07:25 CST 1990
```

这种设置方式便于只设置某几种时间类型。上例中我只设置了年。其余的时间类型均与构建时的当前时间相同(由此也可见在新建GregorianCalendar类时，类似于Date类，无参构造函数默认取当前时刻)。

最后一种常用的设值方式就是直接注入Date对象或毫秒数：

```
calendar.setTime(new Date());    // Date类型
calendar.setTimeInMillis(1000L);    // 毫秒数
```

因日常开发所用的大多也就是GregorianCalendar类，因此设计者在Calendar中为我们添加了一个快捷方式：

```
Calendar calendar = Calendar.getInstance();
```

这种方式得到的同样是GregorianCalendar类的实例。也就是说等价于：

```
Calendar calendar = new GregorianCalendar();
```

**add方法**

第一个参数是待修改的类型，第二个参数是较之calendar的值加减的值

```
calendar.add(Calendar.YEAR, 10);    // calendar中的值加10年
calendar.add(Calendar.YEAR, -10);    // calendar中的值减10年
calendar.add(Calendar.DATE, 10);    // calendar中的值加10天
```

**getActualMinimum与getActualMaximum方法**

calendar的值对应所属传入类型的最小/最大值

```
calendar.getActualMinimum(Calendar.DAY_OF_MONTH)    // calendar中的值所属月份中的日期的最小值
calendar.getActualMaximum(Calendar.DAY_OF_MONTH)    // calendar中的值所属月份中的日期的最大值(例如11月为30,12月为31)
calendar.getActualMaximum(Calendar.DAY_OF_WEEK)    // calendar中的值所属星期中的日期的最大值(默认为7)
```

# 小例子：输出特定日期当月日历

```
import java.text.SimpleDateFormat;
import java.util.Calendar;

public class Test {

    public static void main(String[] args) throws Exception {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new SimpleDateFormat("yyyy-MM-dd").parse("1990-06-05"));
        int day = calendar.get(Calendar.DATE);
        int month = calendar.get(Calendar.MONTH);
        calendar.set(Calendar.DATE, calendar.getActualMinimum(Calendar.DAY_OF_MONTH));
        System.out.println("日\t一\t二\t三\t四\t五\t六");
        for (int i = 1; i <= calendar.get(Calendar.DAY_OF_WEEK) - 1; i++) System.out.print(" \t");
        while (calendar.get(Calendar.MONTH) == month) {
            System.out.print(calendar.get(Calendar.DATE) + (calendar.get(Calendar.DATE) == day ? "*\t" : "\t"));
            if (calendar.get(Calendar.DAY_OF_WEEK) == Calendar.SATURDAY) System.out.println();
            calendar.add(Calendar.DATE, 1);
        }
    }
}
```

输出：

```
日	一	二	三	四	五	六
 	 	 	 	 	1	2	
3	4	5*	6	7	8	9	
10	11	12	13	14	15	16	
17	18	19	20	21	22	23	
24	25	26	27	28	29	30
```