---
title: JSP自定义标签
date: 2021-02-02
categories: Spring
---

# 1. JSP自定义标签
尽管JSP中内置了许多标签，但有时还是需要根据需求自定义标签。如用逗号和空格格式化数字：
`<mytags:formatNumber number="100050.574" format="#,###.00"/>`
根据传入的number和format，在JSP页面上格式化这个数字，上面的数字应该被输出为`100,050.57`。
JSTL中没有提供内置的标签，所以只能自定义标签实现。
## 1.1 JSP自定义标签处理器
首先创建一个类，它继承自`javax.servlet.jsp.tagext.SimpleTagSupport`，并重写`doTag()`方法。重点是需要为标签的属性设置setter方法。所以定义两个setter方法--setFormat(String format)和setNumber(String number)。
SimpleTagSupport类提供方法使我们能获取JspWriter对象并输出数据到response。我们使用DecimalFormat类来生成格式化的字符串并输出到response。
```java
package com.zkzong.format;

import java.io.IOException;
import java.text.DecimalFormat;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.SkipPageException;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class NumberFormatterTag extends SimpleTagSupport {

	private String format;
	private String number;

	public NumberFormatterTag() {
	}

	public void setFormat(String format) {
		this.format = format;
	}

	public void setNumber(String number) {
		this.number = number;
	}

	@Override
	public void doTag() throws JspException, IOException {
		System.out.println("Number is:" + number);
		System.out.println("Format is:" + format);
		try {
			double amount = Double.parseDouble(number);
			DecimalFormat formatter = new DecimalFormat(format);
			String formattedNumber = formatter.format(amount);
			getJspContext().getOut().write(formattedNumber);
		} catch (Exception e) {
			e.printStackTrace();
			// stop page from loading further by throwing SkipPageException
			throw new SkipPageException("Exception in formatting " + number
					+ " with format " + format);
		}
	}

}

```
## 1.2 创建TLD（Tag Library Descriptor）文件
一旦标签处理器创建完成，就需要在WEB-INF文件中定义一个TLD文件。当应用部署后容器就会加载它。
```xml
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
	version="2.0">
	<description>Number Formatter Custom Tag</description>
	<tlib-version>2.1</tlib-version>
	<short-name>mytags</short-name>
	<uri>http://zkzong.com/jsp/tlds/mytags</uri>
	<tag>
		<name>formatNumber</name>
		<tag-class>com.zkzong.format.NumberFormatterTag</tag-class>
		<body-content>empty</body-content>
		<attribute>
			<name>format</name>
			<required>true</required>
		</attribute>
		<attribute>
			<name>number</name>
			<required>true</required>
		</attribute>
	</tag>
</taglib>
```
**注意：URI必须在tld文件中定义，而且format和number属性也使必需的。**
## 1.3 描述符配置
在web.xml文件中配置如下内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<display-name></display-name>
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>

	<jsp-config>
		<taglib>
			<taglib-uri>http://zkzong.com/jsp/tlds/mytags</taglib-uri>
			<taglib-location>/WEB-INF/numberformatter.tld</taglib-location>
		</taglib>
	</jsp-config>
</web-app>
```
`taglib-uri`的值要和TLD文件中定义的一样。
## 1.4 在JSP页面中使用自定义标签
JSP页面如下：
```jsp
<%@ page language="java" contentType="text/html; charset=US-ASCII"
	pageEncoding="US-ASCII"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=US-ASCII">
<title>Custom Tag Example</title>
<%@ taglib uri="http://zkzong.com/jsp/tlds/mytags" prefix="mytags"%>
</head>
<body>

	<h2>Number Formatting Example</h2>

	<mytags:formatNumber number="100050.574" format="#,###.00" />
	<br>
	<br>

	<mytags:formatNumber number="1234.567" format="$# ###.00" />
	<br>
	<br>

	<p>
		<strong>Thanks You!!</strong>
	</p>
</body>
</html>
```
如果有多个标签处理器类可以提供JAR文件供使用。只需在JAR文件的META-INF目录下包含TLD文件就可以了，然后把它放在web应用的lib目录下。这样就不用再web.xml中配置了，因为JSP容器会自动处理。这也是为什么我们使用JSP内置标签时不需要再web.xml中配置。

[源码下载](http://pan.baidu.com/s/1sjv7WHB)

参考文章：[http://www.journaldev.com/2099/jsp-custom-tags-example-tutorial](http://www.journaldev.com/2099/jsp-custom-tags-example-tutorial)
[http://www.codeproject.com/Articles/31614/JSP-JSTL-Custom-Tag-Library](http://www.codeproject.com/Articles/31614/JSP-JSTL-Custom-Tag-Library)

# 2. 如何创建JSTL自定义函数
## 2.1 在/WEB-INF文件夹下创建a.tld文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<taglib version="2.1" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd">
	<tlib-version>1.0</tlib-version>
	<short-name>mytlds</short-name>
	<uri>http://www.zkzong.com/myTlds</uri>
	<function>
		<name>charAt</name>
		<function-class>com.zkzong.jstl.Functions</function-class>
		<function-signature>char charAt(java.lang.String, int)</function-signature>
	</function>
</taglib>
```
## 2.2 创建类文件
```java
package com.zkzong.jstl;

public class Functions {
	public static char charAt(String input, int index) {
		return input.charAt(index);
	}
}
```
## 2.3 在jsp页面中使用
```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
   "http://www.w3.org/TR/html4/loose.dtd">
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@taglib uri="http://www.zkzong.com/myTlds" prefix="mt" %>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <body>
            ${mt:charAt("AB",0)} <!-- It will give you A-->
    </body>
</html>
```
[源码下载](http://pan.baidu.com/s/1qWp4wZq)

参考文章：
[http://www.noppanit.com/how-to-create-a-custom-function-for-jstl/](http://www.noppanit.com/how-to-create-a-custom-function-for-jstl/)
[http://findnerd.com/list/view/How-to-create-a-custom-Function-for-JSTL/2869/](http://findnerd.com/list/view/How-to-create-a-custom-Function-for-JSTL/2869/)


**tld文件中的uri可以不用配置。如果没有配置，在jsp页面中引入标签时，uri的值就是tld文件的路径，如`<%@ taglib prefix="ex" uri="WEB-INF/custom.tld"%>`。**
