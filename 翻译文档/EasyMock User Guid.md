#EasyMock用户指导
----
原文地址：[http://easymock.org/user-guide.html](http://easymock.org/user-guide.html)

译者注：本文使用了EasyMock 3.3-SNAPSHOT版本，目前还没有在中央仓库，请自行下载[源码](https://github.com/easymock/easymock/ "源码")并编译安装到本地的maven仓库中
> mvn install


##安装
###要求
EasyMock要求Java 1.5.0及以上版本

为了模仿class，cglib(2.2以上)和(Objenesis以上)必须放在类路径中
###使用maven
EasyMock已经存在Maven的中央仓库中，只需要把下面的依赖配置添加到pom.xml中

> <dependency\>
>
>  <groupId\>org.easymock</groupId\>
>
>  <artifactId\>easymock</artifactId\>
>
>  <version\>3.2</version\>  <font color=red>*--译者注：例子中的代码其实使用的是3.3版本中的几个新类，所以建议用3.3-SNAPSHOT版本* </font>
>
>  <scope\>test</scope\>
>
></dependency\>

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
###使用注解

###EasyMockSupport



















#end