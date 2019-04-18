MyBatis在转换查询结果到需要的Java业务对象时做了三件事：

1. 解决了数据库列名到Java列名的映射。
2. 解决了数据库类型到Java类型的转换工作。
3. 在转换过程中具备一定的容错能力。

其实核心就是：

1. 数据库中的列名怎么和对象中的字段对应起来。
2. 数据库中的列的类型怎么转换到合适的Java类型，不引起转换失败。

我们先来看第一点，数据库中的列名怎么和对象中的字段对应起来。首先是PO(Persistant Object) CityPO里面有五个字段：
```java
public class CityPO {
    Integer id;
    Long cityId;
    String cityName;
    String cityEnName;
    String cityPyName;
}
```

本次要查询的数据库中的列名如下所示:
```
mysql> mysql> desc SU_City;
+--------------+-------------+------+-----+-------------------+-----------------------------+
| Field        | Type        | Null | Key | Default           | Extra                       |
+--------------+-------------+------+-----+-------------------+-----------------------------+
| id           | int(11)     | NO   | PRI | NULL              | auto_increment              |
| city_id      | int(11)     | NO   | UNI | NULL              |                             |
| city_name    | varchar(20) | NO   |     |                   |                             |
| city_en_name | varchar(20) | NO   |     |                   |                             |
| city_py_name | varchar(50) | NO   |     |                   |                             |
| create_time  | datetime    | NO   |     | CURRENT_TIMESTAMP |                             |
| updatetime   | datetime    | NO   | MUL | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+--------------+-------------+------+-----+-------------------+-----------------------------+
7 rows in set (0.01 sec)
```

我们是按照驼峰式命名，把数据库中的列名对应到了对象的字段名。如下是MyBatis的接口类和映射文件。
接口类：
```java
public interface CityMapper {
    CityPO selectCity(int id);
}
```
映射文件：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.CityMapper">
    <select id="selectCity" resultType="po.CityPO">
        select id,city_id,city_name,city_en_name from SU_City where id = #{id}
    </select>
</mapper>
```

在上面的映射文件中，namespace指定了这个接口类的全限定类名，紧随其后的select代表是select语句，id是接口类中函数的名字，resultType代表了从这条语句中返回的期望类型的类的完全限定名或别名，在此例子中是我们的业务对象CityPO的类路径。

主要有三种方案：

1. 驼峰式命名开关，或者不开——数据库列和字段名全一致。
2. Select时指定AS。
3. resultMap 最稳健。

## 1. 驼峰命名开关

因为CityPO的列名是完全根据数据库列名驼峰式命名后得到的，因此MyBatis提供了一个配置项。开启开配置项后，在匹配时，能够根据数据库列名找到对应对应的驼峰式命名后的字段。
```xml
<settings>
    <!-- 开启驼峰，开启后，只要数据库字段和对象属性名字母相同，无论中间加多少下划线都可以识别 -->
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```
从源码角度解读一下，MyBatis处理ResultSet的映射默认都在DefaultResultSetHandler中完成。

处理行数据的源码主要在下面的函数里进行，由于我们在映射文件中没有定义额外的ResultMap，因此会直接进入else分支的代码。

```java
public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
    if (resultMap.hasNestedResultMaps()) {
        ensureNoRowBounds();
        checkResultHandler();
        handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    } else {
        handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    }
}
```
进入handleRowValuesForSimpleResultMap中，主要处理函数如下，在这里完成了对象的生成及赋值。
```java
Object rowValue = getRowValue(rsw, discriminatedResultMap);
```
 在这里先创建了对象的实例，然后获取了对象的元信息，为反射赋值做准备。
```java
private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap) throws SQLException {
    final ResultLoaderMap lazyLoader = new ResultLoaderMap();
    Object rowValue = createResultObject(rsw, resultMap, lazyLoader, null);
    if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        final MetaObject metaObject = configuration.newMetaObject(rowValue);
        boolean foundValues = this.useConstructorMappings;
        if (shouldApplyAutomaticMappings(resultMap, false)) {
            foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, null) || foundValues;
        }
        foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, null) || foundValues;
        foundValues = lazyLoader.size() > 0 || foundValues;
        rowValue = (foundValues || configuration.isReturnInstanceForEmptyRow()) ? rowValue : null;
    }
    return rowValue;
}
```

在applyAutomaticMappings完成了整个过程，我们进去探一探。

就是下面这个函数创建好了映射关系，这个函数的下半部分是完成赋值的，映射的部分下次会详细分析。

```java
List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);
```

在这个方法里，上半部分是生成了数据库的列名，在这个函数中找到了对应的字段名。

```java
final String property = metaObject.findProperty(propertyName, configuration.isMapUnderscoreToCamelCase());
```

我们进去看一看，它传进了生成好的数据库列名，传进了前面提到的是否根据驼峰式命名映射开关的值。

事实证明，真的很简单，往下看，就是把下划线都去了。

```java
public String findProperty(String name, boolean useCamelCaseMapping) {
    if (useCamelCaseMapping) {
        name = name.replace("_", "");
    }
    return findProperty(name);
}
```

隐隐觉得是不是大小写不敏感啊，继续往下看，这里返回找到的字段名。

```java
private StringBuilder buildProperty(String name, StringBuilder builder) {
    ......
    String propertyName = reflector.findPropertyName(name);
    if (propertyName != null) {
        builder.append(propertyName);
    }
    return builder;
}
```

 好了，真相大白，就是大小写不敏感的。

```java
public String findPropertyName(String name) {
    return caseInsensitivePropertyMap.get(name.toUpperCase(Locale.ENGLISH));
}
```

![大小写不敏感](http://upload-images.jianshu.io/upload_images/292448-5e3ffa7836f72460.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. Select .... AS

当我们的数据库列名和对象字段之间不是驼峰式命名的关系，可以在Select时使用AS，使得列名和对象名匹配上。

映射文件中是本次会执行的sql，我们会查出id，city_id，city_name，city_en_name。 按照开启的驼峰式命名开关，我们会对应到对象的id，cityId，cityName，cityEnName字段。

```xml
<select id="selectCity" resultType="po.CityPO">
    select id,city_id,city_name,city_en_name from SU_City where id = #{id}
