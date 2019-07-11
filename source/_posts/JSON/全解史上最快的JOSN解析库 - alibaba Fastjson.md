---
title: 全解史上最快的JOSN解析库 - alibaba Fastjson
date: 2019-07-11
categories: JSON
---

JSON，全称：JavaScript Object Notation，作为一个常见的轻量级的数据交换格式，应该在一个程序员的开发生涯中是常接触的。简洁和清晰的层次结构使得 JSON 成为理想的数据交换语言。 易于阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率。

Java是面向对象的语言，所以我们更多的在项目中是以对象的形式处理业务的，但是在传输的时候我们却要将对象转换为 JSON 格式便于传输，而且 JSON 格式一般能解析为大多数的对象格式，而不在乎编程语言。

![阿里巴巴Logo](https://upload-images.jianshu.io/upload_images/292448-48ce594b6d973a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在主流的对象与 JSON 互转的工具很多，我们主要介绍今天的主角，阿里巴巴的开源库 - Fastjson。Fastjson是一个Java库，可用于将Java对象转换为其JSON表示。它还可用于将JSON字符串转换为等效的Java对象。Fastjson可以处理任意Java对象，包括您没有源代码的预先存在的对象。

![fastjson](https://upload-images.jianshu.io/upload_images/292448-c9bda211783e828c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1. 什么是 Fastjson?

阿里官方给的定义是， fastjson 是阿里巴巴的开源JSON解析库，它可以解析 JSON 格式的字符串，支持将 Java Bean 序列化为 JSON 字符串，也可以从 JSON 字符串反序列化到 JavaBean。

## 2. Fastjson 的优点

*   **速度快**
    fastjson相对其他JSON库的特点是快，从2011年fastjson发布1.1.x版本之后，其性能从未被其他Java实现的JSON库超越。
*   **使用广泛**
    fastjson在阿里巴巴大规模使用，在数万台服务器上部署，fastjson在业界被广泛接受。在2012年被开源中国评选为最受欢迎的国产开源软件之一。
*   **测试完备**
    fastjson有非常多的testcase，在1.2.11版本中，testcase超过3321个。每次发布都会进行回归测试，保证质量稳定。
*   **使用简单**
    fastjson的 API 十分简洁。
*   **功能完备**
    支持泛型，支持流处理超大文本，支持枚举，支持序列化和反序列化扩展。

## 3. 怎么获得 Fastjson

你可以通过如下地方下载fastjson:

*   maven中央仓库: [http://central.maven.org/maven2/com/alibaba/fastjson/](http://central.maven.org/maven2/com/alibaba/fastjson/)
*   Sourceforge.net : [https://sourceforge.net/projects/fastjson/files/](https://sourceforge.net/projects/fastjson/files/)
*   在maven项目的pom文件中直接配置fastjson依赖

fastjson最新版本都会发布到maven中央仓库，你可以直接依赖。

```
<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>fastjson</artifactId>
     <version>x.x.x</version>
</dependency>
```

其中x.x.x是版本号，根据需要使用特定版本，建议使用最新版本。

 ## 4. Fastjson 主要的API

Fastjson入口类是 com.alibaba.fastjson.JSON，主要的 API 是 JSON.toJSONString 和 parseObject。

```
package com.alibaba.fastjson;
public abstract class JSON {
      // Java对象转换为JSON字符串
      public static final String toJSONString(Object object);
      //JSON字符串转换为Java对象
      public static final <T> T parseObject(String text, Class<T> clazz, Feature... features);
}
```

序列化：

```
String jsonString = JSON.toJSONString(obj);
```

反序列化：

```
VO vo = JSON.parseObject("...", VO.class);
```

泛型反序列化：

```
import com.alibaba.fastjson.TypeReference;

List<VO> list = JSON.parseObject("...", new TypeReference<List<VO>>() {});
```

## 5. Fastjson 的性能

fastjson是目前java语言中最快的json库，比自称最快的jackson速度还要快，第三方独立测试结果看这里：[https://github.com/eishay/jvm-serializers/wiki](https://github.com/eishay/jvm-serializers/wiki)。

自行做性能测试时，需关闭循环引用检测的功能。

```
JSON.toJSONString(obj, SerializerFeature.DisableCircularReferenceDetect)
VO vo = JSON.parseObject("...", VO.class, Feature.DisableCircularReferenceDetect)
```

另外，Fastjson 比 Gson 快大约6倍，测试结果可以看这里：

```
Checking correctness…
[done]
Pre-warmup… java-built-in hessian kryo protostuff-runtime avro-generic msgpack json/jackson/databind json/jackson/databind-strings json/jackson/db-afterburner json/google-gson/databind json/svenson-databind json/flexjson/databind json/fastjson/databind smile/jackson/databind smile/jackson/db-afterburner smile/protostuff-runtime bson/jackson/databind xml/xstream+c xml/jackson/databind-aalto
[done]
pre. create ser deser shal +deep total size +dfl
java-built-in 63 5523 27765 28084 28162 33686 889 514
hessian 64 3776 6459 6505 6690 10466 501 313
kryo 63 809 962 937 1001 1810 214 133
protostuff-runtime 62 671 903 920 957 1627 241 151
avro-generic 436 1234 1122 1416 1760 2994 221 133
msgpack 61 789 1369 1385 1449 2238 233 146
json/jackson/databind 60 1772 3089 3113 3246 5018 485 261
json/jackson/databind-strings 64 2346 3739 3791 3921 6267 485 261
json/jackson/db-afterburner 64 1482 2220 2233 2323 3805 485 261
json/google-gson/databind 64 7076 4894 4962 5000 12076 486 259
json/svenson-databind 64 5422 12387 12569 12468 17890 495 266
json/flexjson/databind 62 20923 26853 26873 27272 48195 503 273
json/fastjson/databind 63 1250 1208 1206 1247 2497 486 262
smile/jackson/databind 60 1697 2117 2290 2298 3996 338 241
smile/jackson/db-afterburner 60 1300 1614 1648 1703 3003 352 252
smile/protostuff-runtime 61 1275 1612 1638 1685 2961 335 235
bson/jackson/databind 63 5151 6729 6977 6918 12069 506 286
xml/xstreamc 62 6358 13208 13319 13516 19874 487 244
xml/jackson/databind-aalto 62 2955 5332 5465 5584 8539 683 286
```

 ## 6. Fastjson 使用示例

我们创建一个班级的对象，和一个学生对象如下：

班级对象

```
public class Grade {

    private Long id;
    private String name;
    private List<Student> users = new ArrayList<Student>();

    // 省略 setter、getter

    public void addStudent(Student student) {
        users.add(student);
    }

    @Override
    public String toString() {
        return "Grade{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", users=" + users +
                '}';
    }
}
```

学生对象

```
public class Student {

    private Long   id;
    private String name;

    // 省略 setter、getter

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

运行的 Main 函数

```
public class MainTest {

    public static void main(String[] args) {
        Grade group = new Grade();
        group.setId(0L);
        group.setName("admin");

        Student student = new Student();
        student.setId(2L);
        student.setName("guest");

        Student rootUser = new Student();
        rootUser.setId(3L);
        rootUser.setName("root");

        group.addStudent(student);
        group.addStudent(rootUser);

        // 转换为 JSON
        String jsonString = JSON.toJSONString(group);
        System.out.println("JSON字符串：" + jsonString);

        // 转换为 对象BEAN
        Grade grade = JSON.parseObject(jsonString, Grade.class);
        System.out.println("JavaBean对象：" + grade);
    }
}
```

最后的运行结果如下：

```
JSON字符串：
{"id":0,"name":"admin","users":[{"id":2,"name":"guest"},{"id":3,"name":"root"}]}

JavaBean对象：
Grade{id=0, name='admin', users=[Student{id=2, name='guest'}, Student{id=3, name='root'}]}
```

 ## 7. 将对象中的空值输出

在fastjson中，缺省是不输出空值的。无论Map中的null和对象属性中的null，序列化的时候都会被忽略不输出，这样会减少产生文本的大小。但如果需要输出空值怎么做呢？

如果你需要输出空值，需要使用 `SerializerFeature.WriteMapNullValue`

```
Model obj = ...;
JSON.toJSONString(obj, SerializerFeature.WriteMapNullValue);
```

几种空值特别处理方式：

| SerializerFeature | 描述 |
| --- | --- |
| WriteNullListAsEmpty | 将Collection类型字段的字段空值输出为[] |
| WriteNullStringAsEmpty | 将字符串类型字段的空值输出为空字符串 "" |
| WriteNullNumberAsZero | 将数值类型字段的空值输出为0 |
| WriteNullBooleanAsFalse | 将Boolean类型字段的空值输出为false |

具体的示例参考如下，可以同时选择多个：

```
class Model {
      public List<Objec> items;
}

Model obj = ....;

String text = JSON.toJSONString(obj, SerializerFeature.WriteMapNullValue, SerializerFeature.WriteNullListAsEmpty);
```

 ## 8. Fastjson 处理日期

Fastjson 处理日期的API很简单，例如：

```
JSON.toJSONStringWithDateFormat(date, "yyyy-MM-dd HH:mm:ss.SSS")
```

使用ISO-8601日期格式

```
JSON.toJSONString(obj, SerializerFeature.UseISO8601DateFormat);
```

全局修改日期格式

```
JSON.DEFFAULT_DATE_FORMAT = "yyyy-MM-dd";
JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);
```

反序列化能够自动识别如下日期格式：

*   ISO-8601日期格式
*   yyyy-MM-dd
*   yyyy-MM-dd HH:mm:ss
*   yyyy-MM-dd HH:mm:ss.SSS
*   毫秒数字
*   毫秒数字字符串
*   .NET JSON日期格式
*   new Date(198293238)

虽然上面处理了单个的日期类型和全局的日期类型格式的配置，但是有时候我们需要的是对象中个别的日期类型差异化，并不一定是同一种格式的。那如何处理呢？接下来介绍 Fastjson 的定制序列化。

## 9. Fastjson 定制序列化
### 9.1 简介

fastjson支持多种方式定制序列化。

*   通过@JSONField定制序列化
*   通过@JSONType定制序列化
*   通过SerializeFilter定制序列化
*   通过ParseProcess定制反序列化

 ### 9.2 使用@JSONField配置

#### 1、JSONField 注解介绍

```
package com.alibaba.fastjson.annotation;

public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;

     // 指定字段的名称
    String name() default "";

    // 指定字段的格式，对日期格式有用
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;
}
```

#### 2、JSONField配置方式

可以把@JSONField配置在字段或者getter/setter方法上，例如：

**配置在字段上**

```
public class VO {
     @JSONField(name="ID")
     private int id;

     @JSONField(name="birthday",format="yyyy-MM-dd")
     public Date date;
}
```

**配置在 Getter/Setter 上**

```
public class VO {
    private int id;

    @JSONField(name="ID")
    public int getId() { return id;}

    @JSONField(name="ID")
    public void setId(int id) {this.id = id;}
}
```

> 注意：若属性是私有的，必须有set*方法。否则无法反序列化。

#### 3、使用format配置日期格式化

可以定制化配置各个日期字段的格式化

```
 public class A {
      // 配置date序列化和反序列使用yyyyMMdd日期格式
      @JSONField(format="yyyyMMdd")
      public Date date;
 }
```

#### 4、使用serialize/deserialize指定字段不序列化

```
public class A {
      @JSONField(serialize=false)
      public Date date;
 }

 public class A {
      @JSONField(deserialize=false)
      public Date date;
 }
```

#### 5、使用ordinal指定字段的顺序

缺省Fastjson序列化一个java bean，是根据fieldName的字母序进行序列化的，你可以通过ordinal指定字段的顺序。这个特性需要1.1.42以上版本。

```
public static class VO {
    @JSONField(ordinal = 3)
    private int f0;

    @JSONField(ordinal = 2)
    private int f1;

    @JSONField(ordinal = 1)
    private int f2;
}
```

#### 6、使用serializeUsing制定属性的序列化类

在fastjson 1.2.16版本之后，JSONField支持新的定制化配置serializeUsing，可以单独对某一个类的某个属性定制序列化，比如：

```
public static class Model {
    @JSONField(serializeUsing = ModelValueSerializer.class)
    public int value;
}

public static class ModelValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType,
                      int features) throws IOException {
        Integer value = (Integer) object;
        String text = value + "元";
        serializer.write(text);
    }
}
```

测试代码

```
Model model = new Model();
model.value = 100;
String json = JSON.toJSONString(model);
Assert.assertEquals("{\"value\":\"100元\"}", json);
```

 ### 9.3 使用@JSONType配置

和JSONField类似，但JSONType配置在类上，而不是field或者getter/setter方法上。

 ### 9.4通过SerializeFilter定制序列化

#### 1、简介

SerializeFilter是通过编程扩展的方式定制序列化。fastjson支持6种SerializeFilter，用于不同场景的定制序列化。

*   PropertyPreFilter 根据PropertyName判断是否序列化
*   PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化
*   NameFilter 修改Key，如果需要修改Key,process返回值则可
*   ValueFilter 修改Value
*   BeforeFilter 序列化时在最前添加内容
*   AfterFilter 序列化时在最后添加内容

#### 2、PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化

```
 public interface PropertyFilter extends SerializeFilter {
    boolean apply(Object object, String propertyName, Object propertyValue);
 }
```

可以通过扩展实现根据object或者属性名称或者属性值进行判断是否需要序列化。例如：

```
PropertyFilter filter = new PropertyFilter() {

    public boolean apply(Object source, String name, Object value) {
        if ("id".equals(name)) {
            int id = ((Integer) value).intValue();
            return id >= 100;
        }
        return false;
    }
};

JSON.toJSONString(obj, filter); // 序列化的时候传入filter
```

#### 3、PropertyPreFilter 根据PropertyName判断是否序列化

和PropertyFilter不同只根据object和name进行判断，在调用getter之前，这样避免了getter调用可能存在的异常。

```
 public interface PropertyPreFilter extends SerializeFilter {
      boolean apply(JSONSerializer serializer, Object object, String name);
  }
```

#### 4、NameFilter 序列化时修改Key

如果需要修改Key,process返回值则可

```
public interface NameFilter extends SerializeFilter {
    String process(Object object, String propertyName, Object propertyValue);
}
```

fastjson内置一个PascalNameFilter，用于输出将首字符大写的Pascal风格。 例如：

```
import com.alibaba.fastjson.serializer.PascalNameFilter;

Object obj = ...;
String jsonStr = JSON.toJSONString(obj, new PascalNameFilter());
```

#### 5、ValueFilter 序列化时修改Value

```
public interface ValueFilter extends SerializeFilter {
  Object process(Object object, String propertyName, Object propertyValue);
}
```

#### 6、BeforeFilter 序列化时在最前添加内容

在序列化对象的所有属性之前执行某些操作，例如调用 writeKeyValue 添加内容

```
public abstract class BeforeFilter implements SerializeFilter {
   protected final void writeKeyValue(String key, Object value) { ... }
    // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
    public abstract void writeBefore(Object object);
}
```

#### 7、AfterFilter 序列化时在最后添加内容

在序列化对象的所有属性之后执行某些操作，例如调用 writeKeyValue 添加内容

```
 public abstract class AfterFilter implements SerializeFilter {
  protected final void writeKeyValue(String key, Object value) { ... }
    // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
    public abstract void writeAfter(Object object);
}
```

 ### 9.5 通过ParseProcess定制反序列化

#### 1、简介

ParseProcess是编程扩展定制反序列化的接口。fastjson支持如下ParseProcess：

*   ExtraProcessor 用于处理多余的字段
*   ExtraTypeProvider 用于处理多余字段时提供类型信息

#### 2、使用ExtraProcessor 处理多余字段

```
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}

