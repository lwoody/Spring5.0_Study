Spring 5.0 스터디

[TOC]



# 1. IoC 컨테이너

Spring 프레임워크에서 가장 중요한 기술이 IoC(inversion of control) 컨테이너고 AOP(Aspect Oriented Programming) 기술이 밀접하게 연결되어있다. -> 보통 Spring과 AspectJ를 결합해 사용



## 1.1 IoC 컨테이너, Bean 소개

<u>spring IoC컨테이너는 DI(depnedency Injection) 의존성 주입</u>

Ioc라는게 컨테이너에 객체 생성, 관리를 일임(즉 제어권의 역전)하고 DL, DI 방식으로 사용할 수 있게 끔 한것으로 암. 

*cf ) DL은 각 컨테이너에서 객체들을 관리하는 별도의 저장소를 가지는데 여기에있는 Bean을 접근하기 위해서 컨테이너에서 제공하는 API를 이용해 bean을 lookup하는 것을 말한다. 따라서 컨테이너 api(app)과 객체들간에 의존관계를 맺게 되는것.*

의존성 = 객체간의 결합관계

 의존성 정의 방법 : 

1. 생성자의 인수 통하여
2. 팩토리 메소드의 인수 통하여
3. 팩토리 메소드에서 생성되거나 리턴된 객체 인스턴스에 설정된 속성 통해서 - setter injection

IoC 컨테이너는 Bean을 생성할때 정의된 의존성을 주입하게 된다.

-> **객체 생성, 의존관계, 생명주기 관리를**를 직접 코드로 처리하지 않고 **xml 설정파일, Annotaion으로 등록된 정보를 바탕으로 해 컨테이너가 자동 처리**할 수 있게 한 것



<u>IoC 컨테이너의 Basis</u>

- BeanFactory : 객체 생성 및 관리의 기본 기능 -> lazy loading(클라이언트가 lookup할 때만 객체 생성)
- **ApplicationContext** - sub interface of BeanFactory : 위 이외의 AOP, 다국어 처리, 트랜잭션 관리 등의 엔터프라이즈 app에 필요한 기능 추가됨 -> pre-loading(컨테이너 구동 시에 객체 생성)

ApplicationContext가 BeanFactory의 상위집합이고 여기서는 그것만 다룸. ApplicationContext가 곧 IoC 컨테이너(여기서 말하자면)



<u>Bean</u>

App의 뼈대를 형성하고 IoC 컨테이너에 의해 관리되는 객체를 Bean이라고 함 -> 컨테이너에 의해 인스턴스화, 조립, 관리되는 객체이다.

Bean들 간의 의존성은 컨테이너가 사용하는 **configuration meatadata(<u>xml, annotation, javaconfig-자바코드</u>)**에 반영되어있다.



## 1.2 컨테이너 개요

컨테이너(ApplicationContext)는 configuration metadata를 기반으로 bean의 인스턴스화, 구성, 조립하는 일을 한다.

**ApplicationContext의 구현체**는 다양하고 이를 인스턴스화하는 코드는 **개발환경 구성 시에 쉽게 작성**할 수 있다. ex) IDE에서 자동으로 작성해줌



#### 1.2.1 Configuration metadata

요즘에는 대부분 xml보다 java-based configuration을 택함.

-> (`@Configuration`, `@Bean`, `@Import` , `@DependsOn ` 같은 어노테이션 사용)



<u>XML 기반 예시</u>

*applicationContext-filter.xml*

```xml
...
<!-- 캐시 설정(Arcus) -->
    <bean id="arcusClientFacadeCore"  class="com.nhncorp.nwe.corelib.arcus.client.ArcusClientFacadeCore">
        <property name="arcusUrl" value="ncloud.arcuscloud.nhncorp.com:17288"/>
        <property name="arcusServiceCode" value="3c89b90e321f4eb096a22d80c92e4f51"/>
    </bean>
```

속성

- id : 개별 bean 식별값 -> 해당 객체가 참조될때 쓰이는 값

- class : bean의 유형 정의

  ​



#### 1.2.2 컨테이너 인스턴스화

ApplicationContext의 생성자에 configuration metadata의 위치 경로(들)을 넣어 생성한다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```



nBaseDataSource를 참조하는 예시

*applicationContext-test.xml*

```xml
<!-- Transaction Rollback을 위한 설정 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="nBaseDataSource" />
    </bean>
