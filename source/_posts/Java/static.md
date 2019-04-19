---
title: static
date: 2019-04-18
categories: Java
---

```java
public class StaticSuper {
    static {
        System.out.println("super static block");
    }

    StaticSuper() {
        System.out.println("super constructor");
    }
}

public class StaticTests extends StaticSuper {
    static int rand;
    static {
        rand = (int) Math.random() * 6;
        System.out.println("static block " + rand);
    }

    StaticTests() {
        System.out.println("constructor");
    }

    public static void main(String[] args) {
        System.out.println("in main");
        StaticTests st = new StaticTests();
    }
}
```

输出：
```
super static block
static block 0
in main
super constructor
constructor
```
