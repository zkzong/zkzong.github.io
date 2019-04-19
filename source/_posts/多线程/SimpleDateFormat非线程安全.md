---
title: SimpleDateFormat非线程安全
date: 2019-04-19
categories: 多线程
---

类SimpleDateFormat主要负责日期的转换与格式化，但在多线程的环境中，使用此类容易造成数据转换及处理的不准确，因为SimpleDateFormat类并不是线程安全的。

## 出现异常

本示例将实现实用类SimpleDateFormat在多线程环境下处理日期但得出的结果却是错误的情况，这也是在多线程环境开发中容易遇到的问题。

类MyThread.java代码如下：
```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class MyThread extends Thread {

	private SimpleDateFormat sdf;
	private String dateString;

	public MyThread(SimpleDateFormat sdf, String dateString) {
		super();
		this.sdf = sdf;
		this.dateString = dateString;
	}

	@Override
	public void run() {
		try {
			Date dateRef = sdf.parse(dateString);
			String newDateString = sdf.format(dateRef).toString();
			if (!newDateString.equals(dateString)) {
				System.out.println("ThreadName=" + this.getName()
						+ "报错了 日期字符串：" + dateString + " 转换成的日期为："
						+ newDateString);
			}
		} catch (ParseException e) {
			e.printStackTrace();
		}

	}

}
```

运行类Test.java代码如下：
```java
import java.text.SimpleDateFormat;

import extthread.MyThread;

public class Test {

	public static void main(String[] args) {

		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

		String[] dateStringArray = new String[] { "2000-01-01", "2000-01-02",
				"2000-01-03", "2000-01-04", "2000-01-05", "2000-01-06",
				"2000-01-07", "2000-01-08", "2000-01-09", "2000-01-10" };

		MyThread[] threadArray = new MyThread[10];
		for (int i = 0; i < 10; i++) {
			threadArray[i] = new MyThread(sdf, dateStringArray[i]);
		}
		for (int i = 0; i < 10; i++) {
			threadArray[i].start();
		}

	}
}
```

程序运行后的结果如下：
```text
ThreadName=Thread-4报错了 日期字符串：2000-01-05 转换成的日期为：2000-02-24
ThreadName=Thread-5报错了 日期字符串：2000-01-06 转换成的日期为：0005-02-24
ThreadName=Thread-8报错了 日期字符串：2000-01-09 转换成的日期为：0001-01-10
ThreadName=Thread-9报错了 日期字符串：2000-01-10 转换成的日期为：0001-01-10
```

从控制台中打印的结果来看，使用单例的SimpleDateFormat类在多线程的环境中处理日期，极易出现日期转换错误的情况。

## 解决异常方法1

类MyThread.java代码如下：
```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import tools.DateTools;

public class MyThread extends Thread {

    private SimpleDateFormat sdf;
    private String dateString;

    public MyThread(SimpleDateFormat sdf, String dateString) {
        super();
        this.sdf = sdf;
        this.dateString = dateString;
    }

    @Override
    public void run() {
        try {
            Date dateRef = DateTools.parse("yyyy-MM-dd", dateString);
            String newDateString = DateTools.format("yyyy-MM-dd", dateRef)
                    .toString();
            if (!newDateString.equals(dateString)) {
                System.out.println("ThreadName=" + this.getName()
                        + "报错了 日期字符串：" + dateString + " 转换成的日期为："
                        + newDateString);
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }

}
```

类DateTools.java代码如下：
```java
public class DateTools {

    public static Date parse(String formatPattern, String dateString)
            throws ParseException {
        return new SimpleDateFormat(formatPattern).parse(dateString);
    }

    public static String format(String formatPattern, Date date) {
        return new SimpleDateFormat(formatPattern).format(date).toString();
    }

}
```

运行类Test.java代码与前面一节是一样的。
控制台中没有输入任何异常。解决处理错误的原理其实就是**创建了多个SimpleDateFormat类的实例**。

## 解决异常方法2

ThreadLocal类能使线程绑定到指定的对象。使用该类也可以解决多线程环境下SimpleDateFormat类处理错误的情况。

类MyThread.java代码如下：
```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import tools.DateTools;

public class MyThread extends Thread {

    private SimpleDateFormat sdf;
    private String dateString;

    public MyThread(SimpleDateFormat sdf, String dateString) {
        super();
        this.sdf = sdf;
        this.dateString = dateString;
    }

    @Override
    public void run() {
        try {
            Date dateRef = DateTools.getSimpleDateFormat("yyyy-MM-dd").parse(dateString);
            String newDateString = DateTools.getSimpleDateFormat("yyyy-MM-dd")
                    .format(dateRef).toString();
            if (!newDateString.equals(dateString)) {
                System.out.println("ThreadName=" + this.getName()
                        + "报错了 日期字符串：" + dateString + " 转换成的日期为："
                        + newDateString);
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }

}
```

类DateTools.java代码如下：
```java
import java.text.SimpleDateFormat;

public class DateTools {

    private static ThreadLocal<SimpleDateFormat> tl = new ThreadLocal<SimpleDateFormat>();

    public static SimpleDateFormat getSimpleDateFormat(String datePattern) {
        SimpleDateFormat sdf = null;
        sdf = tl.get();
        if (sdf == null) {
            sdf = new SimpleDateFormat(datePattern);
            tl.set(sdf);
        }
        return sdf;
    }

}
```
运行类Test.java代码与前面小节是一样的。
控制台没有信息被输出，看来运行结果是正确的。