</select>
```

不过在这次，我们对PO做了小小的改动，把cityEnName改成了cityEnglishName。

```java
public class CityPO {
	Integer id;
	Long cityId;
	String cityName;
	String cityEnglishName;   // 由cityEnName改成了cityEnglishName
}
```

由于找不到匹配的列，cityEnlishName肯定没法被反射赋值，因此值为Null。

```
CityPO{id=2, cityId=2, cityName='北京', cityEnglishName='null'}
```

**解决办法：**在Select字段的时候使用AS，下面是改动后的映射文件。

```xml
<select id="selectCity" resultType="po.CityPO">
        select id,
        city_id,
        city_name,
        city_en_name AS cityEnglishName
        from SU_City
        where id = #{id}
</select>
```

改动后执行得到的结果如下。

```
CityPO{id=2, cityId=2, cityName='北京', cityEnglishName='beijing'}
```

那么我们来看看它是如何生效的，主要的代码在哪里。上面我们第一个介绍的函数handleRowValues中传入了参数rsw，它是对ResultSet的一个包装，在这个包装里，完成了具体使用哪个名字作为数据库的列名。

```java
final ResultSetWrapper rsw = new ResultSetWrapper(rs, configuration);
handleRowValues(rsw, resultMap, resultHandler, new RowBounds(), null);
```

在这个构造函数当中，我们会获取数据库的列名，AS为什么可以生效，具体就在下面这段代码。

```java
super();
this.typeHandlerRegistry = configuration.getTypeHandlerRegistry();
this.resultSet = rs;
final ResultSetMetaData metaData = rs.getMetaData();
final int columnCount = metaData.getColumnCount();
for (int i = 1; i <= columnCount; i++) {
    // 在这里
    columnNames.add(configuration.isUseColumnLabel() ? metaData.getColumnLabel(i) : metaData.getColumnName(i));
    jdbcTypes.add(JdbcType.forCode(metaData.getColumnType(i)));
    classNames.add(metaData.getColumnClassName(i));
}
```

在添加列名时，会从配置中获取是否使用类标签，isUseColumnLabel，默认为true。根据Javadoc，这个ColumnLabel就是AS后的那个名字，如果没有AS的话，就是获取的原生的字段名。

```java
/**
 * Gets the designated column's suggested title for use in printouts and
 * displays. The suggested title is usually specified by the SQL <code>AS</code>
 * clause.  If a SQL <code>AS</code> is not specified, the value returned from
 * <code>getColumnLabel</code> will be the same as the value returned by the
 * <code>getColumnName</code> method.
 *
 * @param column the first column is 1, the second is 2, ...
 * @return the suggested column title
 * @exception SQLException if a database access error occurs
 */
String getColumnLabel(int column) throws SQLException;
```

后面的过程就和上面方案一一模一样了，不再赘述。

## 3. ResultMap

> resultMap 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC ResultSets 数据提取代码中解放出来，并在一些情形下允许你做一些 JDBC 不支持的事情。 实际上，在对复杂语句进行联合映射的时候，它很可能可以代替数千行的同等功能的代码。 ResultMap 的设计思想是，简单的语句不需要明确的结果映射，而复杂一点的语句只需要描述它们的关系就行了。

ResultMap是MyBatis中可以完成复杂语句映射的东西，但在我们的日常开发中，我们往往是一个XML对应JavaBeans 或 POJOs(Plain Old Java Objects，普通 Java 对象)，并没有特别复杂的应用，下面也是基于日常的使用，看看简单的ResultMap在源码层面是如何展现的。

```xml
<resultMap id="cityMap" type="po.CityPO">
        <result column="id" property="id"/>
        <result column="city_id" property="cityId"/>
        <result column="city_name" property="cityName"/>
        <result column="city_en_name" property="cityEnglishName"/>
