---
title: Lombok常用注解
date: 2019-07-21
categories: Lombok
---

和其他语言相比， Java经常因为不必要的冗长被批评。 Lombok提供了一系列注解用以在后台生成模板代码，将其从你的类中删除，从而有助于保持你的代码整洁。较少的模板意味着更简洁的代码，更易于阅读和维护。在本文中，我将涉及我经常使用的 Lombok功能，并向你展示如何使用他们生产更清晰、更简洁的代码。

# 1. @NonNull
对方法参数进行 null 检查通常不是一个坏主意，特别是如果该方法形成的 API被其他开发者使用。虽然这些检查很简单，但是他们可能变得冗长，特别是当你有多个参数时。如下所示，额外的代码无助于可读性，并且可能从方法的主要目的分散注意力。

```java
public void nonNullDemo(Employee employee, Account account) {

    if (employee == null) {
        throw new IllegalArgumentException("Employee is marked @NonNull but is null");
    }

    if (account == null) {
        throw new IllegalArgumentException("Account is marked @NonNull but is null");
    }

    // do stuff
}
```

理想情况下，你需要 null 检查——没有干扰的那种。这就是 `@NonNull`发挥作用的地方。通过用`@NonNull`标记参数，Lombok替你为该参数生成 null 检查。你的方法突然变得更加简洁，但没有丢失那些安全性的 null 检查。

```java
public void nonNullDemo(@NonNull Employee employee, @NonNull Account account) {

    // just do stuff

}
```

默认情况下， Lombok会抛出 NullPointerException，如果你愿意，可以配置 Lombok抛出 IllegalArgumentException。我个人更喜欢 IllegalArgumentException，因为我认为它更适合于对参数检查。

# 2. 更简洁的数据类
数据类是 Lombok 真正有助于减少模板代码的领域。在查看该选项前，思考一下我们经常需要处理的模板种类。数据类通常包括以下一种或全部：

- 构造函数（有或没有参数）
- 私有成员变量的 getter 方法
- 私有非 final 成员变量的 setter 方法
- 帮助记录日志的 toString 方法
- equals 和 hashCode（处理相等/集合）

可以通过 IDE 生成以上内容，因此问题不在于编写他们花费的时间。问题是带有少量成员变量的简单类很快会变得非常冗长。让我们看看 Lombok 如何通过处理上述的每一项来减少混乱。

## 3.1 @Getter 和 @Setter
想想下面的 Car 类。当生成 getter 和 setter 时，我们会得到接近 50 行代码来描述一个包含 5 个成员变量的类。

```java
public class Car {

    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;

    public String getMake() {
        return make;
    }

    public void setMake(String make) {
        this.make = make;
    }

    public String getModel() {
        return model;
    }

    public void setModel(String model) {
        this.model = model;
    }

    public String getBodyType() {
        return bodyType;
    }

    public void setBodyType(String bodyType) {
        this.bodyType = bodyType;
    }

    public int getYearOfManufacture() {
        return yearOfManufacture;
    }

    public void setYearOfManufacture(int yearOfManufacture) {
        this.yearOfManufacture = yearOfManufacture;
    }

    public int getCubicCapacity() {
        return cubicCapacity;
    }

    public void setCubicCapacity(int cubicCapacity) {
        this.cubicCapacity = cubicCapacity;
    }

}
```

Lombok可以替你生成 getter和 setter模板。通过对每个成员变量使用 @Getter和 @Setter注解，你最终得到一个等效的类，如下所示：

```java
public class Car {

    @Getter
    @Setter
    private String make;

    @Getter
    @Setter
    private String model;

    @Getter
    @Setter
    private String bodyType;

    @Getter
    @Setter
    private int yearOfManufacture;

    @Getter
    @Setter
    private int cubicCapacity;

}
```

注意，你只能在非 final 成员变量上使用 @Setter，在 final成员变量上使用将导致编译错误。

如果你需要为每个成员变量生成 getter和 setter，你也可以在类级别使用 @Getter和 @Setter，如下所示。

```java
@Getter
@Setter
public class Car {

    private String make;

    private String model;

    private String bodyType;

    private int yearOfManufacture;

    private int cubicCapacity;

}
```

## 3.2 @AllArgsConstructor
数据类通常包含一个构造函数，它为每个成员变量接受参数。IDE 为 Car 生成的构造函数如下所示：

```java
public class Car {

    @Getter
    @Setter
    private String make;

    @Getter
    @Setter
    private String model;

    @Getter
    @Setter
    private String bodyType;

    @Getter
    @Setter
    private int yearOfManufacture;

    @Getter
    @Setter
    private int cubicCapacity;

    public Car(String make, String model, String bodyType, int yearOfManufacture, int cubicCapacity) {
        super();
        this.make = make;
        this.model = model;
        this.bodyType = bodyType;
        this.yearOfManufacture = yearOfManufacture;
        this.cubicCapacity = cubicCapacity;
    }

}
```