```

*datasources-nbase.xml*

```xml
<bean id="nBaseDataSource" class="com.naver.calendar.core.dao.NBaseDataSourceFactoryBean" primary="true">
		<property name="host" value="#{database['nbase.host']}" />
		<property name="port" value="#{database['nbase.port']}" />
	</bean>
```



<u>XML 기반 설정하기</u>

다수의 xml파일로 나누어 설정하고 싶을 때 import(bean 네임스페이스에서 제공)를 사용한다. - 상대 경로로 넣어줌

각 개별 xml 파일은 아키텍쳐의 논리 계층 또는 모듈을 나타낸다.

*applicationContext.xml*

```xml
<import resource="lucyConfiguration.xml" />
```



**"…/"경로 사용해 상위 디렉토리의 파일 참조는 권장하지 않음** -> 외부 파일에 대한 종속성이 만들어지게 됨.

"classpath:/config/services.xml"과 같은 정규화된 경로 설정도 가능함.

"${...}"를 통해 런타임시 JVM 시스템 등록 정보에 의해 세팅하는 방법이 좋다고 함.



"context", "util" 네임스페이스로 추가적인 구성 기능 사용할 수 있음.

ex)

```xml
<context:component-scan base-package="com.naver.calendar">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
```

```xml
<beans profile="local">
            <util:properties id="core" location="classpath:profiles/naver/local/core.properties"/>
            <util:properties id="database" location="classpath:spring/naver/local/database.properties"/>
            <util:properties id="common" location="classpath:spring/naver/local/common.properties"/>
            <util:properties id="remote" location="classpath:spring/naver/local/remote.properties"/>
            <util:properties id="cache" location="classpath:spring/naver/local/cache.properties"/>
        </beans>
```



#### 1.2.3 컨테이너 사용

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

PetStoreService service = context.getBean("petStore", PetStoreService.class);

List<String> userList = service.getUsernameList();
```

getBean 통해 bean의 인스턴스 가져올 수 있다.

-> **사용하지 말고 Autowired 어노테이션 사용할 것**



## 1.3 Bean 개요

Bean definition 통한 configuration metadata 설정되는 항목들

- package-qualified class name : bean의 실제 구현 class
- 컨테이너에서의 동작 방식 : scope, lifecycle 등
- 다른 Bean에 대한 참조(의존성)
- 해당 Bean의 setting : connecton pool관리하는 bean에서의 connection pool size 제한 등

