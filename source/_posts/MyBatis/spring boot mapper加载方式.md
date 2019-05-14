---
title: spring boot mapper加载方式
date: 2019-05-14
categories: MyBatis
---

1. 引用xml配置
2. 使用class类配置
3. 使用starter配置

    3.1 @Mapper
    
    3.2 @MapperScan(basePackages = "com.gomefinance.cif.mapper")
mybatis.mapperLocations=classpath:mapper/*.xml

http://blog.csdn.net/mickjoust/article/details/51646658