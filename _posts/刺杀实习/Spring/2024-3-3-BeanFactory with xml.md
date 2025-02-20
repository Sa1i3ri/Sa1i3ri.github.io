---
categories: [课外笔记,Spring]
tag: [课外笔记,Spring]
---

## BeanFactory

Spring提供了两种容器类型：BeanFactory和ApplicationContext。

- BeanFactory
  - 基础类型IoC容器，提供完整的IoC服务支持。
  - 如果没有特殊指定，默认采用延迟初始化策略（lazy-load)，即只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。
  - 所以，相对来说，容器启动初期速度较快，所需要的资源有限。
- ApplicationContext
  - ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等。
  - ApplicationContext所管理 的对象，在该类型容器启动之后，默认全部初始化并绑定完成。
  - 所以，相对于BeanFactory来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容 器启动时间较之BeanFactory也会长一些。

![image-20240303195043415](https://salieri-typora.oss-cn-shanghai.aliyuncs.com/img/markdown/image-20240303195043415.png)

## BeanFactory with XML

### 1 `<beans>`

`<beans>`是XML配置文件中最顶层的元素，它下面可以包含0或者1个`<description>`和多个`<bean>`以及`<import>`或者`<alias>`，如图所示

![image-20240303195427852](https://salieri-typora.oss-cn-shanghai.aliyuncs.com/img/markdown/image-20240303195427852.png)

`<beans>`作为所有的“统帅”，它拥有相应的属性（attribute）对所辖的`<bean>`进行统一的默认行为设置，包括如下几个。 

- default-lazy-init
- default-autowire
- default-dependency-check
- default-init-method
- default-destroy-method

### 2 `<description>`、`<import>`和`<alias>`

之所以把这几个元素放到一起讲解，是因为通常情况下它们不是必需的。

- `<description>`
  - 可以通过在配置的文件中指定一些描述性的信息。通常情况下，该元素是省略的。
- `<import>`
  - 通常情况下，可以根据模块功能或者层次关系，将配置信息分门别类地放到多个配置文件中。
  - 在想加载主要配置文件，并将主要配置文件所依赖的配置文件同时加载时，可以在这个主要的配置文件中通过`<import>`元素对其所依赖的配置文件进行引用。
  - 比如，如果A.xml中的定义可能依赖 B.xml中的某些定义，那么就可以在A.xml中使用`<import>`将B.xml引入到A.xml，以类似于 `<import resource="B.xml"/>`的形式。
- `<alias>`
  - 可以通过为某些起一些“外号”（别名），通常情况下是为了减少输入。比如， 假设有个`<bean>`，它的名称为dataSourceForMasterDatabase，你可以为其添加一个`<alias>`，像`<alias name="dataSourceForMasterDatabase"alias="masterDataSource"/>`这样。以后通过 dataSourceForMasterDatabase或者masterDataSource来引用这个`<bean>`都可以，只要你觉得方便就行。 

### 3. `<bean>`

每个对象作为个体，在Spring的XML配置文件中是与 元素一一对应的。如下代码演示了最基础的对象配置 形式： 

```xml
<bean id="diNewsListener"class="..impl.DowJonesNewsListener">
</bean>
```

或者

`<bean id="diNewsListener"class="..impl.DowJonesNewsListener"/>`

#### 1. id

- 通常，每个注册到容器的对象都需要一个唯一标志来将其与“同处一室”的“兄弟们”区分开来。
- 有些情况下，id可以省略，比如后面会提到的内部`<bean>`以及不需要根据 beanName明确依赖关系的场合等。

除了可以使用id来指定在容器中的标志，还可以使用name属性来指定别名 （alias）。

```xml
<bean id="djNewsListener" 
 name="/news/djNewsListener,dowJonesNewsListener"  
 class="..impl.DowJonesNewsListener"> 
</bean> 
```

与id属性相比，name属性的灵活之处在于，name可以使用id不能使用的一些字符，比如/。

而且还可以通过逗号、空格或者冒号分割指定多个name。name的作用跟使用`<alias>`为id指定多个别名基本相同。

#### 2. class

每个注册到容器的对象都需要通过元素的class属性指定其类型，否则，容器可不知道这个对象到底是何方神圣。

仅在少数情况下不需要指定，如使用抽象配置模板的情况下。

### 4. 注入表达

对象之间需要相互协 作，在横向上它们存在一定的依赖性。

#### 1.  构造方法注入

需要使用`<constructor-arg>`。正常情况下，如以下代码所示：

```xml
<bean id="diNewsProvider"class="..FXNewsProvider">
	<constructor-arg>
		<ref bean="djNewsListener"/>
	</constructor-arg>
	<constructor-arg>
		<ref bean="djNewsPersister"/>
	</constructor-arg>
</bean>
```

Spring也支持配置简写形式，如以下代码所示：

```xml
<bean id="djNewsProvider"class="..FXNewsProvider">
	<constructor-arg ref="djNewsListener"/>
	<constructor-arg ref="djNewsPersister"/>
</bean>
```

##### 1. type

```java
public class MockBusinessobject{
    private String dependencyl;
	private int dependency2;
    public MockBusinessobject(String dependency){
  		this.dependency1 dependency;  
	}
    public MockBusinessobject(int dependency){
    	this.dependency2 dependency;
	}

	@Override
	public String tostring(){
        return new ToStringBuilder(this)
	.append("dependency1",dependency1)
	.append("dependency2",dependency2).tostring();
    }
}
```

该类声明了两个构造方法，分别都只是传入一个参数，且参数类型不同。这时，我们可以进行配置，如以下代码所示：

```xml
<bean id="mockBO"class="..MockBusinessobject">
	<constructor-arg>
		<va1ue>111111</va1ue>
	</constructor-arg>
</bean>
```

Spring调用的是第一个构造方法

通过指定构造方法的参数类型来调用第一个方法

```xml
<bean id="mockBO"class="..MockBusinessobject">
	<constructor-arg type="int">
		<va1ue>111111</va1ue>
	</constructor-arg>
</bean>
```

##### 2. index

```java
public class MockBusinessObject { 
	private String dependency1; 
	private String dependency2; 
	public MockBusinessObject(String dependency1,String    dependency2) { 
		this.dependency1 = dependency1; 
		this.dependency2 = dependency2; 
 } 
 ... 
 	@Override 
	public String toString() { 
		return new ToStringBuilder(this) ➥ 
		.append("dependency1", dependency1) ➥ 
		.append("dependency2", dependency2).toString(); 
 	} 
} 
```

如此配置：

```xml
<bean id="mockBo"class=".MockBusinessobject">
	<constructor-arg value="11111"/>
	<constructor-arg value="22222"/>
</bean>
```

那么，"11111"是第1个参数，"22222"是第2个参数

```xml
<bean id="mockBO"class="..MockBusinessobject">
	<constructor-arg index="1"value="11111"/>
	<constructor-arg index="0"value="22222"/>
</bean>
```

这样，"11111"是第2个参数，"22222"是第1个参数

#### 2. setter方法注入

与构造方法注入可以使用注入配置相对应，Spring为setter方法注入提供了`<property>`元素。

`<property>`有一个name属性（attribute），用来指定该`<property>`将会注入的对象所对应的实例变量名称。之后通过value或者ref属性或者内嵌的其他元素来指定具体的依赖对象引用或者值，如以下代码所示：

```xml
<bean id="djNewsProvider" class="..FXNewsProvider">  
	<property name="newsListener"> 
		<ref bean="djNewsListener"/> 
	</property> 
	<property name="newPersistener"> 
		<ref bean="djNewsPersister"/> 
	</property> 
</bean> 
```

以上配置形式还可以简化为如下形式： 

```xml
<bean id="djNewsProvider" class="..FXNewsProvider">  
	<property name="newsListener" ref="djNewsListener"/> 
	<property name="newPersistener" ref="djNewsPersister"/> 
</bean> 
```

#### 3. `<property>`和`<constructor-arg>`中可用的配置项 

##### 1. `<value>`

可以通过value为主体对象注入简单的数据类型，不但可以指定String类型的数据， 而且可以指定其他Java语言中的原始类型以及它们的包装器（wrapper）类型，比如int、Integer等。

```xml
<constructor-arg> 
	<value>111111</value> 
</constructor-arg> 
<property name="attributeName"> 
	<value>222222</value> 
</property> 
```

简化形式：

```xml
<constructor-arg value="111111"/> 
<property name="attributeName" value="222222"/> 
```



##### 2. `<ref>`

使用ref来引用容器中其他的对象实例，可以通过ref的local、parent和bean属性来 指定引用的对象的beanName是什么。

```xml
<constructor-arg> 
 <ref local="djNewsPersister"/> 
</constructor-arg> 
或者 
<constructor-arg> 
 <ref parent="djNewsPersister"/> 
</constructor-arg> 
或者 
<constructor-arg> 
 <ref bean="djNewsPersister"/> 
</constructor-arg> 
```

local、parent和bean的区别在于： 

- local只能指定与当前配置的对象在同一个配置文件的对象定义的名称（可以获得XML解析器 的id约束验证支持）
- parent则只能指定位于当前容器的父容器中定义的对象引用
- bean则基本上通吃，所以，通常情况下，直接使用bean来指定对象引用就可以了。 

> 注意 BeanFactory可以分层次（通过实现HierarchicalBeanFactory接口），容器A在初始化的时候，可以首先加载容器B中的所有对象定义，然后再加载自身的对象定义，这样，容器B就成为了容器A的父容器，容器A可以引用容器B中的所有对象定义： 
> BeanFactory parentContainer = new XmlBeanFactory(new ClassPathResource("父容器配置文件路
> 径")); 
> BeanFactory childContainer  = new XmlBeanFactory(new ClassPathResource("子容器配置文件路
> 径"),parentContainer); 
> childContainer中定义的对象，如果通过parent指定依赖，则只能引用parentContainer中的对象定义。 

`<ref>`的定义为`<!ELEMENT ref EMPTY>`，也就是说，它下面没有其他子元素可用了

##### 3. `<idref>`

如果要为当前对象注入**所依赖的对象的名称**，**而不是引用**，那么通常情况下，可以使用`<value>`来达到这个目的，使用如下形式：

```xml
<property name="newsListenerBeanName"> 
 <value>djNewsListener</value> 
</property> 
```

但这种场合下，使用idref才是最为合适的。因为使用idref，容器在解析配置的时候就可以**帮你检查这个beanName到底是否存在**，而不用等到运行时才发现这个beanName对应的对象实例不存在。

```xml
<property name="newsListenerBeanName"> 
	<idref bean="djNewsListener"/> 
</property> 
```

##### 4.  内部`<bean>`

使用内嵌的`<bean>`，将这个私有的对象定义仅局限在当前对象。

```xml
<bean id="djNewsProvider" class="..FXNewsProvider"> 
 <constructor-arg index="0"> 
<bean class="..impl.DowJonesNewsListener"> 
</bean> 
 </constructor-arg> 
 <constructor-arg index="1"> 
<ref bean="djNewsPersister"/> 
 </constructor-arg> 
</bean>
```

这样，该对象实例就只有当前的djNewsProvider可以使用，其他对象无法取得该对象的引用。

##### 5. `<list>`

```java
public class MockDemoObject 
{ 
private List param1; 
private String[] param2; 
 ... 
 // 相应的setter和getter方法 
 ... 
} 
```

```xml
<property name="param1"> 
 <list> 
<value> something</value> 
<ref bean="someBeanName"/> 
<bean class="..."/> 
 </list> 
</property> 
<property name="param2"> 
 <list> 
<value>stringValue1</value> 
<value>stringValue2</value> 
 </list> 
</property>
```

##### 6. `<set>`

```java
public class MockDemoObject 
{ 
private Set valueSet; 
 // 必要的setter和getter方法 
 ... 
} 
```

```xml
<property name="valueSet"> 
 <set> 
<value> something</value> 
<ref bean="someBeanName"/> 
<bean class="..."/> 
<list> 
  ... 
</list> 
 </set> 
</property> 
```

##### 7. `<map>`

```java
public class MockDemoObject 
{ 
private Map mapping; 
 // 必要的setter和getter方法 
 ... 
} 
```

```xml
<property name="mapping"> 
 <map> 
<entry key="strValueKey"> 
  <value>something</value> 
</entry> 
<entry> 
  <key>objectKey</key> 
  <ref bean="someObject"/> 
</entry> 
<entry key-ref="lstKey"> 
  <list> 
   ... 
  </list> 
</entry> 
... 
 </map> 
</property>
```

简化：

```xml
<property name="valueSet"> 
 <map> 
<entry key="strValueKey" value="something"/> 
<entry key-ref="" value-ref="someObject"/> 
<entry key-ref="lstKey"> 
  <list> 
   ... 
  </list> 
</entry> 
... 
 </map> 
</property> 
```

##### 8. `<null/>`

对于String类型来说，如果通过value以这样的方式指定注入， 即`<value></value>`，那么，得到的结果是""，而不是null。所以，如果需要为这个string对应的值注入null的话，请使用`<null/>`。

#### 4. depends-on

通常情况下，可以直接通过之前提到的所有元素，来显式地指定bean之间的依赖关系。这样，容器在初始化当前bean定义的时候，会根据这些元素所标记的依赖关系，首先实例化当前bean定义所依 赖的其他bean定义。

但是，如果某些时候，我们**没有通过类似的元素**明确指定对象A依赖于对象B的话，如何让容器在实例化对象A之前首先实例化对象B呢？

```xml
<bean id="classAInstance" class="...ClassA" depends-on="configSetup"/> 
<bean id="configSetup" class="SystemConfigurationSetup"/> 
```

如果说ClassA拥有多个类似的非显式依赖关系，那么，你可以在ClassA的depends-on中通过逗号分割各个beanName，如下代码所示：  

```xml
<bean id="classAInstance" class="...ClassA" depends-on="configSetup,configSetup2,..."/> 
```

#### 5. autowire

spirng还提供了根据bean定义的某些特点将相互依赖的某些bean直接**自动绑定**的功能

Spring提供了5种自动绑定模式，即no、byName、byType、constructor和autodetect

注意：

- 手工明确指定的绑定关系总会覆盖自动绑定模式的行为。
- 自动绑定对  “原生类型、String类型和Classes类型”以及“这些类型的数组”应用自动绑定是无效的。 

`<beans>`有一个default-autowire属性，它可以帮我们省去为多个`<bean>`单独设置autowire属性的麻烦，default-autowire的默认值为no，即不进行自动绑定。

```xml
<beans default-autowire="byType"> 
<bean id="..." class="..."/> 
 ... 
</beans> 
```

##### 1. no

容器**默认**的自动绑定模式，也就是不采用任何形式的自动绑定，完全依赖手工明确配置各个bean之间的依赖关系

##### 2. byName

按照类中声明的**实例变量的名称**，与XML配置文件中声明的bean定义的**beanName的值**进行匹配， 相匹配的bean定义将被自动绑定到当前实例变量上。

```java
public class Foo 
{ 
 private Bar emphasisAttribute; 
 ... 
 // 相应的setter方法定义 
} 
public class Bar 
{ 
 ... 
} 
```

应该使用如下代码所演示的自动绑定定义，才能达到预期的目的：  

```xml
<bean id="fooBean" class="...Foo" autowire="byName"> 
</bean> 
<bean id="emphasisAttribute" class="...Bar"> 
</bean> 
```

需要注意两点：

第一，我们并没有明确指定fooBean的依赖关系，而仅指定了它的autowire属性 为byName；

第二，第二个bean定义的id为emphasisAttribute，与Foo类中的实例变量**名称相同**。

##### 3. byType

如果指定当前bean定义的autowire模式为byType，那么，容器会根据当前bean定义类型，分析其相应的依赖**对象类型**，然后到**容器所管理的所有bean**定义中寻找与依赖**对象类型相同**的bean定义，然后将找到的符合条件的bean自动绑定到当前bean定义。

如果找到，则将找到的bean绑定到Foo的bean定义；

如果没有找到，则不做设置。

如果找到多个，容器会告诉你它解决不了“该选用哪一个”的问题，你只好自己查找原因，并自己修正该问题。

```xml
<bean id="fooBean" class="...Foo" autowire="byType"> 
</bean> 
<bean id="anyName" class="...Bar"> 
</bean>
```

##### 4. constructor 

byName和byType类型的自动绑定模式是针对property的自动绑定，而constructor类型则是**针对构造方法参数的类型**而进行的自动绑定，它同样是**byType类型的绑定模式**。不过，constructor是**匹配构造方法的参数类型**，而不是实例属性的类型。

```java
public class Foo 
{ 
private Bar bar; 
public Foo(Bar arg) 
 { 
this.bar = arg; 
 } 
 ... 
} 
```

```xml
<bean id="foo" class="...Foo" autowire="constructor"/> 
<bean id="bar" class="...Bar"> 
</bean> 
```

##### 5. autodetect

这种模式是byType和constructor模式的结合体

如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，会使用constructor模式。

#### 6. dependency-check

作用不大

#### 7. lazy-init

延迟初始化（lazy-init）这个特性的作用，主要是可以针对ApplicationContext容器的bean初始化行为施以更多控制。

改变某个或者某些bean定 义在ApplicationContext容器中的默认实例化时机。

```xml
<bean id="lazy-init-bean" class="..." lazy-init="true"/> 
<bean id="not-lazy-init-bean" class="..."/> 
```

这样，ApplicationContext容器在启动的时候，只会默认实例化not-lazy-init-bean而不会实 例化lazy-init-bean。 

> 如果某个非延迟初始化的bean定义依赖于lazy-init-bean，那么毫无疑问，按照依赖的顺序，容器还是会首先实例化lazy-init-bean，然后再实例化后者

可以通过`<beans>`设置统一的延迟初始化行为 

```xml
<beans default-lazy-init="true"> 
<bean id="lazy-init-bean" class="..."/> 
<bean id="not-lazy-init-bean" class="..."> 
<property name="propName"> 
  <ref bean="lazy-init-bean"/> 
</property> 
 </bean> 
 ... 
</beans>
```

#### 8. 继承

```java
class SpecificFXNewsProvider extends FXNewsProvider 
{ 
private IFXNewsListener  newsListener; 
private IFXNewsPersister newPersistener; 
 ... 
} 
```

使用同一个IFXNewsPersister依赖对象的FXNewsProvider和SpecificFXNews-  Provider配置内容 ，可以这么写

```xml
<bean id="superNewsProvider" class="..FXNewsProvider"> 
 <property name="newsListener"> 
<ref bean="djNewsListener"/> 
 </property> 
 <property name="newPersistener"> 
<ref bean="djNewsPersister"/> 
 </property> 
</bean> 

<bean id="subNewsProvider" class="..SpecificFXNewsProvider"> 
 <property name="newsListener"> 
<ref bean="specificNewsListener"/> 
 </property> 
 <property name="newPersistener"> 
<ref bean="djNewsPersister"/> 
 </property> 
</bean> 
```

但实际上，这种配置存在冗余，而且也没有表现两者之间的纵向关系。所以，我们可以引入XML中的bean的“继承”配置

```xml
<bean id="superNewsProvider" class="..FXNewsProvider"> 
 <property name="newsListener"> 
<ref bean="djNewsListener"/> 
 </property> 
 <property name="newPersistener"> 
<ref bean="djNewsPersister"/> 
 </property> 
</bean> 

<bean id="subNewsProvider" parent="superNewsProvider"  
 class="..SpecificFXNewsProvider"> 
 <property name="newsListener"> 
	<ref bean="specificNewsListener"/> 
 </property> 
</bean> 
```

这样就继承了superNewsProvider定义的默认值，只需要将特定的属性进行更改，而不要全部又重新定义一遍。

##### 1. abstract

parent属性还可以与abstract属性结合使用，达到将相应bean定义模板化的目的。

```xml
<bean id="newsProviderTemplate" abstract="true"> 
 <property name="newPersistener"> 
<ref bean="djNewsPersister"/> 
 </property> 
</bean>

<bean id="superNewsProvider" parent="newsProviderTemplate"  
 class="..FXNewsProvider"> 
 <property name="newsListener"> 
<ref bean="djNewsListener"/> 
 </property> 
</bean> 

<bean id="subNewsProvider" parent="newsProviderTemplate"  
 class="..SpecificFXNewsProvider"> 
 <property name="newsListener"> 
<ref bean="specificNewsListener"/> 
 </property> 
</bean> 
```

newsProviderTemplate的bean定义通过abstract属性声明为true，说明这个bean定义不需要实例化。

该bean定义只是一个配置模板，不对应任何对象。superNews-  Provider和subNewsProvider通过parent指向这个模板定义，就拥有了该模板定义的所有属性配置。

#### 9. scope

可以翻译成“作用域”

配置中的bean定义可以看作是一个模板，容器会根据这个模板来构造对象。但是要根据这个模板 构造多少对象实例，又该让这些构造完的对象实例存活多久，则由容器根据bean定义的scope语意来决定。

##### 1. singleton 

singleton是容器**默认**的scope

标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。（单例）

该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出， 也就是说，它与IoC容器“几乎”拥有相同的“寿命”。 

![image-20240303221227464](https://salieri-typora.oss-cn-shanghai.aliyuncs.com/img/markdown/image-20240303221227464.png)

以下三个是同样的效果

```xml
<bean id="mockObject1" class="...MockBusinessObject"/> 

<bean id="mockObject1" class="...MockBusinessObject" singleton="true"/> 

<bean id="mockObject1" class="...MockBusinessObject" scope="singleton"/> 
```

##### 2. prototype

容器在接到该类型对象的请求的时候，会每次都重新 生成一个新的对象实例给请求方。

只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。（自生自灭了）

![image-20240303221437170](https://salieri-typora.oss-cn-shanghai.aliyuncs.com/img/markdown/image-20240303221437170.png)

```xml
<bean id="mockObject1" class="...MockBusinessObject" singleton="false"/> 

<bean id="mockObject1" class="...MockBusinessObject" scope="prototype"/>
```

##### 3. request、session和global session

它们只适用于Web应用程序，通常是与XmlWebApplicationContext共同使用

- request

  ```xml
  <bean id="requestProcessor" class="...RequestProcessor" scope="request"/>
  ```

  Spring容器，即XmlWebApplicationContext会为每个HTTP请求创建一个全新的Request- Processor对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。

- session

  //TODO

- global session

  //TODO

## FactoryBean

在强调“面向接口编程”的同时，有一点需要注意：虽然对象可以通过声明接口来避免对特定接 口实现类的过度耦合，但总归需要一种方式将声明依赖接口的对象与接口实现类关联起来。

通过使用工厂方法（Factory Method）模式，提供一个工厂类来实例化具体的接口 实现类，这样，主体对象只需要依赖工厂类，具体使用的实现类有变更的话，只是变更工厂类，而主 体对象不需要做任何变动。

```java
public class Foo 
{ 
private BarInterface barInterface; 
 public Foo() 
 { 
// barInterface = BarInterfaceFactory.getInstance(); 
// 或者 
// barInterface = new BarInterfaceFactory().getInstance(); 
 } 
 ... 
} 
```

### 1. 静态工厂方法（Static Factory Method） 

假设某个第三方库发布了BarInterface，为了向使用该接口的客户端对象屏蔽以后可能对 BarInterface实现类的变动，同时还提供了一个静态的工厂方法实现类StaticBarInterface-  Factory，代码如下：

```java
public class StaticBarInterfaceFactory 
{ 
public static BarInterface getInstance() 
 {
 return new BarInterfaceImpl(); 
 } 
} 
```

为了将该静态工厂方法类返回的实现注入Foo，我们使用以下方式进行配置（通过setter方法注入 方式为Foo注入BarInterface的实例）： 

```xml
<bean id="foo" class="...Foo"> 
 <property name="barInterface"> 
<ref bean="bar"/> 
 </property> 
</bean>

<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/> 
```

带参数的：

```xml
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"> 
 <constructor-arg> 
<ref bean="foobar"/> 
 </constructor-arg> 
</bean> 
```

### 2.  非静态工厂方法（Instance Factory Method）

```java
public class NonStaticBarInterfaceFactory 
{ 
public BarInterface getInstance() 
 { 
return new BarInterfaceImpl(); 
 } 
 ... 
} 
```

因为工厂方法为非静态的，我们只能通过某个NonStaticBarInterfaceFactory实例来调用该方 法（哦，错了，是容器来调用），那么也就有了如下的配置内容：

```xml
<bean id="foo" class="...Foo"> 
 <property name="barInterface"> 
<ref bean="bar"/> 
 </property> 
</bean> 
<bean id="barFactory" class="...NonStaticBarInterfaceFactory"/> 
<bean id="bar" factory-bean="barFactory" factory-method="getInstance"/> 
```

NonStaticBarInterfaceFactory是作为正常的bean注册到容器的，而bar的定义则与静态工厂方 法的定义有些不同。现在使用**factory-bean**属性来指定工厂方法所在的工厂类实例，而**不是通过 class**属性来指定工厂方法所在类的类型。

### 3. FactoryBean 

它本身与其他注 册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂 （Factory）

org.springframework.beans.factory. FactoryBean只定义了三个方法，如以下代码所示：

```java
public interface FactoryBean { 
 Object getObject() throws Exception; 
 Class getObjectType(); 
 boolean isSingleton(); 
} 
```

getObject()方法会返回该FactoryBean“生产”的对象实例

getObjectType()方法仅返回getObject()方法所返回的对象的类型，如果预先 无法确定，则返回null

isSingleton()方法返回结果用于表明，工厂方法（getObject()）所“生 产”的对象是否要以singleton形式存在于容器中。如果以singleton形式存在，则返回true，否则返回false

```java
import org.joda.time.DateTime; 
import org.springframework.beans.factory.FactoryBean; 

public class NextDayDateFactoryBean implements FactoryBean { 
	public Object getObject() throws Exception { 
		return new DateTime().plusDays(1); 
 	} 
	public Class getObjectType() { 
		return DateTime.class; 
 	} 
	public boolean isSingleton() { 
		return false; 
 	} 
} 
```

```xml
<bean id="nextDayDateDisplayer" class="...NextDayDateDisplayer"> 
 <property name="dateOfNextDay"> 
<ref bean="nextDayDate"/> 
 </property> 
</bean> 

<bean id="nextDayDate" class="...NextDayDateFactoryBean"> 
</bean> 
```

NextDayDateDisplayer的定义如下： 

```java
public class NextDayDateDisplayer 
{ 
private DateTime dateOfNextDay; 
 // 相应的setter方法 
 // ... 
} 
```

NextDayDateDisplayer所声明的依赖dateOfNextDay的类型为DateTime，而不是 NextDayDateFactoryBean。也就是说FactoryBean类型的bean定义，通过正常的id引用，容器返回的是**FactoryBean所“生产”的对象类型**，而**非FactoryBean实现本身**。 



如果一定要取得FactoryBean本身的话，可以通过在bean定义的id之前加前缀&来达到目的。





## 方法注入（Method Injection）

```java
public class MockNewsPersister implements IFXNewsPersister { 
	private FXNewsBean newsBean; 
	public void persistNews(FXNewsBean bean) { 
		persistNewes(); 
	} 
	public void persistNews() { 
		System.out.println("persist bean:"+getNewsBean()); 
 	} 
	public FXNewsBean getNewsBean() { 
		return newsBean; 
 	}
    public void setNewsBean(FXNewsBean newsBean) { 
		this.newsBean = newsBean; 
 	} 
}
```

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false"> 
</bean> 
<bean id="mockPersister" class="..impl.MockNewsPersister"> 
 <property name="newsBean"> 
<ref bean="newsBean"/> 
 </property> 
</bean> 
```

虽然FXNewsBean拥有prototype类型的scope，但当容器将一个FXNewsBean的实例注入 MockNewsPersister之后，MockNewsPersister就会一直持有这个FXNewsBean实例的引用。虽然每 次输出都调用了getNewsBean()方法并返回了 FXNewsBean 的实例，但实际上每次返回的都是 MockNewsPersister持有的容器**第一次注入的实例**。



方法注入：

```xml
<bean id="newsBean" class="..domain.FXNewsBean" singleton="false"> 
</bean> 
<bean id="mockPersister" class="..impl.MockNewsPersister"> 
	<lookup-method name="getNewsBean" bean="newsBean"/> 
</bean> 
```

`<lookup-method>`通过的name属性指定需要注入的方法名，bean属性指定需要注入的对象，当 getNewsBean方法被调用的时候，容器可以每次返回一个新的FXNewsBean类型的实例。



## 方法替换（Method Replacement）

与方法注入只是通过相应方法为主体对象注入依赖对象不同，方法替换更多体现在方法的实现层 面上，它可以灵活替换或者说以新的方法实现覆盖掉原来某个方法的实现逻辑。基本上可以认为，方 法替换可以帮助我们实现简单的方法拦截功能。

首先，我们需要给出org.springframework.beans.factory.support.MethodReplacer的实现 类，在这个类中实现将要替换的方法逻辑。

```java
public class FXNewsProviderMethodReplacer implements MethodReplacer { 12  
 	private static final transient Log logger =LogFactory.getLog(FXNewsProviderMethodReplacer.class); 
  
 	public Object reimplement(Object target, Method method, Object[] args)throws Throwable { 
  		logger.info("before executing method["+method.getName()+"] on 			 Object["+target.getClass().getName()+"]."); 
   
  System.out.println("sorry,We will do nothing this time."); 
  logger.info("end of executing method["+method.getName()+"] on Object["+target.getClass().getName()+"]."); 
	return null; 
}
```

有了要替换的逻辑之后，我们就可以把这个逻辑通过`<replaced-method>`配置到FXNewsProvider的bean定义中，使其生效

```xml
<bean id="djNewsProvider" class="..FXNewsProvider"> 
 <constructor-arg index="0"> 
<ref bean="djNewsListener"/> 
 </constructor-arg> 
 <constructor-arg index="1"> 
<ref bean="djNewsPersister"/> 
 </constructor-arg> 
	<replaced-method name="getAndPersistNews" replacer="providerReplacer"> 
 	</replaced-method> 
</bean> 

<bean id="providerReplacer" class="..FXNewsProviderMethodReplacer"> 
</bean> 
<!--其他bean配置--> 
... 
```

我们把FXNewsProvider的getAndPersistNews方法逻辑给完全替换掉了。
