- 测试方法必须使用@Test修饰
- 测试方法必须使用public void修饰
- 新建test源代码目录来存放测试代码
- 测试类的包名应该和被测试类的报名保持一致
- 测试单元中的每个方法必须可以独立测试，测试方法之间不能有任何依赖
- 测试类一般使用Test作为后缀
- 测试方法一般使用test作为前缀

JUnit4中提供的常用注解大致包括：

- @BeforeClass
- @AfterClass
- @Before
- @After
- @Test
- @Ignore
- @RunWith

注解的执行顺序和作用：

- @Test将一个普通的方法修饰为一个测试方法
- @BeforeClass会在所有方法运行前执行，static修饰
- @AfterClass会在所有方法运行结束后执行，static修饰
- @Before会在每个测试方法运行前执行一次
- @After会在每个测试方法运行后被执行一次