我们可以使用 @AllArgsConstructor 注解实现同样功能。 使用 @Getter 和 @Setter、 @AllArgsConstructor可以减少模板，保持类更干净且更简洁。

```java
@AllArgsConstructor
public class Car {

    @Getter
    @Setter
    private String make;

    @Getter
    @Setter
    private String model;

    @Getter
    @Setter
    private String bodyType;

    @Getter
    @Setter
    private int yearOfManufacture;

    @Getter
    @Setter
    private int cubicCapacity;

}
```

还有其他选项用于生成构造函数。 @RequiredArgsConstructor 将创建带有每个 final成员变量参数的构造函数， @NoArgsConstructor
将创建没有参数的构造函数。

## 3.3 @ToString
在你的数据类上覆盖 toString方法是有助于记录日志的良好实践。IDE 为 Car类生成的 toString方法如下所示：

```java
@AllArgsConstructor
public class Car {

    @Getter
    @Setter
    private String make;

    @Getter
    @Setter
    private String model;

    @Getter
    @Setter
    private String bodyType;

    @Getter
    @Setter
    private int yearOfManufacture;

    @Getter
    @Setter
    private int cubicCapacity;

    @Override
    public String toString() {
        return "Car [make=" + make + ", model="
                + model + ", bodyType=" + bodyType + ", yearOfManufacture="
                + yearOfManufacture + ", cubicCapacity=" + cubicCapacity
                + "]";
    }

}
```

我们可以使用 ToString注解替换这个，如下所示：

```java
@ToString
@AllArgsConstructor
public class Car {
    @Getter
    @Setter
    private String make;
    @Getter
    @Setter
    private String model;
    @Getter
    @Setter
    private String bodyType;
    @Getter
    @Setter
    private int yearOfManufacture;
    @Getter
    @Setter
    private int cubicCapacity;
}
```

默认情况下，Lombok 生成包含所有成员变量的 toString 方法。可以通过 exclude 属性 `@ToString(exclude={"someField"},"someOtherField"})` 覆盖行为将某些成员变量排除。

## 3.4 @EqualsAndHashCode
如果你正在将你的数据类和任何类型的对象比较，则需要覆盖 equals和 hashCode 方法。对象的相等是基于业务规则定义的。举个例子，在 Car类中，如果两个对象有相同的 make、 model和 bodyType，我可能认为他们是相等的。如果我使用 IDE 生成 equals 方法检查 make、 model 和 bodyType，它看起来会是这样：

```java
@Override
public boolean equals(Object obj) {
    if (this == obj)
        return true;
    if (obj == null)
        return false;
    if (getClass() != obj.getClass())
        return false;
    Car other = (Car) obj;
    if (bodyType == null) {
        if (other.bodyType != null)
            return false;
    } else if (!bodyType.equals(other.bodyType))
        return false;
    if (make == null) {
        if (other.make != null)
            return false;
    } else if (!make.equals(other.make))
        return false;
    if (model == null) {
        if (other.model != null)
            return false;
    } else if (!model.equals(other.model))
        return false;
    return true;
}
```