| 속성                     | 설명                                                         |
| ------------------------ | ------------------------------------------------------------ |
| class                    | bean의 실제 구현 class 경로(패키지 경로 포함) - 필수 [Instantiating beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| name                     | bean 식별값(id와 다르게 자유로운 형식가능) [Naming beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-beanname) |
| scope                    | bean이 사용될 scope 지정(default값은 singleton - 전체 범위에서 1개) [Bean scopes](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes) |
| constructor arguments    | 생성자 주입 시 사용 [Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| properties               | setter 주입 시 사용 [Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| autowiring mode          | 적절한 코드로 자동 주입 시 사용 [Autowiring collaborators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-autowire) -> 그냥 annotation쓰면됨; |
| lazy-initialization mode | 클라이언트 lookup 시점에 객체 생성하는 지연로딩 설정(default값은 false) [Lazy-initialized beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init) |
| initialization method    | 멤버변수 초기화 메소드 지정 [Initialization callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| destruction method       | 컨테이너가 객체 삭제 직전 호출할 메소드 지정 [Destruction callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

이외에 ApplicationContext의 getBeanFactory() 통해서 컨테이너 외부에서 생성한 객체를 bean으로 등록할수도 있지만 권장하지 않음.

> **Bean metadata and manually supplied singleton instances need to be registered as early as possible**, in order for the container to properly reason about them during autowiring and other introspection steps. While overriding of existing metadata and existing singleton instances is supported to some degree, **the registration of new beans at runtime (concurrently with live access to factory) is not officially supported and may lead to concurrent access exceptions and/or inconsistent state in the bean container.**



#### 1.3.1 Bean 네이밍

<u>xml 기반</u> configuration metadata에서는 id나 name 속성 사용해 Bean 식별자 지정한다.

굳이 Bean 식별자를 쓰지 않아도 innerBean으로 작성하거나 autowiring collaborator 방식을 쓰면 된다.

autowire collaborator - byName 예시 -> Customer의 필드명중 person과 이름이 매칭되어 autowire됨

```java
public class Customer 
{
	private Person person;
	
	public Customer(Person person) {
		this.person = person;
	}
	
	public void setPerson(Person person) {
		this.person = person;
	}
	//...
}
```

```xml
<bean id="customer" class="com.mkyong.common.Customer" autowire="byName" />
<bean id="person" class="com.mkyong.common.Person" />
```



alias 설정으로 별칭 부여할 수 있다. -> ex) 실제 bean 네임에 대해 종속적이지 않아 profile 설정 변경 시 참조해야하는 bean값 다르게 할 수 있음 (A->B : B환경에서 / A->C : C환경에서)

<u>어노테이션으로도</u> @Configuration설정된 class 내에서 메소드에 @Bean 설정하여 bean 생성 가능.



#### 1.3.2 Bean 인스턴스화

Bean 인스턴스는 위에서 설명한 bean definition 기반으로 생성된다.



<u>*cf) static nested class의 bean 등록*</u>

Foo 클래스의 inner class로 Bar라는 static 클래스가 있다면,

'com.example.Foo$Bar'를 class 속성으로 할당하여 정의한다.

> `$` character in the name to separate the nested class name from the outer class name.



##### <u>기본적인 방법</u>

1. 생성자 인젝션

   생성자의 매개변수로 의존관계에 있는 객체 정보를 전달

   - xml based

     ```xml
     <bean id="mockedContactApi" class="org.mockito.Mockito" factory-method="mock" primary="true">
                 <constructor-arg value="com.naver.calendar.core.remote.contact.CalendarDomainContactAPIFetcher"/>
             </bean>
     ```

     ​

2. setter 인젝션

   setter 함수를 사용하여 의존관계에 있는 객체 정보를 전달

   - xml based

     ```xml
     <bean id="calendarComponentMapperRaw" class="org.mybatis.spring.mapper.MapperFactoryBean" primary="true">
             <property name="mapperInterface" value="com.naver.calendar.core.dao.mappers.raw.CalendarComponentMapperRaw"/>
             <property name="sqlSessionFactory" ref="sqlSessionFactoryBean-nBase"/>
         </bean>
     ```

     MapperFactoryBean에 setter 메소드를 사용해서 주입



3. annotation 기반

   @Autowired, @Qualifier 등 사용

   ​

##### <u>Static factory method 사용한 방법</u>

팩토리 메소드 패턴 이용하여 객체를 생성하는 방식이다.

1. bean을 생성할 static 팩토리 메소드를 포함하는 클래스를 class 속성에 넣어준다.
2. factory-method 속성에 팩토리 메소드 이름을 할당한다.
3. constructor-arg 속성에는 해당 팩토리 메소드에 들어갈 인자를 넣어준다.

*testContext-mocks.xml*

```xml
<bean id="mockedEnterpriseBO" class="org.mockito.Mockito" factory-method="mock" primary="true">
            <constructor-arg value="com.naver.calendar.core.enterpriseconfiguration.EnterpriseConfigurationBO"/>
</bean>
```

```java
public class Mockito extends ArgumentMatchers {
    ...
    public Mockito() {
    }

    public static <T> T mock(Class<T> classToMock) {
        return mock(classToMock, withSettings().defaultAnswer(RETURNS_DEFAULTS));
    }
    ...
```

<u>따라서 **Bean Definition에 생성되는 객체의 타입이 정의되어 있지 않는 것**이다.</u>



##### <u>instance factory method 사용한 방법</u>

위와 유사하지만 기존에 존재하는 Bean의 non-static 메소드를 호출해 새로운 Bean을 생성한다.

1. class 속성은 비운다.
2. factory-bean 속성에 bean을 생성할 팩토리 메소드를 가진 기존의 bean의 이름을 지정한다.
3. factory-method 속성에 해당 팩토리 메소드 이름을 할당한다.
4. constructor-arg 속성은 위와 같다.

우리 프로젝트에는 없는듯?

아래 예시같이 여러개의 팩토리 메소드를 포함할 수도 있다.

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {
	private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```



-> 3가지 방법 차이점 궁금? (어떤 상황에서 각각 쓰일지)



## 1.4 의존성

대부분의 app들은 하나의 많은 object들이 결합하여 동작하기 때문에 의존성을 가질 수 밖에 없다.

이 의존성 관리에 대한 내용

#### 1.4.1 DI(dependency injection) - 의존성 주입

DI는 위에 설명한 생성자 인젝션 + setter 인젝션 + 팩토리 메소드 통한 방법으로만 객체간 의존성을 정의하는 절차를 말한다.

> *Dependency injection* (DI) is a process whereby objects define their dependencies, that is, the other objects they work with, only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.



##### constructor-based injection

```java
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```

생성자의 인수들의 각 타입이 상속관계같은 '잠재적 모호성'(문서에서의 표현)이 없으면 아래와 같은 xml 정의를 해주면 제대로 설정된다.

```java
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```

또 아래와 같이 **타입 매칭, index, name**(debug flag enabled 되야함 - spring이 파라미터 이름을 lookup할 수 있게 -> 아니면 생성자 위에 <u>@ConstructorProperties({"years", "ultimateAnswer"}) 붙여줘야함</u>)로 각 인수를 구별할 수 있다.

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```



##### setter-based injection

컨테이너가 bean을 생성(no-argument constructor나 no-argument static factory method로)후 setter 메소드로 주입하는 방식이다.



<u>컨테이너의 dependency resolution process</u>

1. ApplicationContext 생성 및 초기화(configuration metadata기반 - xml, java code, annotation)
2. bean이 생성될때 정의된 방식에 따라 의존성 주입

이때 Bean은 정의된 scope에 따라서 생성되는 시점이 다르다

- singleton(default)->컨테이너 생성시점
- 다른 scope은 해당 요청이 있을 때 - request->http request, session->http session 등



Circular dependencies가 일어날 수 있으니 조심해야한다.

ex) class A가 classB를 constructor 주입해야하고  classB도 classA를 constructor 주입해야하는 경우 `BeanCurrentlyInCreationException` 발생함.

-> setter 인젝션으로는 가능함

만약 circular 의존성 설정이 아니라면 순서에 맞게 생성하고 주입 과정을 진행한다.

> if bean A has a dependency on bean B, the Spring IoC container completely configures bean B prior to invoking the setter method on bean A



#### 1.4.2 의존성과 configuration 상세내용

`<property/>` ,`<constructor-arg/>` element나 p-namespace 통해 값을 넣을 수 있고, `ref`로 참조되는 다른 bean을 설정할 수 있다.

`<property/>` 사용하면 setter 인젝션, `<constructor-arg/>` 는 생성자 인젝션 임. 

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>
```

```xml
<ref bean="someBean"/>
```



<u>inner bean</u>

다음과 같이 쓸 수 있다. 대신 안에 있는 bean은 설정된 scope 무시 + 내부안(outer bean)에서만 사용될 수 있고 외부에서는 참조할 수 없게 된다.

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```



<u>collections</u>

`<list/>`, `<set/>`, `<map/>`, and `<props/>` 로 각각 list, set, map, properties로 세팅할 수 있다. (java collection 타입)

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

(Properties 클래스는 자바 어플리케이션에서 사용될 상수를 텍스트 파일로 미리 작성한 뒤 필요한 곳에서 그 파일을 읽어오는데 사용) -> .properties파일 참고



<u>collection merging</u>

다음과 같이 parent, child 관계의 bean을 설정하면 child에서 설정한 값으로 parent를 덮어쓰기(override)한다. - list, map, set, propeties 모두 가능 / 하지만 다른 타입끼리는 불가능

props에 merge="true"설정 해주어야한다.

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

`adminEmails` properties collection 결과값은 아래와 같아진다.

```properties
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk // child에 의해 덮어씌워짐
```



<u>null값 할당</u>

null value 는 아래와 같이 할당한다.

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```



<u>c-namespace</u>

spring 3.1붙어 도입되었고 `constructor-arg`tag를 길게 줄줄이 쓰는 대신 bean안에 attribute으로  다음과 같이 작성할 수 있게 했다. (c:a-ref="" 형식) 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c" //다음과 같은 정의가 필요함
       ...

<!-- traditional declaration -->
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
        <constructor-arg value="foo@bar.com"/>
    </bean>

    <!-- c-namespace declaration -->
    <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

..
```



<u>compound propery names</u>

속성안에 속성안에 속성같이 합성된 속성에 할당하고 싶으면 다음과 같이 작성 가능하다. 하지만 여기서 fred, bob이 null이면 안된다. -> NPE발생

```xml
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```



#### 1.4.3 depends-on 사용

객체간의 의존성은 불명확하나 객체간 <u>생성 순서</u>를 결정해주고 싶을 때 사용한다.

아래는 manager bean이 생성되기전에 beanOne이 생성되게 한다.

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```
뿐만 아니라 manager bean이 소멸되기 전에 beanOne이 먼저 소멸되게 끔 <u>소멸 순서</u>도 결정해준다.



#### 1.4.4 lazy-initialized beans(지연 로딩)

ApplicationContext(컨테이너) 생성시에 bean이 생성되는게(singleton scope) 일반적으로 error 상황을 일찍 잡아낼수 있어 바람직하지만, lazy-init="true"로 설정하면 시작과 동시에 생성되지 않고 해당 bean이 요청될때 생성한다.

```xml
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

container level에서도 설정 가능하다.

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```



#### 1.4.5 Autowiring collaborators

객체간 의존성을 위와같이 ref로 직접 나타내지 않고 각각의 bean을 만들고 autowire 속성으로 판단하여 주입하는 것이다.

아래를 예로 들면

```java
package com.mkyong.common;

public class Customer 
{
	private Person person;
	
	public Customer(Person person) {
		this.person = person;
	}
	
	public void setPerson(Person person) {
		this.person = person;
	}
	//...
}

...
package com.mkyong.common;

public class Person 
{
	//...
}
```

```xml
<bean id="customer" class="com.mkyong.common.Customer" autowire="byType" />
	
	<bean id="person" class="com.mkyong.common.Person" />
```

Customer클래스에서 setter method들의 type을 가지는 bean을 찾아 알아서 주입한다.(여기서 person bean을 가져오게 된다.)



-> 사실 이런식으로 할거면 @Autowired 쓰는게 편할듯?

spring 메뉴얼에서도 그리 권장은 안함. -> 명시적이지가 않아서 개발 시 혼동줌

> Autowiring works best when it is used consistently across a project. If autowiring is not used in general, it might be confusing to developers to use it to wire only one or two bean definitions.



#### 1.4.6 method injection

만약 singleton Bean에 prototype-scoped bean(context에 접근해서 bean가져올때마다 새로 생성하는 방식)을 주입했을 때 문제가 생긴다.

singleton bean은 컨테이너 생성 시에 한번만 생성되고 캐시되어 사용되기 때문에 서로 맞지가 않는 것이다.

이렇게 bean 간의 scope가 달라서(lifecycle이 달라서) 의존성 설정에 문제가 생기는 경우 spring에서는 method injection을 사용한다. -> 여기서는 <u>Lookup method</u> 방식 사용

<u>prototype-scoped bean의 생성되어 주입되는 방식을 abstract method를 통하게 끔 한다.</u>

예를 들어, 다음과 같은 오류를 가진 코드를

```java
public class Singleton {

    private Prototype prototype;

    public Singleton(Prototype prototype) {//singleton에 prototype을 주입 중
        this.prototype = prototype;
    }

    public void doSomething() {
        prototype.foo();
    }

    public void doSomethingElse() {
        prototype.bar();
    }
}
```

다음과 같이 <u>abstract로 바꾸어 상속하게 하고, abstract method를 작성</u>한다.

```java
public abstract class Singleton {

    protected abstract Prototype createPrototype();

    public void doSomething() {
        createPrototype().foo();
    }

    public void doSomethingElse() {
        createPrototype().bar();
    }
}
```

그리고 <u>createPrototpye 추상 메소드는 xml에 metadata를 통해 설정하면 spring에서 알아서 구현</u>해준다.(lookup-method element에 설정)

```xml
<bean id="prototype" class="ch.frankel.blog.Prototype" scope="prototype" />
<bean id="singleton" class="sample.MySingleton">//상속한 singleton 클래스
	<lookup-method name="createPrototype" bean="prototype" />
</bean>
```

spring 4.1 이후에는 annotation으로 아래와 같이 @Component하위에 @Lookup으로 사용가능

```java
@Component
public class MyClass1 {
  doSomething() {
    myClass2();
  }

  //I want this method to return MyClass2 prototype
  @Lookup
  public MyClass2 myClass2(){
    return null; // This implementation will be overridden by dynamically generated subclass
  }
}
```



-> spring의 힘 신기방기