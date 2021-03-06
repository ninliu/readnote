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

EasyMock提供了一个属性允许修改它的行为。它的目的是允许在新版本中使用历史遗留下来的一些行为。当前支持的一些属性是：

* easymock.notThreadSafeByDefault:如果是true，模拟对象默认不是线程安全的。可能的值是true或者false，默认是false；
* easymock.enableThreadSafetyCheckByDefault:如果是true，默认打开线程安全检查。可能的值是true或者false,默认是false；
* easymock.disableClassMocking:不允许类的模拟（只允许接口模拟），可能的值是true或者false，默认是false；

属性能通过两种方法设置：

* 在默认包的类路径下的文件easymock.properties
* 调用EasyMock.setEasyMockProperty。EasyMock类中有一些常量。在代码中设置的属性会覆盖配置文件easymock.properties配置的属性

###对象方法

Object对象的四个方法equals(),hashCode(),toString()和finalize()的行为在EasyMock中创建的模拟对象中不能被修改，即使他们属于被创建的模拟对象的接口的一部分。

###使用方法的部分行为

有时候，我们希望我们的模拟对象爱你个响应某些方法调用，但又不希望去检查他们被调用的频率，他们什么时候被调用甚至他们是否被调用。这个行为可以使用方法andStubReturn(Object value),andStubThrow(Throwable throwable), andStubAnswer(IAnswer<T> answer)和asStub()来定义。下面的代码配置了模拟对象执行voteForRemoval("Document")返回42其他参数返回-1:

>1. expect(mock.voteForRemoval("Document")).andReturn(42); 
>1. expect(mock.voteForRemoval(not(eq("Document")))).andStubReturn(-1);

###复用模拟对象

模拟对象能通过reset(mock)方法重置

如果需要，一个模拟对象还可以通过resetToNice(mock),resetToDefault(mock)和resetToStrict(mock)方法转成其他类型。

##验证

###第一个验证

到这里我们有一个错误没有处理：如果我们描述了行为，我们就想要去验证是否真的被使用了。当前的测试将会通过如果没有任何一个模拟对象的方法被调用。为了检查我们声明的行为是否被使用，执行verify(mock):

>1. @Test 
>1. public void testAddDocument() {
>1.   mock.documentAdded("New Document"); // 2
>1.   replay(mock); // 3
>1.   classUnderTest.addDocument("New Document", new byte[0]);
>1.   verify(mock);
>1. }

如果模拟对象的这个方法并没有被使用，我们会看到下面的异常信息：

>1. java.lang.AssertionError: 
>1.   Expectation failure on verify: 
>1.     documentAdded("New Document"): expected: 1, actual: 0 
>1.       at org.easymock.internal.MocksControl.verify(MocksControl.java:70) 
>1.       at org.easymock.EasyMock.verify(EasyMock.java:536) 
>1.       at org.easymock.samples.ExampleTest.testAddDocument(ExampleTest.java:31) 
>1.       ... 

异常信息列出了所有错失的预期。

###预期一个明确的调用数值

到目前为止，我们的测试只验证一个简单方法的调用。下面的测试将检查是否已存在的文档的添加会去使用适当的参数调用mock.documentChanged()方法。为了确保结果，我们检查这个三次（这只是个例子）：

>1. @Test 
>1. public void testAddAndChangeDocument() {
>1. &nbsp;&nbsp;mock.documentAdded("Document");
>1. &nbsp;&nbsp;  mock.documentChanged("Document");
>1. &nbsp;&nbsp;  mock.documentChanged("Document");
>1. &nbsp;&nbsp;  mock.documentChanged("Document");
>1. &nbsp;&nbsp;  replay(mock);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  verify(mock);
>1. }

为了避免代码mock.documentChanged("Document")的重复，EasyMock提供了一个快捷方法。我们只需要通过方法expectLastCall()返回的对象上执行times(int times)声明执行次数即可。 现在的代码将会是这样：

>1. @Test 
>1. public void testAddAndChangeDocument() {
>1. &nbsp;&nbsp;mock.documentAdded("Document");
>1. &nbsp;&nbsp;  mock.documentChanged("Document");
>1. &nbsp;&nbsp;  expectLastCall().times(3);
>1. &nbsp;&nbsp;  replay(mock);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  classUnderTest.addDocument("Document", new byte[0]);
>1. &nbsp;&nbsp;  verify(mock);
>1. }

