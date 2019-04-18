在SpringMVC的Controller中使用`@ModelAttribute`时，其位置包括下面三种：
+ 应用在方法上
+ 应用在方法的参数上
+ 应用在方法上，并且方法也使用了`@RequestMapping`

## 应用在方法上
首先说明一下，被`@ModelAttribute`注解的方法会在`Controller`每个方法执行之前都执行，因此对于一个`Controller`中包含多个URL的时候，要谨慎使用。

1）使用`@ModelAttribute`注解无返回值的方法
```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeController {

    @ModelAttribute
    public void myModel(@RequestParam(required = false) String abc, Model model) {
        model.addAttribute("attributeName", abc);
    }

    @RequestMapping(value = "/method")
    public String method() {
        return "method";
    }
}
```
这个例子，在请求`/modelattribute/method?abc=aaa`后，会先执行`myModel`方法，然后接着执行`method`方法，参数`abc`的值被放到`Model`中后，接着被带到`method`方法中。

当返回视图`/modelattribute/method`时，`Model`会被带到页面上，当然你在使用`@RequestParam`的时候可以使用`required`来指定参数是否是必须的。

如果把`myModel`和`method`合二为一，代码如下，这也是我们最常用的方法：
```java
@RequestMapping(value = "/method")
public String method(@RequestParam(required = false) String abc, Model model) {
	model.addAttribute("attributeName", abc);
	return "method";
}
```

2）使用`@ModelAttribute`注解带有返回值的方法
```java
@ModelAttribute
public String myModel(@RequestParam(required = false) String abc) {
	return abc;
}

@ModelAttribute
public Student myModel(@RequestParam(required = false) String abc) {
	Student student = new Student(abc);
	return student;
}

@ModelAttribute
public int myModel(@RequestParam(required = false) int number) {
	return number;
}
```
对于这种情况，返回值对象会被默认放到隐含的`Model`中，在`Model`中的`key`为**`返回值首字母小写`**，`value`为返回的值。

上面3种情况等同于：
```
model.addAttribute("string", abc);
model.addAttribute("int", number);
model.addAttribute("student", student);
```
> 在jsp页面使用`${int}`表达式时会报错：`javax.el.ELException: Failed to parse the expression [${int}]`。
> 解决办法：
> 在tomcat的配置文件`conf/catalina.properties`添加配置`org.apache.el.parser.SKIP_IDENTIFIER_CHECK=true`

如果只能这样，未免太局限了，我们很难接受`key`为`string`、`int`、`float`等等这样的。

想自定义其实很简单，只需要给`@ModelAttribute`添加`value`属性即可，如下：
```java
@ModelAttribute(value = "num")
public int myModel(@RequestParam(required = false) int number) {
	return number;
}
```
这样就相当于`model.addAttribute("num", number);`。

## 使用@ModelAttribute注解的参数

```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeParamController {

    @ModelAttribute(value = "attributeName")
    public String myModel(@RequestParam(required = false) String abc) {
        return abc;
    }

    @ModelAttribute
    public void myModel3(Model model) {
        model.addAttribute("name", "zong");
        model.addAttribute("age", 20);
    }

    @RequestMapping(value = "/param")
    public String param(@ModelAttribute("attributeName") String str,
                       @ModelAttribute("name") String str2,
                       @ModelAttribute("age") int str3) {
        return "param";
    }
}
```
从代码中可以看出，使用`@ModelAttribute`注解的参数，意思是从前面的`Model`中提取对应名称的属性。

这里提及一下`@SessionAttributes`的使用：
> + 如果在类上面使用了`@SessionAttributes("attributeName")`注解，而本类中恰巧存在`attributeName`，则会将`attributeName`放入到`session`作用域。
> + 如果在类上面使用了`@SessionAttributes("attributeName")`注解，SpringMVC会在执行方法之前，自动从`session`中读取`key`为`attributeName`的值，并注入到`Model`中。所以我们在方法的参数中使用`ModelAttribute("attributeName")`就会正常的从`Model`读取这个值，也就相当于获取了`session`中的值。
> + 使用了`@SessionAttributes`之后，Spring无法知道什么时候要清掉`@SessionAttributes`存进去的数据，如果要明确告知，也就是在方法中传入`SessionStatus`对象参数，并调用`status.setComplete`就可以了。

## 应用在方法上，并且方法上也使用了`@RequestMapping`

```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeController {

    @RequestMapping(value = "/test")
    @ModelAttribute("name")
    public String test(@RequestParam(required = false) String name) {
        return name;
    }
}
```
这种情况下，返回值String（或者其他对象）就不再是视图了，而是放入到`Model`中的值，此时对应的页面就是`@RequestMapping`的值`test`。
如果类上有`@RequestMapping`，则视图路径还要加上类的`@RequestMapping`的值，本例中视图路径为`modelattribute/test.jsp`。
