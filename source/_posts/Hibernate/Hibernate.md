# 1. Hibernate核心组件
> Configuration
> - Database Connection
> - Class Mapping Setup
> SessionFactory
> Session
> Transaction
> Query
> Criteria

# 2. 注解
注解可以放在成员变量或者getter方法上。
@Entity
@Table
@Id @GeneratedValue
@Column
# 注解和xml的区别

# 3. HQL分页
Query setFirstResult(int startPosition)
Query setMaxResults(int maxResult)

# 4. Criteria查询
1. 最简单的criteria查询如下，它会查处所有Employee记录。
```java
Criteria cr = session.createCriteria(Employee.class);
List results = cr.list();
```
...
2. 使用Restrictions添加条件进行Criteria查询
```java
Criteria cr = session.createCriteria(Employee.class);
cr.add(Restrictions.eq("salary", 2000));
List results = cr.list();
```
更多例子如下：
```java
Criteria cr = session.createCriteria(Employee.class); 

// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000)); 

// To get records having salary less than 2000
cr.add(Restrictions.lt("salary", 2000)); 

// To get records having fistName starting with zara
cr.add(Restrictions.like("firstName", "zara%")); 

// Case sensitive form of the above restriction. CHAPTER 14
cr.add(Restrictions.ilike("firstName", "zara%")); 

// To get records having salary in between 1000 and 2000
cr.add(Restrictions.between("salary", 1000, 2000)); 

// To check if the given property is null
cr.add(Restrictions.isNull("salary")); 

// To check if the given property is not null
cr.add(Restrictions.isNotNull("salary")); 

// To check if the given property is empty
cr.add(Restrictions.isEmpty("salary")); 

// To check if the given property is not empty
cr.add(Restrictions.isNotEmpty("salary"));
```
使用LogicalExpression创建AND或OR条件查询
```java
Criteria cr = session.createCriteria(Employee.class); 

Criterion salary = Restrictions.gt("salary", 2000);
Criterion name = Restrictions.ilike("firstNname","zara%");

// To get records matching with OR condistions 
LogicalExpression orExp = Restrictions.or(salary, name);
cr.add( orExp );

// To get records matching with AND condistions 
LogicalExpression andExp = Restrictions.and(salary, name); 
cr.add( andExp ); 
List results = cr.list();
```
分页方法
> public Criteria setFirstResult(int firstResult)
> public Criteria setMaxResults(int maxResults)

对结果排序
```java
Criteria cr = session.createCriteria(Employee.class); 
// To get records having salary more than 2000 cr.add(Restrictions.gt("salary", 2000)); 

// To sort records in descening order 
crit.addOrder(Order.desc("salary")); 

// To sort records in ascending order 
crit.addOrder(Order.asc("salary")); 

List results = cr.list();
```
# 5. 批量处理
首先设置hibernate.jdbc.batch_size为需要批量处理的数字。
其次修改代码：
```java
'''添加功能'''
Session session = SessionFactory.openSession(); 
Transaction tx = session.beginTransaction(); 
for ( int i=0; i<100000; i++ ) { 
    Employee employee = new Employee(.....); 
    session.save(employee); 
    if( i % 50 == 0 ) { // Same as the JDBC batch size 
        //flush a batch of inserts and release memory: 
        session.flush(); 
        session.clear(); 
    } 
} 
tx.commit(); session.close();
```

```java
'''修改功能'''
Session session = sessionFactory.openSession(); 
Transaction tx = session.beginTransaction();
ScrollableResults employeeCursor = session.createQuery("FROM EMPLOYEE") .scroll(); 

int count = 0; 

while ( employeeCursor.next() ) { 
    Employee employee = (Employee) employeeCursor.get(0); 
    employee.updateEmployee(); 
    seession.update(employee); 
    if ( ++count % 50 == 0 ) { 
        session.flush(); session.clear(); 
    } 
} 
tx.commit(); 
session.close();
```
