---
title: iBATIS
date: 2019-05-14
categories: MyBatis
---

**（针对iBATIS 2.x版本）**
# 1. 简介
iBATIS是一个ORM框架，它和其它框架（如Hibernate）的最大不同是：iBATIS强调SQL的使用，而其它框架使用自定义的查询语言如HQL。
# 2. 环境配置
1. 创建EMPLOYEE表，SQL语句如下：
```sql
CREATE TABLE EMPLOYEE (
    id INT NOT NULL auto_increment,
    first_name VARCHAR(20) default NULL,
    last_name VARCHAR(20) default NULL,
    salary INT default NULL,
    PRIMARY KEY (id)
);
```
创建EMPLOYEE表。
2. 创建SqlMapConfig.xml
此文件配置数据库连接等信息。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMapConfig
PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
<sqlMapConfig>
	<settings useStatementNamespaces="true" />
	<transactionManager type="JDBC">
		<dataSource type="SIMPLE">
			<property name="JDBC.Driver" value="com.mysql.jdbc.Driver" />
			<property name="JDBC.ConnectionURL" value="jdbc:mysql://localhost:3306/test" />
			<property name="JDBC.Username" value="root" />
			<property name="JDBC.Password" value="root" />
		</dataSource>
	</transactionManager>
	<sqlMap resource="Employee.xml" />
</sqlMapConfig>
```

以下是一些可选的配置信息：
```xml
<property name="JDBC.AutoCommit" value="true"/>
<property name="Pool.MaximumActiveConnections" value="10"/>
<property name="Pool.MaximumIdleConnections" value="5"/>
<property name="Pool.MaximumCheckoutTime" value="150000"/>
<property name="Pool.MaximumTimeToWait" value="500"/>
<property name="Pool.PingQuery" value="select 1 from Employee"/>
<property name="Pool.PingEnabled" value="false"/>
```
# 3. CREATE操作
1. Employee POJO类
```java
package com.zkzong.ibatis;

public class Employee {
	private int id;
	private String first_name;
	private String last_name;
	private int salary;

	/* Define constructors for the Employee class. */
	public Employee() {
	}

	public Employee(String fname, String lname, int salary) {
		this.first_name = fname;
		this.last_name = lname;
		this.salary = salary;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getFirstName() {
		return first_name;
	}

	public void setFirstName(String fname) {
		this.first_name = fname;
	}

	public String getLastName() {
		return last_name;
	}

	public void setlastName(String lname) {
		this.last_name = lname;
	}

	public int getSalary() {
		return salary;
	}

	public void setSalary(int salary) {
		this.salary = salary;
	}
} /* End of Employee */

```
2. 创建Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<!-- Perform Insert Operation -->
	<insert id="insert" parameterClass="com.zkzong.ibatis.Employee">
	
		insert into EMPLOYEE(first_name, last_name, salary)
		values (#first_name#, #last_name#, #salary#)
		
		<selectKey resultClass="int" keyProperty="id">
			select last_insert_id() as id
		</selectKey>
		
	</insert>
	
</sqlMap>
```
parameterClass可以是string、int、float、double或其他类对象。本例中我们在调用insert方法传入Employee作为参数。
如果数据库表使用了IDENTITY、AUTO_INCREMENT、SERIAL或定义了SEQUENCE/GENERATOR，可以在<insert>中使用<selectKey>来使用或返回数据库生成的值。
3. IbatisInsert.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisInsert {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		/* This would insert one record in Employee table. */
		System.out.println("Going to insert record.....");
		Employee em = new Employee("Zara", "Ali", 5000);
		
		smc.insert("Employee.insert", em);
		
		System.out.println("Record Inserted Successfully ");
	}
}