ExtraProcessor processor = new ExtraProcessor() {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
};

VO vo = JSON.parseObject("{\"id\":123,\"name\":\"abc\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals("abc", vo.getAttributes().get("name"));
```

#### 3、使用ExtraTypeProvider 为多余的字段提供类型

```
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}

class MyExtraProcessor implements ExtraProcessor, ExtraTypeProvider {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }

    public Type getExtraType(Object object, String key) {
        if ("value".equals(key)) {
            return int.class;
        }
        return null;
    }
};
ExtraProcessor processor = new MyExtraProcessor();

VO vo = JSON.parseObject("{\"id\":123,\"value\":\"123456\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals(123456, vo.getAttributes().get("value")); // value本应该是字符串类型的，通过getExtraType的处理变成Integer类型了。
```

 ## 10. 在 Spring MVC 中集成 Fastjson

如果你使用 Spring MVC 来构建 Web 应用并对性能有较高的要求的话，可以使用 Fastjson 提供的FastJsonHttpMessageConverter 来替换 Spring MVC 默认的 HttpMessageConverter 以提高 @RestController @ResponseBody @RequestBody 注解的 JSON序列化速度。下面是配置方式，非常简单。

**XML式**
如果是使用 XML 的方式配置 Spring MVC 的话，只需在 Spring MVC 的 XML 配置文件中加入下面配置即可

```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter"/>      
    </mvc:message-converters>
</mvc:annotation-driven>
```

通常默认配置已经可以满足大部分使用场景，如果你想对它进行自定义配置的话，你可以添加 FastJsonConfig Bean。

```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
            <property name="fastJsonConfig" ref="fastJsonConfig"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="fastJsonConfig" class="com.alibaba.fastjson.support.config.FastJsonConfig">
    <!--   自定义配置...   -->
</bean>
```

**编程式**
如果是使用编程的方式（通常是基于 Spring Boot 项目）配置 Spring MVC 的话只需继承 `WebMvcConfigurerAdapter`覆写`configureMessageConverters`方法即可，就像下面这样。

```
@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        //自定义配置...
        //FastJsonConfig config = new FastJsonConfig();
        //config.set ...
        //converter.setFastJsonConfig(config);
        converters.add(0, converter);
    }
}
```

> **注意**：
> **1、**如果你使用的 Fastjson 版本小于1.2.36的话(强烈建议使用最新版本)，在与Spring MVC 4.X 版本集成时需使用 FastJsonHttpMessageConverter4。
> 
> **2、**SpringBoot 2.0.1版本中加载WebMvcConfigurer的顺序发生了变动，故需使用converters.add(0, converter);指定FastJsonHttpMessageConverter在converters内的顺序，否则在SpringBoot 2.0.1及之后的版本中将优先使用Jackson处理。

 ## 11. 在 Spring Data Redis 中集成 Fastjson

通常我们在 Spring 中使用 Redis 是通过 Spring Data Redis 提供的 RedisTemplate 来进行的，如果你准备使用 JSON 作为对象序列/反序列化的方式并对序列化速度有较高的要求的话，建议使用 Fastjson 提供的 `GenericFastJsonRedisSerializer` 或 `FastJsonRedisSerializer` 作为 RedisTemplate 的 RedisSerializer。下面是配置方式，非常简单。

**XML式**
如果是使用 XML 的方式配置 Spring Data Redis 的话，只需将 RedisTemplate 中的 Serializer 替换为 GenericFastJsonRedisSerializer 即可。

```
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
    <property name="defaultSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
