Spring中事务分为编程式事务和声明式事务。编程式事务由于需要在代码中硬编码，在实际项目开发中比较少用到。实际开发中用的比较多的就是声明式事务。

声明式事务又分为基于配置的和基于`@Transactional`注解的。

## 1. 基于配置的声明式事务

1. 配置事务管理器
```xml
<bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">       
          <property name="dataSource" ref="dataSource"></property>  
</bean>
```
2. 配置需要加入事务的规则
```xml
<tx:advice id="iccardTxAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="delete*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" no-rollback-for="java.lang.RuntimeException"/>
		<tx:method name="insert*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="add*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="create*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />
		<tx:method name="update*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" />

		<tx:method name="find*" propagation="SUPPORTS" />
		<tx:method name="get*" propagation="SUPPORTS" />
		<tx:method name="select*" propagation="SUPPORTS" />
		<tx:method name="query*" propagation="SUPPORTS" />
	</tx:attributes>
</tx:advice>

<!-- 把事务控制在service层 -->
<aop:config>
	<aop:pointcut id="txPointcut" expression="execution(public * com.zkzong.service.*.*(..))" />
	<aop:advisor pointcut-ref="txPointcut" advice-ref="iccardTxAdvice" />
</aop:config>
```

## 2. 基于@Transactional注解的声明式事务

1. 配置事务管理器
```xml
<!-- 定义事务管理器 -->    
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    
    <property name="dataSource" ref="dataSource" />    
</bean>    
<!--使用注释事务 -->    
<tx:annotation-driven  transaction-manager="transactionManager" />
```
2. 在需要加入事务的方法或者类上添加@Transactional
