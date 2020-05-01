一、前言
----

相信做过开发的同学，都多多少少写过下面的代码，很长一段时间我一直以为这就是单元测试...

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class UnitTest1 {

    @Autowired
    private UnitService unitService;

    @Test
    public void test() {
        System.out.println("----------------------");
        System.out.println(unitService.sayHello());
        System.out.println("----------------------");
    }
}
```

但这是单元测试嘛？unitService 中可能还依赖了 Dao 的操作；如果是微服务，可能还要起注册中心。那么这个 “单元” 也太大了吧！如果把它称为集成测试，可能更恰当一点，那么有没有可能最小粒度进行单元测试嘛？

单元测试应该是一个带有隔离性的功能测试。在单元测试中，应尽量避免其他类或系统的副作用影响。

单元测试的目标是一小段代码，例如方法或类。方法或类的外部依赖关系应从单元测试中移除，而改为测试框架创建的 mock 对象来替换依赖对象。

单元测试一般由开发人员编写，通过验证或断言目标的一些行为或状态来达到测试的目的。

二、JUnit 框架
----------

JUnit 是一个测试框架，它使用注解来标识测试方法。JUnit 是 Github 上托管的一个[开源项目](https://github.com/junit-team/junit4)。

一个 JUnit 测试指的是一个包含在测试类中的方法，要定义某个方法为测试方法，请使用 @Test 注解标注该方法。该方法执行被测代码，可以使用 JUnit 或另一个 Assert 框架提供的 assert 方法来检查预期结果与实际结果是否一致，这些方法调用通常称为断言或断言语句。

```java
public class UnitTest2 {

    @Test
    public void test() {
        String sayHello = "Hello World";
        Assert.assertEquals("Hello World", sayHello);
    }
}
```

以下是一些常用的 JUnit 注解：

| 注解 | 描述 |
| --- | --- |
| @Test | 将方法标识为测试方法 |
| @Before | 在每次测试之前执行。用于准备测试环境（例如，读取输入数据，初始化类） |
| @After | 每次测试之后执行。用于清理测试环境（例如，删除临时数据，恢复默认值） |
| @BeforeClass | 用于 static 方法，在所有测试开始之前执行一次。它用于执行耗时的活动，例如：连接到数据库 |
| @AfterClass | 用于 static 方法，在完成所有测试之后，执行一次。它用于执行清理活动，例如：与数据库断开连接 |
| @Ignore | 指定要忽略的测试 |
| @Test(expected = Exception.class) | 如果该方法未引发命名异常，则失败 |
| @Test(timeout=100) | 如果该方法花费的时间超过 100 毫秒，则失败 |

以下是一些常用的 Assert 断言：

| 声明 | 描述 |
| --- | --- |
| fail([message]) | 使方法失败。在执行测试代码之前，可用于检查未到达代码的特定部分或测试失败 |
| assertTrue([message，] 布尔条件) | 检查布尔条件是否为真 |
| assertFalse([message，] 布尔条件) | 检查布尔条件是否为假 |
| assertEquals([message，] 预期，实际) | 测试两个值是否相同。注意：对于数组，会检查引用而不是数组的内容 |
| assertNull([message，] 对象) | 检查对象是否为空 |
| assertNotNull([message，] 对象) | 检查对象是否不为空 |
| assertSame([message，] 预期，实际) | 检查两个变量是否引用同一对象 |
| assertNotSame([message，] 预期，实际) | 检查两个变量是否引用了不同的对象 |

三、Mockito 框架
------------

从上面的介绍我们可以认识到，如何减少对外部的依赖才是实践单元测试的关键。而这正是 [Mockito](https://github.com/mockito/mockito) 的使命，Mockito 是一个流行的 mock 框架，可以与 JUnit 结合使用，Mockito 允许我们创建和配置 mock 对象，使用 Mockito 将大大简化了具有外部依赖项的类的测试开发。spring-boot-starter-test 中默认集成了 Mockito，不需要额外引入。

在测试中使用 Mockito，通常会：

*   mock 外部依赖关系并将 mock 对象插入待测代码
*   执行被测代码
*   验证代码是否正确执行  
    ![](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/1153954-20200429112038014-1271464186.png)

### 3.1 使用 Mockito 创建 mock 对象

Mockit o 提供了几种创建 mock 对象的方法：

*   使用静态 mock() 方法
*   使用 @Mock 注解

如果使用 @Mock 注解，则必须触发创建带有 @Mock 注解的对象。使用 MockitoRule 可以做到，它通过调用静态方法 MockitoAnnotations.initMocks(this) 来填充带 @Mock 注解的字段。或者可以使用 @RunWith(MockitoJUnitRunner.class)。

```java
public class UnitTest3 {

    // 触发创建带有 @Mock 注解的对象
    @Rule public MockitoRule mockitoRule = MockitoJUnit.rule();
    // 1. 使用 @Mock 注解创建 mock 对象
    @Mock private UnitDao unitDao;

    @Test
    public void test() {
        // 2. 使用静态 mock() 方法创建 mock 对象
        Iterator iterator = mock(Iterator.class);
        // when...thenReturn / doReturn...when 模拟依赖调用
        when(iterator.next()).thenReturn("hello");
        doReturn(1).when(unitDao).delete(anyLong());
        // 断言
        Assert.assertEquals("hello", iterator.next());
        Assert.assertEquals(new Integer(1), unitDao.delete(1L));
    }
}
```

### 3.2 使用 mock 对象实践单元测试

我们要单元测试的内容，常常包含着对数据库的访问等等，那么我们要如何 mock 掉这部分调用呢？我们可以使用 @InjectMocks 注解创建实例并使用 mock 对象进行依赖注入。

```java
@Service
public class UnitServiceImpl implements UnitService {

    @Autowired
    private UnitDao unitDao;

    @Override
    public String sayHello() {
        Integer delete = unitDao.delete(1L);
        System.out.println(delete);
        return "hello unit";
    }
}
```

```java
@RunWith(MockitoJUnitRunner.class)
public class UnitTest2 {
    
    @Mock
    private UnitDao unitDao;
    @InjectMocks
    private UnitServiceImpl unitService;

    @Test
    public void unitTest() {
        // mock 调用
        when(unitDao.delete(anyLong())).thenReturn(1);
        Assert.assertEquals("hello unit", unitService.sayHello());
    }
}
```

Mockito 还有很多有趣的实践，比如：@Spy 或 spy() 方法、verify() 验证等等，鉴于篇幅原因，读者可自行挖掘。

### 3.3 使用 PowerMock mock 静态方法。

Mockito 也有一些局限性。例如：不能 mock 静态方法和私有方法。有关详细信息，请参阅 [Mockito 限制的常见问题解答](https://github.com/mockito/mockito/wiki/FAQ#what-are-the-limitations-of-mockito)。这个时候我们就要用到 PowerMock，PowerMock 支持 JUnit 和 TestNG，扩展了 EasyMock 和 Mockito 框架，增加了 mock static、final 方法的功能。

首先需要引入 PowerMock 的依赖：

```xml
<!-- PowerMock -->
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <version>2.0.7</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito2</artifactId>
            <version>2.0.7</version>
        </dependency>
```

接下来就能愉快的 mock 静态方法了。

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({StringUtils.class})
public class UnitTest4 {

    @Test
    public void test() {
        mockStatic(StringUtils.class);
        when(StringUtils.getFilename(anyString())).thenReturn("localhost");
        Assert.assertEquals("localhost", StringUtils.getFilename(""));
    }
}
```