```
# 4. READ操作
1. Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<!-- Perform Read Operation -->
	<select id="getAll" resultClass="com.zkzong.ibatis.Employee">
		SELECT * FROM EMPLOYEE
	</select>

</sqlMap>
```
2. IbatisRead.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;
import java.util.List;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisRead {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		/* This would read all records from the Employee table. */
		System.out.println("Going to read records.....");
		List<Employee> ems = (List<Employee>) smc.queryForList("Employee.getAll", null);
		Employee em = null;
		for (Employee e : ems) {
			System.out.print(" " + e.getId());
			System.out.print(" " + e.getFirstName());
			System.out.print(" " + e.getLastName());
			System.out.print(" " + e.getSalary());
			em = e;
			System.out.println("");
		}
		
		System.out.println("Records Read Successfully ");
	}
}

```
# 5. UPDATE操作
1. Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<!-- Perform Update Operation -->
	<update id="update" parameterClass="com.zkzong.ibatis.Employee">
		UPDATE EMPLOYEE
		SET first_name = #first_name#
		WHERE id = #id#
	</update>

</sqlMap>
```
2. IbatisUpdate.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;
import java.util.List;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisUpdate {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		/* This would update one record in Employee table. */
		System.out.println("Going to update record.....");
		Employee rec = new Employee();
		rec.setId(1);
		rec.setFirstName("Roma");
		smc.update("Employee.update", rec);
		System.out.println("Record updated Successfully ");
		
		System.out.println("Going to read records.....");
		List<Employee> ems = (List<Employee>) smc.queryForList("Employee.getAll", null);
		Employee em = null;
		for (Employee e : ems) {
			System.out.print(" " + e.getId());
			System.out.print(" " + e.getFirstName());
			System.out.print(" " + e.getLastName());
			System.out.print(" " + e.getSalary());
			em = e;
			System.out.println("");
		}
		
		System.out.println("Records Read Successfully ");
	}
}

```
# 6. DELETE操作
1. Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<!-- Perform Delete Operation -->
	<delete id="delete" parameterClass="int">
		DELETE FROM EMPLOYEE
		WHERE id = #id#
	</delete>

</sqlMap>
```
2. IbatisDelete.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;
import java.util.List;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisDelete {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		/* This would delete one record in Employee table. */
		System.out.println("Going to delete record.....");
		int id = 1;
		
		smc.delete("Employee.delete", id);
		System.out.println("Record deleted Successfully ");
		
		System.out.println("Going to read records.....");
		List<Employee> ems = (List<Employee>) smc.queryForList(
				"Employee.getAll", null);
		Employee em = null;
		for (Employee e : ems) {
			System.out.print(" " + e.getId());
			System.out.print(" " + e.getFirstName());
			System.out.print(" " + e.getLastName());
			System.out.print(" " + e.getSalary());
			em = e;
			System.out.println("");
		}
		
		System.out.println("Records Read Successfully ");
	}
}

```
# 7. RESULT MAPS
resultMap是iBATIS中最重要的元素。使用iBATIS ResultMap最多可以减少90%的JDBC编码，甚至可以实现JDBC不支持的操作。
1. Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<!-- Using ResultMap -->
	<resultMap id="result" class="com.zkzong.ibatis.Employee">
		<result property="id" column="id"/>
		<result property="first_name" column="first_name"/>
		<result property="last_name" column="last_name"/>
		<result property="salary" column="salary"/>
	</resultMap>
	<select id="useResultMap" resultMap="result">
		SELECT * FROM EMPLOYEE
		WHERE id=#id#
	</select>
	
</sqlMap>
```
2. IbatisResultMap.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisResultMap {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		int id = 2;
		System.out.println("Going to read record.....");
		Employee e = (Employee) smc.queryForObject("Employee.useResultMap", id);
		
		System.out.println("ID: " + e.getId());
		System.out.println("First Name: " + e.getFirstName());
		System.out.println("Last Name: " + e.getLastName());
		System.out.println("Salary: " + e.getSalary());
		
		System.out.println("Record read Successfully ");
	}
}