如果调用次数过多，我们会得到一个异常信息告诉我们这个方法被调用次数过多了。这个错误会在第一个方法调用触及到这个限制的时候立刻发生：

>1. java.lang.AssertionError:
>1. &nbsp;&nbsp;  Unexpected method call documentChanged("Document"): 
>1. &nbsp;&nbsp;&nbsp;&nbsp;    documentChanged("Document"): expected: 3, actual: 4 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:29) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.internal.ObjectMethodsFilter.invoke(ObjectMethodsFilter.java:44) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at $Proxy0.documentChanged(Unknown Source) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ClassUnderTest.notifyListenersDocumentChanged(ClassUnderTest.java:67) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ClassUnderTest.addDocument(ClassUnderTest.java:26) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ExampleTest.testAddAndChangeDocument(ExampleTest.java:43) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      ... 

如果调用次数少于预期，verify(mock)同样会抛出AssertionError：

>1. java.lang.AssertionError: 
>1. &nbsp;&nbsp;  Expectation failure on verify: 
>1. &nbsp;&nbsp;&nbsp;&nbsp;    documentChanged("Document"): expected: 3, actual: 2 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.internal.MocksControl.verify(MocksControl.java:70) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.EasyMock.verify(EasyMock.java:536) 
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      at org.easymock.samples.ExampleTest.testAddAndChangeDocument(ExampleTest.java:43)
>1. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;      ...

###指定返回值

为了指定返回值，需要使用expect(T value)把预期的方法调用包装起来，然后这个方法的返回结果对象上用方法andReturn(Object reurnValue)声明返回值。

举个例子，我们检查移除文档的流程。如果ClassUnderTest接收到一个移除文档的调用，它将会通过byte voteForRemoval(String title)去询问所有的协作方的投票。正数表示同意移除。如果所有值的和是正数，就会调用documentRemoved(String title)删除这个文档：

>1. @Test 
>1. public void testVoteForRemoval() {
>1.   mock.documentAdded("Document");
>1.   // expect document addition 
>1.   // expect to be asked to vote for document removal, and vote for it 
>1.   expect(mock.voteForRemoval("Document")).andReturn((byte) 42);
>1.   mock.documentRemoved("Document");
>1.   // expect document removal 
>1.   replay(mock); 
>1.   classUnderTest.addDocument("Document", new byte[0]); 
>1.   assertTrue(classUnderTest.removeDocument("Document")); 
>1.   verify(mock); 
>1. }
>1. 
>1. @Test
>1. public void testVoteAgainstRemoval() {
>1.   mock.documentAdded("Document");
>1.   // expect document addition 
>1.   // expect to be asked to vote for document removal, and vote against it 
>1.   expect(mock.voteForRemoval("Document")).andReturn((byte) -42);
>1.   replay(mock);
>1.   classUnderTest.addDocument("Document", new byte[0]);
>1.   assertFalse(classUnderTest.removeDocument("Document"));
>1.   verify(mock);
>1. }

返回值的类型在编译时候就会被检查。举例说，下面的代码不会被编译成功，因为提供的返回值的类型和方法返回值的类型不匹配：

>1. expect(mock.voteForRemoval("Document")).andReturn("wrong type");

除了使用expect(T value)返回对象来设置返回值外，还可以使用expectLastCall()方法的返回对象。替换下面的代码：

>1. expect(mock.voteForRemoval("Document")).andReturn((byte) 42);

可以使用:

>1. mock.voteForRemoval("Document"); 
>1. expectLastCall().andReturn((byte) 42);

这种类型的语句建议在语句太长的情况下使用，因为它并不支持编译时候的类型检查。

###处理异常

为了支持声明和抛出异常（通常是Throwable）,exceptLastCall()和except(T value)返回的对象提供了andThrow(Throwable throwable)方法。这个方法必须在录制阶段调用了模拟对象抛出异常方法的那个方法被调用之后执行。

非受检异常（比如RuntimeException，Error以及他们的子类）不能在方法中被抛出。检查异常只能抛出在那些方法真的抛出异常的地方。

###创建返回值和异常

