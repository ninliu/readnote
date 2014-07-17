#EasyMock用户指导
----
翻译：龙庄主(http://blog.ninliu.net/)

原文地址：[http://easymock.org/user-guide.html](http://easymock.org/user-guide.html)

译者注：本文使用了EasyMock 3.3-SNAPSHOT版本，目前还没有在中央仓库，请自行下载[源码](https://github.com/easymock/easymock/ "源码")并编译安装到本地的maven仓库中
> mvn install


##安装
###要求
EasyMock要求Java 1.5.0及以上版本

为了模仿class，cglib(2.2以上)和(Objenesis以上)必须放在类路径中
###使用maven
EasyMock已经存在Maven的中央仓库中，只需要把下面的依赖配置添加到pom.xml中

> &lt;dependency&gt;
>
>   &lt;groupId&gt;org.easymock &lt;/groupId&gt;
>
>   &lt;artifactId&gt;easymock &lt;/artifactId&gt;
>
>   &lt;version&gt;3.2 &lt;/version&gt;  <font color=red>*--译者注：例子中的代码其实使用的是3.3版本中的几个新类，所以建议用3.3-SNAPSHOT版本* </font>
>
>   &lt;scope&gt;test &lt;/scope&gt;
>
> &lt;/dependency&gt;

你当然可以使用任何和maven仓库兼容的依赖工具

###独立包
1. 下载[EasyMock压缩包(ZIP)](http://sourceforge.net/projects/easymock/files/EasyMock/3.2/easymock-3.2.zip/download)
2. 把包含的**easymock.jar**放到你的classpath
3. 为了实现模拟类，把cglib和Objenesis都放到类路径中
4. 这个包同时包括了javadoc，测试，源码和例子的jar包


###Android
Since 3.2

*TBD*

##模拟

###第一个模拟对象
现在我们将创建一个测试用例并通过它来了解掌握EasyMock的功能。你也可以查看[例子](https://github.com/easymock/easymock/tree/master/easymock/src/samples/java/org/easymock/samples)和[快速入门](http://easymock.org/getting-started.html)

我们的第一个测试将会检查移除一个不存在的文档不会引起collaborator的通知，下面是一个测试没有使用Mock对象

>import org.junit.*; 
>
>public class ExampleTest {
>
>  private ClassUnderTest classUnderTest; 
>  
>  private Collaborator mock; 
>  
>  @Before 
>  
>  public void setUp() { 
>  
>    classUnderTest = new ClassUnderTest(); 
>    
>    classUnderTest.<font color=green>setListener</font>(mock); 
>    
>  } 
>  
>  @Test 
>  
>  public void testRemoveNonExistingDocument() { 
>  
>    // This call should not lead to any notification 
>    
>    // of the Mock Object: 
>    
>    classUnderTest.<font color=green>removeDocument</font>("<font color=red>Does not exist</font>"); 
>    
>  } 
>  
>} 
>

大多数情况下使用EasyMock，只需要静态导入org.easymock.EasyMock的方法

>1. import static org.easymock.EasyMock.*;
>1. import org.junit.*;
>1. public class ExampleTest {
>1.  private ClassUnderTest classUnderTest;  
>1.  private Collaborator mock;
>1. } 

为了得到Mock对象，我们需要做：

1. 为我们想要模拟的接口创建模拟对象；
2. 录制预期行为；
3. 切换模拟对象到回放状态；

这是第一个例子

>1. @Before
>1. public void setUp() {
>1.   mock = createMock(Collaborator.class); // 1
>1.    classUnderTest = new ClassUnderTest();
>1.    classUnderTest.setListener(mock); 
>1. } 
>1. 
>1. @Test 
>1. public void testRemoveNonExistingDocument() { 
>1.   // 2 (we do not expect anything) 
>1.  replay(mock); // 3 
>1.  classUnderTest.removeDocument("Does not exist"); 
>1. }

在第三步激活后，mock就是Collaborator接口没有预期调用的一个模拟对象。这就说明如果我们改变ClassUnderTest类去调用接口的任何一个方法，模拟对象都会跑出AeertionError错误：

>1. java.lang.AssertionError: 
>1. &nbsp;&nbsp;Unexpected method call documentRemoved("Does not exist"): 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:29) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at org.easymock.internal.ObjectMethodsFilter.invoke(ObjectMethodsFilter.java:44) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at $Proxy0.documentRemoved(Unknown Source) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at org.easymock.samples.ClassUnderTest.notifyListenersDocumentRemoved(ClassUnderTest.java:74) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at org.easymock.samples.ClassUnderTest.removeDocument(ClassUnderTest.java:33) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   at org.easymock.samples.ExampleTest.testRemoveNonExistingDocument(ExampleTest.java:24) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     ...  


###使用注解(since 3.2)

这里有一个优雅和简单的方法去创建你的模拟对象并把它们注入到你的测试类中。下面是上面的例子，现在使用的是注解：

>1. import static org.easymock.EasyMock.*; 
>1. import org.easymock.EasyMockRunner;
>1. import org.easymock.TestSubject; 
>1. import org.easymock.Mock; 
>1. import org.junit.Test; 
>1. import org.junit.runner.RunWith;
>1. 
>1. @RunWith(EasyMockRunner.class) 
>1. public class ExampleTest {
>1. 
>1.   @TestSubject 
>1.   private ClassUnderTest classUnderTest = new ClassUnderTest(); // 2 
>1.   
>1. @Mock 
>1.   private Collaborator mock; // 1 
>1.   
>1.   @Test 
>1.   public void testRemoveNonExistingDocument() { 
>1.  &nbsp;&nbsp;   replay(mock); 
>1.  &nbsp;&nbsp;   classUnderTest.removeDocument("Does not exist"); 
>1.   } 
>1. } 

mock对象在第一步的时候被runner实例化。 然后在第二步的时候被runner设置到了listener字段中。因为所有的初始化工作全都被runner做了，setUp方法就可以被删除。

另外，从EasyMock 3.3开始，如果你在测试中需要使用别的runner， 一个JUnit规则可以被使用。他们有着同样的行为，选择其中之一是个品味问题：

>1. import static org.easymock.EasyMock.*; 
>1. import org.easymock.EasyMockRule; 
>1. import org.easymock.TestSubject; 
>1. import org.easymock.Mock; 
>1. import org.junit.Rule; 
>1. import org.junit.Test; 
>1. 
>1. public class ExampleTest {
>1. 
>1.   @Rule 
>1.   public EasyMockRule mocks = new EasyMockRule(this); 
>1.   
>1.   @TestSubject 
>1.   private ClassUnderTest classUnderTest = new ClassUnderTest(); 
>1.   
>1.   @Mock 
>1.   private Collaborator mock; 
>1.   
>1.   @Test 
>1.   public void testRemoveNonExistingDocument() {
>1.     replay(mock); 
>1.     classUnderTest.removeDocument("Does not exist"); 
>1.   } 
>1. } 

注解有一个可选的要素'type'，用来定义这些模拟对象是nice还是strict模拟。另外一个可选注解是‘name’，允许设置模拟对象的名称，在调用createMock方法的时候被使用，也会在错误消息中被显示。最后一个可选的元素是‘fieldNmae’，用来指定模拟对象将会被注入的目标域的名称。模拟对象会被注入到@TestSubject兼容类型的任何域中。但是两个以上的模拟对象被指派给同一个域是一个错误。在这样的场景中fieldName的唯一性检查将会消除这个歧义。

>1. @Mock(type = MockType.NICE, name = "mock", fieldName = "someField") 
>1. private Collaborator mock; 
>1. 
>1. @Mock(type = MockType.STRICT, name = "anotherMock", fieldName = "someOtherField") 
>1. private Collaborator anotherMock; 

###EasyMockSupport

EasyMockSupport是一个帮助跟踪你的模拟对象的类。你的测试用例可以继承它或者代理。它会自动得注册所有的模拟对象并且批量回放、重置和验证他们而不是明确调用。下面是个例子：

>1. public class SupportTest extends EasyMockSupport {
>1. 
>1.   private Collaborator firstCollaborator;
>1.   private Collaborator secondCollaborator;
>1.   private ClassTested classUnderTest;
>1. 
>1.   @Before
>1.   public void setup() {
>1.     classUnderTest = new ClassTested();
>1.   }
>1. 
>1.   @Test
>1.   public void addDocument() {
>1.     // creation phase
>1.     firstCollaborator = <font color=red>createMock</font>(Collaborator.class);
>1.     secondCollaborator = <font color=red>createMock</font>(Collaborator.class);
>1.     classUnderTest.addListener(firstCollaborator);
>1.     classUnderTest.addListener(secondCollaborator);
>1. 
>1.     // recording phase
>1.     firstCollaborator.documentAdded("New Document");
>1.     secondCollaborator.documentAdded("New Document");
>1.     <font color=red>replayAll()</font>;  <font color=green>**// replay all mocks at once**</font>
>1. 
>1.     // test
>1.     classUnderTest.addDocument("New Document", new byte[0]);
>1.     <font color=red>verifyAll()</font>; <font color=green>**// verify all mocks at once**</font>
>1.   }
>1. }

###严格的模拟(Strict Mocks)

通过EasyMock.createMock()方法创建的模拟对象，方法调用的顺序并不会被检查。如果你需要一个检查方法执行顺序的模拟对象，使用EasyMock.createStrictMock()来创建。

如果在严格模拟对象上有个未预期的方法被调用，异常信息会在期望方法调用的第一个冲突点上显示。verfy(mock)会显示所有没有调用到的方法。

###友好的模拟(Nice Mocks)

createMock()方法创建的模拟，为所有方法不符合期望的调用的默认行为是抛出一个AssertionError错误。如果你偏好一个友好的模拟对象，它允许所有方法的调用并且返回空值（0,null或者false)，那么使用createNickMock()方法。

###局部模拟(Partial mocking)

有时候你只希望模拟一个类的某些方法保留其他方法的行为。这种情况通常发生在你打算去测试一个方法，但是这个方法调用了这个类型其他方法，所以你需要保留被测试方法的正常行为而只是模拟其他的。

在这样的情况下，通常第一件事情是考虑重构，因为这样问题的产生的原因是不好的设计。如果不是这样的情况或者你因为一些开发的限制没有办法这么做，这是一个解决方案：

>1. ToMock mock = createMockBuilder(ToMock.class)
>1. &nbsp;&nbsp;  .addMockedMethod("mockedMethod").createMock();

在这个情况下，只有通过addMockedMethod(s)方法加入的方法才会被模拟（比如上面例子里面的mockedMethod()方法)。其他的方法会保留它们原本的行为。一个特例：为了方便虚方法默认就会被模拟的。

createMockBuilder()返回一个IMockBuilder接口。它包含了方便创建局部模拟的一些方法。详细查看javadoc。

**备注:** 

EasyMock为Object对象的方法(equals, hashCode, toString, finalize)提供了默认的行为。然而，在局部模拟中，如果这些方法不被明确的模拟，他们会有正常的行为而不是EasyMock的默认行为。

###自测试

通过调用某一个构建函数来创建一个模拟是可能的。 这会方便一个类方法需要被测试而且他方法被模拟。 你可以像这样做：

>1. ToMock mock = createMockBuilder(ToMock.class)
>1. .withConstructor(1, 2, 3); // 1, 2, 3 are the constructor parameters

查看例子中的ConstructorCalledMockTest

###替换默认的类实例化器

因为某些原因（通常不支持的JVM），EasyMock没有办法在你的环境中模拟一个类是可能的。这种情况下，类实例化通常使用的是工厂模式。如果失败，需要替换默认的实例化方法：

* 好用的旧类DefaultClasInstantiator很好的处理了序列化类，否则要尝试去猜测使用最好的构造行数和参数。
* 你自己定义的实例化器质需要实现IClassInstantiator接口

使用ClassInstantiatorFactory.setInstantiator()设置这个新的实例化器。通过setDefaultInstantiator()方法设置回默认的。

**重要：**

初始化器在你的测试用例之间是静态被保留的，所以请确保需要的时候重置它。

###序列化一个模拟类

一个模拟对象同样支持序列化，因为它也是序列化类的子类，这个类定义了一个特殊的行为writeObject。这些方法在序列化这个对象的时候仍然会可能被调用而失败。解决办法通常是创建模拟对象的时候调用构造函数。

同样，在不同的class loader反序列化模拟对象也会失败。这没有测试过。

###类模拟的限制

* 为了接口模拟的连贯，EasyMock为类模拟的方法（equals(), toString(),hashCode()和finalize()）提供了内嵌的行为实现。这意味着你不能为这些方法录制自己的行为。 这个限制被认为是避免你不得不去关注这些方法的特点。
* Final方法是不能被模拟的。如果被调用，他们的正常代码将会被执行。
* Private方法不能被模拟。如果被调用，正常代码逻辑会被执行。在局部模拟中，如果你测试的方法调用了一些私有方法， 你需要测试他们因为不能模拟他们。
* 使用[Objenesis](http://objenesis.googlecode.com/svn/docs/index.html)来执行类的实例化，支持的虚拟机列在[这里](http://code.google.com/p/objenesis/wiki/ListOfCurrentlySupportedVMs)

###命名模拟对象

模拟对象在创建的时候可以使用createMock(String name, Class<T> toMock), createStrictMock(String name, Class<T> toMock)或者createNiceMock(String name, Class<T> toMock)命名。这些名字会显示在异常信息中。

##行为

###第二个测试

让我们开始编写第二个测试用例。 如果一个文档被测试类添加了，我们期望调用模拟对象的mock.documentAdded()方法，参数是文档的名称。

>1. @Test
>1. public void testAddDocument(){
>1. mock.documentAdded("New Docuemnt");  //2
>1. replay(mock); //3
>1. clasUnderTest.addDocument("New Document", new byte[0]);
>1. }

那么在录制阶段（调用replay前），模拟对象并不会表现得像一个模拟对象的行为。但是它录下了方法的调用。在执行replay后，它的行为就是一个模拟独享，会检查是否所有期望的方法调用都真实得执行完成了。

如果classUnderTest.addDocument("New Document", new byte[0])调用了预期的方法但是使用了错误的参数。模拟对象也会报出一个AssertionError错误：

>1. java.lang.AssertionError: 
>1. &nbsp;&nbsp;  Unexpected method call documentAdded("Wrong title"): 
>1. &nbsp;&nbsp;&nbsp;&nbsp;    documentAdded("New Document"): expected: 1, actual: 0 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:29) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.internal.ObjectMethodsFilter.invoke(ObjectMethodsFilter.java:44) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at $Proxy0.documentAdded(Unknown Source) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ClassUnderTest.notifyListenersDocumentAdded(ClassUnderTest.java:61) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ClassUnderTest.addDocument(ClassUnderTest.java:28) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ExampleTest.testAddDocument(ExampleTest.java:30) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;      ...

所有错失的期望，没有期望却被执行的调用也会显示（这个例子没有），如果一个方法被执行了多次，模拟对象同样也会报警：

>1. java.lang.AssertionError: 
>1. &nbsp;&nbsp; Unexpected method call documentAdded("New Document"): 
>1. &nbsp;&nbsp;&nbsp;&nbsp;   documentAdded("New Document"): expected: 1, actual: 2 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:29) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at org.easymock.internal.ObjectMethodsFilter.invoke(ObjectMethodsFilter.java:44) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at $Proxy0.documentAdded(Unknown Source) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at org.easymock.samples.ClassUnderTest.notifyListenersDocumentAdded(ClassUnderTest.java:62) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at org.easymock.samples.ClassUnderTest.addDocument(ClassUnderTest.java:29) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;     at org.easymock.samples.ExampleTest.testAddDocument(ExampleTest.java:30) 
 
###改变同样方法调用的行为

改变一个方法的行为也是可行的。 方法times，addReturn和andThrow可以被串成一个链。举个例子，我们定义方法voteForRemoval("Document")的行为如下：

* 前三次执行返回42；
* 接下四次执行抛出RuntimeException;
* 返回一次-42

>1. expect(mock.voteForRemoval("Document"))
>1.   .andReturn((byte) 42).times(3) 
>1.   .andThrow(new RuntimeException(), 4) 
>1.   .andReturn((byte) -42);

###修改EasyMock的默认行为



#end