等价的 hashCode 实现如下所示：

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + ((bodyType == null) ? 0 : bodyType.hashCode());
    result = prime * result + ((make == null) ? 0 : make.hashCode());
    result = prime * result + ((model == null) ? 0 : model.hashCode());
    return result;
}
```

虽然 IDE 处理了繁重的工作，但我们在类中仍然有大量的模板代码。 Lombok允许我们使用 @EqualsAndHashCode 类注解实现相同的功能，如下所示：

```java
@ToString
@AllArgsConstructor
@EqualsAndHashCode(exclude = {"yearOfManufacture", "cubicCapacity"})
public class Car {
    @Getter
    @Setter
    private String make;
    @Getter
    @Setter
    private String model;
    @Getter
    @Setter
    private String bodyType;
    @Getter
    @Setter
    private int yearOfManufacture;
    @Getter
    @Setter
    private int cubicCapacity;
}
```

默认情况下，@EqualsAndHashCode 会创建包含所有成员变量的 equals 和 hashCode 方法。 exclude选项可用于通知 Lombok排除某些成员变量。在上面的代码片段中。我已经从生成的 equals和 hashCode方法中排除了 yearOfManuFacture 和 cubicCapacity。

## 3.5 @Data
如果你想使数据类尽可能精简，可以使用 @Data 注解。 @Data 是 @Getter、 @Setter、 @ToString、 @EqualsAndHashCode 和 @RequiredArgsConstructor 的快捷方式。

```java
@ToString
@RequiredArgsConstructor
@EqualsAndHashCode(exclude = {"yearOfManufacture", "cubicCapacity"})
public class Car {
    @Getter
    @Setter
    private String make;
    @Getter
    @Setter
    private String model;
    @Getter
    @Setter
    private String bodyType;
    @Getter
    @Setter
    private int yearOfManufacture;
    @Getter
    @Setter
    private int cubicCapacity;
}
```

通过使用 @Data，我们可以将上面的类精简如下：

```java
@Data
public class Car {
    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;
}
```

# 4. 使用 @Buidler 创建对象
建造者设计模式描述了一种灵活的创建对象的方式。 Lombok可以帮你轻松的实现该模式。看一个使用简单 Car 类的示例。假设我们希望可以创建各种 Car对象，但我们希望在创建时设置的属性具有灵活性。

```java
@AllArgsConstructor
public class Car {
    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;
    private List<LocalDate> serviceDate;
}
```

假设我们要创建一个 Car，但只想设置 make和 model。在 Car上使用标准的全参数构造函数意味着我们只提供 make和 model并设置其他参数为 null。

```java
Car2 car2 = new Car2("Ford", "Mustang", null, null, null, null);
```

这可行但并不理想，我们必须为我们不感兴趣的参数传递 null。我们可以创建一个只接受 make和 model的构造函数来避开这个问题。这是一个合理的解决方法，但不够灵活。如果我们有许多不同的字段排列，我们怎么来创建一个新 Car？最终我们得到了一堆不同的构造函数，代表了我们可以实例化 Car的所有可能方式。

解决该问题的一种干净、灵活的方式是使用建造者模式。 Lombok通过 @Builder 注解帮你实现建造者模式。当你使用 @Builder 注解 Car 类时， Lombok会执行以下操作：

- 添加一个私有构造函数到 Car
- 创建一个静态的 CarBuilder 类
- 在 CarBuilder 中为 Car 中的每个成员创建一个 setter风格方法
- 在 CarBuilder中添加创建 Car 的新实例的建造方法

CarBuilder上的每个 setter 风格方法返回自身的实例（ CarBuilder）。这允许你进行方法链式调用并为对象创建提供流畅的 API。让我们看看它如何使用。

```java
Car muscleCar = Car.builder().make("Ford")
        .model("mustang")
        .bodyType("coupe")
        .build();
```

现在只使用 make 和 model 创建 Car 比之前更简洁了。只需在 Car 上简单的调用生成的 builder 方法获取 CarBuilder 实例，然后调用任何我们感兴趣的 setter 风格方法。最后，调用 build 创建 Car 的新实例。

另一个值得一提的方便的注解是 @Singular。默认情况下，Lombok 为集合创建使用集合参数的标准的 setter 风格方法。在下面的例子中，我们创建了新的 Car 并设置了服务日期列表。

```java
@Builder
public class Car {
    private String make;
    private String model;
    private String bodyType;
    private int yearOfManufacture;
    private int cubicCapacity;
    @Singular
    private List<LocalDate> serviceDate;
}
```

向集合成员变量添加 @Singular 将提供一个额外的方法，允许你向集合添加单个项。

```java
Car muscleCar3 = Car.builder()
        .make("Ford")
        .model("mustang")
        .serviceDate(LocalDate.of(2016, 5, 4))
        .build();
```

现在我们可以添加单个服务日期，如下所示：

```java
Car muscleCar3 = Car.builder()
        .make("Ford")
        .model("mustang")
        .serviceDate(LocalDate.of(2016, 5, 4))
        .build();
```

这是一个有助于在创建对象期间处理集合时保持代码简洁的快捷方法。

# 5. 日志
Lombok另一个伟大的功能是日志记录器。如果没有 Lombok，要实例化标准的 SLF4J日志记录器，通常会有以下内容：

```java
public class SomeService {
    
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);

    public void doStuff() {
        log.debug("doing stuff....");
    }
}
```

这些日志记录器很沉重，并为每个需要日志记录的类添加了不必要的混乱。值得庆幸的是 Lombok提供了一个为你创建日志记录器的注解。你要做的所有事情就是在类上添加注解，这样就可以了。

```java
@Slf4j
public class SomeService {
    public void doStuff() {
        log.debug("doing stuff....");
    }
}
```

我在这里使用了 @SLF4J注解，但 Lombok能为几乎所有通用 Java日志框架生成日志记录器。有关更多日志记录器的选项，请参阅文档。

我非常喜欢 Lombok 的一点是它的不侵入性。如果你决定在使用如 @Getter、 @Setter 或 @ToString 时也想要自己的方法实现，你的方法将总是优先于 Lombok。它允许你在大多数时间使用 Lombok，但在你需要的时候仍有控制权。