有时候希望我们的模拟对象在真实的调用的时候返回值或者抛出异常。从EasyMock 2.2开始expectLastCall()和expect(T value)提供了方法andAnswer(IAnswer answer)方法，通过声明一个实现IAnswer接口的实现用来创建返回值和异常。

在IAnswer的回调，通过EasyMock.getCurrentArguments()可以把参数传递给模拟对象。如果你这样使用，向参数重新排序的重构会打断你的测试，这是警告。

一个替代IAnswer的方法是使用andDelegateTo()和andSubDelegateTo()方法。它们允许你把调用委派给一个模拟接口的具体实现，由它来提供响应。这样做的优点是，为IAnswer而在EasMock.getCurrentArguments()方法里面发现的参数现在都会被传递到具体实现中。这是重构安全的。缺点是必须必须手工提供一个模拟行为的实现，而这就是你想使用EasyMock来避免的。如果接口有很多方法，这将是痛苦的。最后，这个具体类的类型不能被静态得和模拟对象做检查。如果因为某些原因，这个特定类并没有实现被代理的方法，在回放阶段你会得到一个异常错误。但是这种情况是相当罕见的。

为了正确理解这两个知识点，下面是个例子：

>1. List&lt;String&gt; l = createMock(List.class); 
>1. 
>1. // andAnswer style 
>1. expect(l.remove(10)).andAnswer(new IAnswer&lt;String&gt;() {
>1.   public String answer() throws Throwable {
>1.     return getCurrentArguments()[0].toString();
>1.   } 
>1. }); 
>1. 
>1. // andDelegateTo style 
>1. expect(l.remove(10)).andDelegateTo(new ArrayList&lt;String&gt;() {
>1.   @Override 
>1.   public String remove(int index) {
>1.     return Integer.toString(index); 
>1.   } 
>1. }); 

###检查模拟对象间的方法执行顺序

到目前为止，我们看到通过EasyMock类的静态方法来配置模拟对象都是单一的一个对象。但是大多数这样的静态方法只是标识出了模拟对象的隐藏控制方法并且委派它。模拟控制器是一个实现了IMockControl接口的对象。

那么，替换下面的代码

>1. IMyInterface mock = createStrictMock(IMyInterface.class);
>1. replay(mock);
>1. verify(mock);
>1. reset(mock);

我们可以使用等效的代码：

>1. IMocksControl ctrl = createStrictControl();
>1. IMyInterface mock = ctrl.createMock(IMyInterface.class);
>1. ctrl.replay();
>1. ctrl.verify();
>1. ctrl.reset();

IMockControl允许不止一个模拟对象，于是我们就能够检查多个模拟对象之间的方法调用顺序。比如，我们为IMyInterface实现了两个模拟对象，而且我们期望按顺序执行mock1.a()和mock2.a(),然后任意次数的执行mock1.c()和mock2.c()，最后再执行mock2.b()和mock1.b()

>1. IMocksControl ctrl = createStrictControl();
>1. IMyInterface mock1 = ctrl.createMock(IMyInterface.class);
>1. IMyInterface mock2 = ctrl.createMock(IMyInterface.class);
>1. mock1.a();
>1. mock2.a();
>1. ctrl.checkOrder(false);
>1. mock1.c();
>1. expectLastCall().anyTimes();
>1. mock2.c();
>1. expectLastCall().anyTimes();
>1. ctrl.checkOrder(true);
>1. mock2.b();
>1. mock1.b();
>1. ctrl.replay();

###灵活的调用次数

为了释放对方法调用次数的限制，有一些附加的方法用来替代times(int count):

times(int min, int max) - 期望在最大和最小值之间
atLeastOnce() - 至少执行一次
anyTimes() - 无限制的任意调用次数

如果没有显示的调用次数指定，默认是预期一次调用。如果你想明确得指定这个数字，可以使用once()或者times(1)

###顺序检查开关

有时候，需要让模拟对象检查一些方法的调用顺序。在录制阶段，你可以通过checkOrder(mock, true)来打开顺序检查和checkOrder(mock,false)来关闭。

这里在严格的模拟和普通模拟对象之间有两个差异：

1. 严格对象在创建之后就是打开顺序检查功能；
1. 严格对象在重置之后也是打开顺序检查功能（查看模拟对象复用）

###参数匹配和灵活的预期