</resultMap>

<select id="selectCity" resultMap="cityMap">
        select id,
        city_id,
        city_name,
        city_en_name
        from SU_City
        where id = #{id}
</select>
```

在resultMap的子元素result对应了result和对象字段之间的映射，并通过id标示，你在Select语句中指定需要使用的resultMap即可。

源码层面的话，依旧在DefaultResultSetHandler的handleResultSets中处理返回集合。

```
List<ResultMap> resultMaps = mappedStatement.getResultMaps();
```

在这次的ResultMap中，相比之前方案，其属性更加的丰富起来。将之前写的Result的信息保存在resultMappings，idResultMappings等中，以备后续使用。

![resultMappings](http://upload-images.jianshu.io/upload_images/292448-73956168a0b02a6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后续的函数走向和方案一二一致，但在创建自动映射的时候出现了不同。

```java
private List<UnMappedColumnAutoMapping> createAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {
}
```

在这个函数中，会获取没有映射过的列名。

```java
final List<String> unmappedColumnNames = rsw.getUnmappedColumnNames(resultMap, columnPrefix);
```

之后会根据resultMap查看是否有未映射的字段。

```java
loadMappedAndUnmappedColumnNames(resultMap, columnPrefix);
```

```java
private void loadMappedAndUnmappedColumnNames(ResultMap resultMap, String columnPrefix) throws SQLException {
    List<String> mappedColumnNames = new ArrayList<>();
    List<String> unmappedColumnNames = new ArrayList<>();
    final String upperColumnPrefix = columnPrefix == null ? null : columnPrefix.toUpperCase(Locale.ENGLISH);
    // 这里没有配置前缀，根据之前的图，定义了ResultMap后，会记录这些已经配置映射的字段。
    final Set<String> mappedColumns = prependPrefixes(resultMap.getMappedColumns(), upperColumnPrefix);
    for (String columnName : columnNames) {
        // 遍历列名，如果在已映射的配置中，那么就加入已经映射的列名数据
        final String upperColumnName = columnName.toUpperCase(Locale.ENGLISH);
        if (mappedColumns.contains(upperColumnName)) {
            mappedColumnNames.add(upperColumnName);
        } else {
            unmappedColumnNames.add(columnName);
        }
    }
    // 生成未映射和已映射的Map
    mappedColumnNamesMap.put(getMapKey(resultMap, columnPrefix), mappedColumnNames);
    unMappedColumnNamesMap.put(getMapKey(resultMap, columnPrefix), unmappedColumnNames);
}
```

如果有没配置在ResultMap中，且Select出来的，那么之后也会按照之前方案一那样，继续往下走，从对象中寻找映射关系。

由于没有未映射的字段，使用自动映射的结果是false。

```java
foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
```

之后继续往下走，使用applyPropertyMappings来创建对象。其用到了propertyMappings，里面包含了字段名，列名，字段的类型和对应的处理器。

![propertyMappings](http://upload-images.jianshu.io/upload_images/292448-2ccc0ef42b243d04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

遍历整个Mappings。

```java
Object value = getPropertyMappingValue(rsw.getResultSet(), metaObject, propertyMapping, lazyLoader, columnPrefix);
```

函数里主要的就是获取这个字段对应的类型处理器，防止类型转换失败，这一部分下次会专门看一下。

```java
final TypeHandler<?> typeHandler = propertyMapping.getTypeHandler();
final String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
return typeHandler.getResult(rs, column);
```

TypeHandler就是一个接口，主要完成的工作就是从Result根据列名，获取相应类型的值，为下一步反射赋值做准备。至于它是怎么决定为什么用这个类型的TypeHandler下次再看。

然后就是给对应字段赋值。

```java
metaObject.setValue(property, value);
```

最后就完成了整个类的赋值。

![赋值](http://upload-images.jianshu.io/upload_images/292448-9271873a13994420.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 总结

大致上，MyBatis完成映射主要是两种方式：
1. 只根据列名，利用自动映射，根据反射类的信息，得到列名和字段之间的关系，使用对应的TypeHandler完成字段的赋值。
2. 使用ResultMap预先定义好映射关系，最后也是根据TypeHandler和反射完成字段的赋值。

就简单的用法来说，两者都可以。在一次会话中，Configuration中的ResultMap关系建立好，在每一次查询的时候就不用再去重新建立了，直接用就行。而自动映射的话，执行过一次后，也会在会话中建立自动映射的缓存，所以没什么差别。但如果复杂的映射的话，就非ResultMap莫属啦。具体可以参考MyBatis文档关于映射的章节，因为目前用不到比较复杂的映射，不做深究了。
