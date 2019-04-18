```java
BigDecimal bdTest = new BigDecimal(1.745);
BigDecimal bdTest1 = new BigDecimal(0.745);
bdTest = bdTest.setScale(2, BigDecimal.ROUND_HALF_UP);
bdTest1 = bdTest1.setScale(2, BigDecimal.ROUND_HALF_UP);
System.out.println("bdTest:" + bdTest); // 1.75
System.out.println("bdTest1:" + bdTest1); // 0.74
```
运行以上代码可以看到，***1.745***四舍五入的结果是***1.75***，***0.745***四舍五入的结果是***0.74***。

**原因：**
使用参数为float或double的BigDecimal创建对象会丢失精度。因此强烈建议不要使用参数为float或double的BigDecimal创建对象。
```java
System.out.println(new BigDecimal(1.745)); // 1.74500000000000010658141036401502788066864013671875
System.out.println(new BigDecimal(0.745)); // 0.74499999999999999555910790149937383830547332763671875
```
**解决办法：**
1. 使用BigDecimal(String val)的构造方法创建对象
`new BigDecimal("1.745");`
`new BigDecimal("0.745");`
2. 使用使用BigDecimal的valueOf(double val)方法创建对象
`BigDecimal.valueOf(1.745);`
`BigDecimal.valueOf(0.745);`

[http://stackoverflow.com/questions/12460482/how-to-round-0-745-to-0-75-using-bigdecimal-round-half-up](http://stackoverflow.com/questions/12460482/how-to-round-0-745-to-0-75-using-bigdecimal-round-half-up)