```
# 8. 存储过程

# 9. 动态SQL
动态SQL是iBATIS非常重要的特性。有时需要改变WHERE语句的查询条件，在这种情况下iBATIS提供了一系列的动态SQL标签用来映射条件以加强SQL的可重用性和灵活性。
所有的逻辑都是放在.XML文件的标签里。
1. Employee.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE sqlMap
PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap namespace="Employee">

	<select id="findByID" resultClass="com.zkzong.ibatis.Employee">
		SELECT * FROM EMPLOYEE
		<dynamic prepend="WHERE ">
			<isNull property="id">
				id IS NULL
			</isNull>
			<isNotNull property="id">
				id = #id#
			</isNotNull>
		</dynamic>
	</select>
</sqlMap>
```
2. IbatisReadDy.java
```java
package com.zkzong.ibatis;

import java.io.IOException;
import java.io.Reader;
import java.sql.SQLException;
import java.util.List;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class IbatisReadDy {
	public static void main(String[] args) throws IOException, SQLException {
		Reader rd = Resources.getResourceAsReader("SqlMapConfig.xml");
		SqlMapClient smc = SqlMapClientBuilder.buildSqlMapClient(rd);
		
		/* This would read all records from the Employee table. */
		System.out.println("Going to read records.....");
		Employee rec = new Employee();
		rec.setId(2);
		
		List<Employee> ems = (List<Employee>) smc.queryForList(
				"Employee.findByID", rec);
		Employee em = null;
		for (Employee e : ems) {
			System.out.print(" " + e.getId());
			System.out.print(" " + e.getFirstName());
			System.out.print(" " + e.getLastName());
			System.out.print(" " + e.getSalary());
			em = e;
			System.out.println("");
		}
		
		System.out.println("Records Read Successfully ");
	}
}

```
3. iBATIS OGNL表达式
iBATIS提供强大的OGNL表达式以减少其他元素。
+ if
+ choose, when, otherwise
+ where
+ foreach
**if语句**
动态SQL中最常用的是在where子句中包含条件：
```xml
<select id="findActiveBlogWithTitleLike" parameterType="Blog" resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = 'ACTIVE.
    <if test="title != null">
        AND title like #{title}
    </if>
</select>
```
这个语句提供可选的文本查询功能。如果传入的title为空，返回所有的active Blog。如果传入的title不为空，根据like条件查询。
可以包含多个if语句：
```xml
<select id="findActiveBlogWithTitleLike" parameterType="Blog" resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = 'ACTIVE.
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null">
        AND author like #{author}
    </if>
</select>
```
**choose, when, otherwise语句**
iBATIS中的**choose**和Java中的switch语句类似，选择其中一个执行。
```xml
<select id="findActiveBlogWithTitleLike"
parameterType="Blog" resultType="Blog">
    SELECT * FROM BLOG
    WHERE state = 'ACTIVE'
    <choose>
        <when test="title != null">
            AND title like #{title}
        </when>
        <when test="author != null and author.name != null">
            AND author like #{author}
        </when>
        <otherwise>
            AND featured = 1
        </otherwise>
    </choose>
</select>
```
**where语句**
如果条件中没有符合的，可能会出现这样的SQL:
```sql
SELECT * FROM BLOG
WHERE
```
这个SQL会执行失败，如果改为以下的代码就可以执行：
```xml
<select id="findActiveBlogLike"
parameterType="Blog" resultType="Blog">
    SELECT * FROM BLOG
    <where>
        <if test="state != null">
            state = #{state}
        </if>
        <if test="title != null">
            AND title like #{title}
        </if>
        <if test="author != null>
            AND author like #{author}
        </if>
    </where>
</select>
```
只有有标签返回内容是才会插入where语句。而且如果返回的内容以`AND`或`OR`开始，它会自动去除。
**foreach语句**
foreach元素允许指定collection和声明item、index以便在元素体内使用。
允许指定open和close字符串，在iterations之间添加separator。创建in条件如下：
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    WHERE ID in
    <foreach item="item" index="index" collection="list"
open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```