在一个模拟对象上预期一个确切的方法，对象参数默认是通过equals()方法来比较的。这会导致一个问题，举例说我们有下面预期：

>1. String[] documents = new String[] { "Document 1", "Document 2" };
>1. expect(mock.voteForRemovals(documents)).andReturn(42);

如果这个方法使用了相同内容的另外一个数组为参数， 我们得到一个异常，因为数组也是使用equals()方法来对比参数的。

>1. java.lang.AssertionError: 
>1.   Unexpected method call voteForRemovals([Ljava.lang.String;@9a029e):
>1.     voteForRemovals([Ljava.lang.String;@2db19d): expected: 1, actual: 0 
>1.     documentRemoved("Document 1"): expected: 1, actual: 0 
>1.     documentRemoved("Document 2"): expected: 1, actual: 0 
>1.       at org.easymock.internal.MockInvocationHandler.invoke(MockInvocationHandler.java:29) 
>1.       at org.easymock.internal.ObjectMethodsFilter.invoke(ObjectMethodsFilter.java:44) 
>1.       at $Proxy0.voteForRemovals(Unknown Source) 
>1.       at org.easymock.samples.ClassUnderTest.listenersAllowRemovals(ClassUnderTest.java:88) 
>1.       at org.easymock.samples.ClassUnderTest.removeDocuments(ClassUnderTest.java:48) 
>1.       at org.easymock.samples.ExampleTest.testVoteForRemovals(ExampleTest.java:83) 
>1.       ...

在这个例子中，为方法指定数组内容匹配，可以使用EasyMock静态方法aryEq()

>1. String[] documents = new String[] { "Document 1", "Document 2" };
>1. expect(mock.voteForRemovals(aryEq(documents))).andReturn(42);

如果你在调用中要使用匹配器，必须为这个方法调用的参数指定匹配器

下面是一些可用的预定义的参数匹配器：

* eq(X value) 检查真实值和预期值是否相同， 支持原生类型和对象；
* anyBoolean(), anyByte(),anyChar(),anyDouble(),anyFloat(),anyInt(),anyLong(),anyObject(),anyObject(Class clazz),anyShort(),anyString() 匹配任何值，对原生类型和对象
* eq(X value, X delta) 检查真实值和预期值是否在一定的误差范围内相同，只针对float和double类型
* aryEq(X value) 通过使用Arrays.equals()方法来检查真实值和预期值，适用原生类型和状态
* isNull(),isNull(Class clazz) 检查真实值为空，适用所有对象
* notNull(),notNull(Class clazz) 检查真实值不为空，适用对象
* same(X value) 检查真实值是否和预期值相同，适用对象
* isA(Class clazz) 检查真实值是指定类的实例，或者是指定类子类的实例。 Null永远返回false，适用对象
* lt(X value), leq(X value),geq(X value),gt(X value) 真实值是否小于，小于等于，大于等于，大于给定预期值，适用所有的数字类型和能够比较的Comparable对象
* startsWith(String prefix),contains(String substring),endsWith(String substring) 检查是否以预期值开始，包含预期值以及以预期值结尾的字符串，适用所有的String
* matches(String regex),find(String regex) 检查真实值或者它的子字符串匹配给定的正则表达式，适用String
* and(X first, X second) 检查真实值和first和second都满足，适用原生类型和对象
* or(X first, X second) 检查first或者second中有一个满足，适用原生类型和对象
* not(X value) 检查适用的匹配检查器是否不匹配
* cmpEq(X value) 通过使用Comparable.compareTo(X o)来检查真实值是否匹配，适用所有数值型和Comparable对象
* cmp(X value, Comparator<X> comparator, LogicalOperator operator) 使用comparator(actual, value) operator 0来检查，operator是<,<=,>=,>或者=的一种。适用所有对象
* capture(Capture<T> capture), captureXXX(Capture<T> capture) 匹配任何值，并放到capture中便于后面的获取。通过使用and(someMatcher(...), capture(c)) 在某些特殊的方法调用中获取方法参数。也可以通过CaptureType 来告诉给定的capture保留第一个，最后一个还是不用保留任何值。

###定义自己的匹配器

有时候需要自己编写参数匹配器。让我们看下一个参数匹配器，它会去检查一个异常是否有相同的类型和同样的消息。匹配器会这样使用：

>1. IllegalStateException e = new IllegalStateException("Operation not allowed.")
>1. expect(mock.logThrowable(eqException(e))).andReturn(true);

达到这个目的需要两步：定义新的参数匹配器，声明一个静态方法eqException()

新的参数匹配器需要实现org.easymock.IArgumentMatcher接口。这个接口定义了两个方法，matches(Object actual)用于检查真实的参数是否和期望值匹配，appendTo(StringBuffer buffer)用于增加这个参数匹配器的字符串表示到指定的stringbuffer中。下面是具体的实现代码：

>1. import org.easymock.IArgumentMatcher; 
>1. 
>1. public class ThrowableEquals implements IArgumentMatcher { 
>1.   
>1.   private Throwable expected; 
>1.   
>1.   public ThrowableEquals(Throwable expected) {
>1.     this.expected = expected;
>1.   }
>1.   
>1.   public boolean matches(Object actual) {
>1.     if (!(actual instanceof Throwable)) {
>1.       return false;
>1.     }
>1.     
>1.     String actualMessage = ((Throwable) actual).getMessage();
>1.     return expected.getClass().equals(actual.getClass()) &amp;&amp; expected.getMessage().equals(actualMessage);
>1.   }
>1.   
>1.   public void appendTo(StringBuffer buffer) {
>1.     buffer.append("eqException("); 
>1.     buffer.append(expected.getClass().getName()); 
>1.     buffer.append(" with message \""); 
>1.     buffer.append(expected.getMessage()); 
>1.     buffer.append("\"")"); 
>1.   } 
>1. }
>1. 
 
方法eqException用给定的Throwable创建参数匹配器，通过静态方法reportMatcher(IArgumentMatcher matcher)报告给EasyMock框架，并且返回一个有可能在这个方法中使用到的值（通常是0, null或者false ）。第一个尝试看起来是这样的：

>1. public static Throwable eqException(Throwable in) {
>1.   EasyMock.reportMatcher(new ThrowableEquals(in)); 
>1.   return null;
>1. } 

然而，例子中的方法logThrowable接受Throwable的时候能够工作，但是并不能处理RuntimeException的时候无法得到具体的信息。在后面的例子中，我们的代码是无法编译通过的：

>1. IllegalStateException e = new IllegalStateException("Operation not allowed.")
>1. expect(mock.logThrowable(eqException(e))).andReturn(true);

Java 5解决了这个问题，用Throwable为参数来定义一个eqException并返回值，替代的方法是使用泛型继承自Throwable：

>1. public static&lt;T extends Throwable&gt;T eqException(T in) { 
>1.   reportMatcher(new ThrowableEquals(in)); 
>1.   return null; 
>1. }

##高级

###序列化模拟

模拟对象可以在他们生命周期中任何时候被序列化，然而也有一些明显的制约：

1. 所有被用到的匹配器需要被序列化（所有真实的EasyMock都这样）
2. 录制的参数也应该被序列化

###多线程

在录制阶段，一个mock不是线程安全的。所以给定的一个mock（或者多个mock关联到同一个IMockControl）只能在一个线程中录制。然而不同的模拟可以在不同的线程中同时录制。

在回放阶段，模拟默认是线程安全的。但是在录制阶段可以通过方法makeThreadSafe(mock, false)来修改给定的mock对象。这可以防止在某些罕见的情况下发生死锁。

最后在一个模拟对象上调用checkIsUsedInOneThread(mock, true)方法来确保一个mock在一个线程上使用，否则会抛出一个异常。这会确保线程不安全的模拟对象被正确得使用。

###OSGI

EasyMock的jar包可以用作OSGI绑定。它导出了org.easymock, org.easymock.internal和org.easymock.internal.matchers包。但是为了后面导入两个，你需要设置poweruser的值为true(poweruser=true)。这些包是为了扩展EasyMock，所以他们不需要被import。

###向后兼容性

EasyMock 3依然有一个类扩展项目（虽然被遗弃）用来支持迁移早期的EasyMock2到EasyMock3。它是代码而不是二进制兼容的。所以这个代码需要被重新编译。

EasyMock2.1有一个回调特性但是在EasyMock2.2版本中移除了，因为实在太复杂。从EasyMock2.2开始，接口IAnswer提供了会掉的功能。

----