</bean>
```

下面是完整的 Spring 集成 Redis 配置供参考。

```
<!-- Redis 连接池配置(可选) -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="${redis.pool.maxActive}"/>
    <property name="maxIdle" value="${redis.pool.maxIdle}"/>
    <property name="maxWaitMillis" value="${redis.pool.maxWait}"/>
    <property name="testOnBorrow" value="${redis.pool.testOnBorrow}"/>
     <!-- 更多连接池配置...-->
</bean>
<!-- Redis 连接工厂配置 -->
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <!--设置连接池配置，不设置的话会使用默认的连接池配置，若想禁用连接池可设置 usePool = false -->   
    <property name="poolConfig" ref="jedisPoolConfig" />  
    <property name="hostName" value="${host}"/>
    <property name="port" value="${port}"/>
    <property name="password" value="${password}"/>
    <property name="database" value="${database}"/>
    <!-- 更多连接工厂配置...-->
</bean>
<!-- RedisTemplate 配置 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <!-- 设置 Redis 连接工厂-->
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
    <!-- 设置默认 Serializer ，包含 keySerializer & valueSerializer -->
    <property name="defaultSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
    <!-- 单独设置 keySerializer -->
    <property name="keySerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
    <!-- 单独设置 valueSerializer -->
    <property name="valueSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
