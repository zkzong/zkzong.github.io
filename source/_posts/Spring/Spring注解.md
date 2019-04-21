---
title: Spring注解
date: 2019-04-21
categories: Spring
---

### @ModelAttribute
spring mvc @ModelAttribute 接收前台参数问题
直接接收url?后面的参数，如url?id=123&name=456，可直接转换为Pojo

### @RequestBody
接收post请求的body内容--json串，可直接转换为Pojo
依赖jackson-databind包

### @Autowired
byType自动装配

### @Qualifier
当使用@Autowired自动装配Bean时，可能有多个Bean满足装配条件。此时可以使用@Qualifier指定Bean。

### @Inject（Java自带）
和@Autowired一样，@Inject可以用来自动装配属性、方法和构造器；与@Autowired不同的是，@Inject没有required属性。
@Named对应@Qualifier。

### @Component
通用的构造型注解，标识该类为Spring组件。

### @Controller
标识将该类定义为Spring MVC Controller。

### @Repository
标识将该类定义为服务。

### @Service
标识将该类定义为服务。

### @Configuration
作为一个标识告知Spring：这个类将包含一个或多个Spring Bean的定义。

### @Bean
告知Spring这个方法将返回一个对象，该对象应该被注册为Spring应用上下文中的一个Bean。方法名将作为该Bean的ID。

### @Value
加上static则mvc值为空



Spring MVC注解：
1. @PathVariable：url中“/”之后的参数。如：www.zkzong.com/profile/**username**
```java
@Controller
public class ShowProfileController {
    @RequestMapping("/profile/{username}")
    public String showProfile(Model model, @PathVariable("username") String username) {
        model.addAttribute("username", username);
        return "showProfile"; // the view name
    }
}
```
2. @RequestParam：
@RequestParam注解不是严格需要的。在查询参数与方法参数的名称不匹配的时候，@RequestParam是有用的。基于约定，如果处理方法的所有参数没有使用注解的话，将绑定到同名的查询参数上。
+ url中问号之后，以“key=value”的形式出现。如：www.zkzong.com/id?**flag=true**
+ form表单提交，参数名称和表单中控件相同
3. 

Hibernate注解：
1. @DynamicInsert、@DynamicUpdate：
用在数据库表所对应的实体类中。
```java
@Entity
@Table(name = "SYS_USER")
@DynamicInsert(false)
@DynamicUpdate(false)
public class User implements Serializable {
}
```