</bean>
```

**编程式**

如果是使用编程的方式（通常是基于 Spring Boot 项目）配置 RedisTemplate 的话只需在你的配置类(被@Configuration注解修饰的类)中显式创建 RedisTemplate Bean，设置 Serializer 即可。

```
@Bean
public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate redisTemplate = new RedisTemplate();
    redisTemplate.setConnectionFactory(redisConnectionFactory);

    GenericFastJsonRedisSerializer fastJsonRedisSerializer = new GenericFastJsonRedisSerializer();
    redisTemplate.setDefaultSerializer(fastJsonRedisSerializer);//设置默认的Serialize，包含 keySerializer & valueSerializer

    //redisTemplate.setKeySerializer(fastJsonRedisSerializer);//单独设置keySerializer
    //redisTemplate.setValueSerializer(fastJsonRedisSerializer);//单独设置valueSerializer
    return redisTemplate;
}
```

通常使用 `GenericFastJsonRedisSerializer` 即可满足大部分场景，如果你想定义特定类型专用的 RedisTemplate 可以使用 `FastJsonRedisSerializer`来代替 `GenericFastJsonRedisSerializer`，配置是类似的。

参考：[https://github.com/alibaba/fastjson/wiki](https://github.com/alibaba/fastjson/wiki)
[[https://www.cnblogs.com/jajian/p/10051901.html](https://www.cnblogs.com/jajian/p/10051901.html)
]([https://www.cnblogs.com/jajian/p/10051901.html](https://www.cnblogs.com/jajian/p/10051901.html)
)