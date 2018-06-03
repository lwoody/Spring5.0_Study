## 1.10 Classpath scanning and managed components



~~이전 장에서 source-level 어노테이션으로 configuration 메타데이터를 제공하는지 알려줬다면, 이번 장에서는 xml-based로 설명함.~~

<u>클래스 경로를 스캔하여 후보 컴포넌트를 검색하는 옵션을 설명함.</u>

-> 후보 컴포넌트(candidate component)란 상응하는 빈에 대한 정의가 등록됨 + 필터기준에 부합한 클래스들을 말함

cf) @Component, @Bean, @Import와 같이 xml 대신 어노테이션 방식으로 빈을 정의할 수 있다.



#### 1.10.1 @Component and further stereotype annotations

<u>spring이 제공하는 stereotype annotations</u>

@Component : Spring-managed component를 위한 일반적인 어노테이션

그 하위로 특화된 어노테이션이 @Repository, @Service, @Controller이다.



- @Repository - for persistence layers(DAO와 같은 repository역할을 하는 클래스들에 붙이는 어노테이션)
- @Service - for service layers
- @Controller- for presentation layers



@Component를 하위 항목 대신에 붙여도 되지만, 각 어노테이션에 붙는 기능도 있고 포인트컷에 이상적인 타겟을 설정 or spring 향후 릴리즈에 추가적인 의미가 더해질 수 있기 때문에 더 명시적으로 표현할 수 있음

ex) @Repository - automatic exception translation 기능이 더해진다. -> lowlevel의 persistence exception을 high level로 변환해주는것(spring exception)



#### 1.10.2. Meta-annotations

spring에서 제공하는 많은 어노테이션들은 meta-annotation으로도 사용가능.

meta-annotation이란 -> 다른 어노테이션에 적용될 수 있는 어노테이션

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {
    // ....
}
```

위처럼 @Service 어노테이션을 @Component와 같은 방식으로 다뤄질 수 있게 @Component를 적용한것



또 Composed annotation을 만들때 사용할 수 있음

ex) @ResponseBody + @Controller = @RestController : 해당 컨트롤러 내에서는 객체 리턴 시 @ResponseBody 생략 가능



또 사용자의 customizing을 위해 선택적으로 특성을 재선언할 수 있게함

ex) @Scope 어노테이션에서 porxyMode를 TARGET_CLASS로 디폴트 지정한게 @SessionScope임. -> @Scope라는 어노테이션의 attributes 중 일부만 노출시키는 것.

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

`@SessionScope` can then be used without declaring the `proxyMode` as follows:

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

Or with an overridden value for the `proxyMode` as follows:

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

> If you want to store the whole bean in the session, use @Scope, otherwise use @SessionAttributes. In the case of using @Scope, <u>if the class implements some interfaces then use INTERFACES proxy mode</u>, if not use TARGET_CLASS.
>
> Proxying with interfaces means Spring will use JDK proxies which can only take the target bean's interface types. Proxying with target class means Spring will use CGLIB proxies which can take the target bean's class and interface types.
>
> Using INTERFACES should be used if possible and TARGET as last resort if the bean does not implement interfaces.
>
> cglib, jdk proxy :
>
> http://wonwoo.ml/index.php/post/1576,
>
>  http://wiki.javajigi.net/pages/viewpage.action?pageId=1065
>
> http://javacan.tistory.com/entry/114
>
> https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/
>
> https://m.blog.naver.com/PostView.nhn?blogId=tmondev&logNo=220558804255&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F



#### 1.10.3. Automatically detecting classes and registering bean definitions

spring은 자동으로 stereotyped class들을 검색해서 그에 상응하게 빈으로 등록시킬 수 있음.

-> @Configuration이 붙여진 class에 @ComponentScan을 더함으로써 가능

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

xml로 대체한 것은

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> The use of `<context:component-scan>` implicitly enables the functionality of `<context:annotation-config>`. There is usually no need to include the `<context:annotation-config>` element when using `<context:component-scan>`.
>
> `AutowiredAnnotationBeanPostProcessor` and`CommonAnnotationBeanPostProcessor` are both included implicitly when you use the component-scan element. That means that the two components are autodetected and wired together - all without any bean configuration metadata provided in XML.
>
> 위 두개를 사용하고 싶지 않으면  `<context:annotation-config>` 값을 false로 지정하면 됨.



#### 1.10.4. Using filters to customize scanning

@Component, @Repository, @Service, @Controller, @Component 적용하여 커스터마이징된 component 들만 detected 후보 컴포넌트이다.

그러나 @ComponentScan(xml은 component-scan element)에 includeFilter, excludeFilter 파라미터를 사용해 custom fliter를 적용할 수 있다. 각 filer에는 type과 expression이 요구된다.

<u>Filtering Option</u>

| Filter Type          | Example Expression           | Description                                                  |
| -------------------- | ---------------------------- | ------------------------------------------------------------ |
| annotation (default) | `org.example.SomeAnnotation` | An annotation to be present at the type level in target components. |
| assignable           | `org.example.SomeClass`      | A class (or interface) that the target components are assignable to (extend/implement). |
| aspectj              | `org.example..*Service+`     | An AspectJ type expression to be matched by the target components. |
| regex                | `org\.example\.Default.*`    | A regex expression to be matched by the target components class names. |
| custom               | `org.example.MyTypeFilter`   | A custom implementation of the `org.springframework.core.type .TypeFilter` interface. |

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

위처럼 포함시키고 제외할 항목을 custom하게 지정할 수 있다.

아래는 xml 버전

``` xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```



useDefaultFilters=false를 사용하면 `@Component`, `@Repository`, `@Service`, `@Controller`, or `@Configuration` 어노테이션 적용된 클래스들에 대해 auto detection 싹 무시함.



#### 1.10.5. Defining bean metadata within components

@Component, @Configuration 어노테이션 붙인 class 내에서 Bean 메타 데이터를 정의할 수 있다.

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

publicInstance, protectedInstance, privateInstance, requestScopedInstance 각각의 bean들이 정의되있는 것.

protectedInstance 메소드를 보면 country라는 string parameter를 받아 privateInstance 빈의 age라는 property에 wiring 시키는 것이다. 여기서 Spring Expression Language가 @Value 어노테이션을 사용해`#{ <expression> }` 와 같은 방식으로 쓰였다.

Spring 4.3부터는 factory method 파리미터의 타입으로 InjectionPoint를 선언할 수 있다. -> bean의 생성을 트리거하는 requesting injection point에 접근하기 위해 쓰인다. 기존 빈의 주입이 아닌 빈 instance 생성에만 적용된다. 따라서 prototype scope 빈에 의미가 있다.

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```



@Component과 @Configuration에서의 @Bean 차이점 : CGLIB 처리 여부

- @Component 클래스들은 CGLIB proxy에 의해 메소드나 필드의 호출을 intercept하는 과정이 없다. 

  > invoking a method or field in an `@Bean` method within a plain `@Component` class *has*standard Java semantics, with no special CGLIB processing or other constraints applying

- @Configuration 클래스들 내부에 구현된 @Bean 메소드는 다른 빈들을 참조할 때 컨테이너에서의 생명주기 관리와 Spring bean의 프록시를 제공받게 된다.

  > CGLIB proxying is the means by which invoking methods or fields within `@Bean` methods in `@Configuration` classes creates bean metadata references to collaborating objects



#### 1.10.6. Naming autodetected components

컴포넌트가 autodetecting 될때 BeanNameGenerator 전략에 의해 scanner가 bean name을 생성하게 된다.

- name value를 지정하면 그 값이 이름
- 없다면 uncaplitalized, nonqualified 클래스 name

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

-> myMovieLister

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

-> movieFinderImpl



#### 1.10.7. Providing a scope for autodetected components

default값은 singleton이고 @Scope에 스코프를 지정할 수 있다.

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```



non-singleton 스코프 빈을 사용하게 될 때 scoped object를 위한 proxy가 필요한 경우가 있다.

-> scopedPoxy attribute가 component scan에서 사용가능하다.

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

no, interfaces, targetClass 세가지가 가능하고 위의 configuration의 경우 CGLIB가 아닌 JDK dynamic proxy가 된다.



#### 1.10.8. Providing qualifier metadata with annotations

@Quailifer 어노테이션, custom qualifier을 통해서 autowiring을 좀 더 세밀하게 컨트롤할 수 있다.



custom qualifier

```java
@Target(value = { ElementType.TYPE, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

}
```



```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```



#### 1.10.9. Generating an index of candidate components

Spring 5에서는 classpath scanning이 굉장히 빠르긴 하지만 app의 start performance 개선하기 위해서 컴파일 타임에 후보 컴포넌트들의 static list를 생성하므로써 가능하다. 이때 모든 모듈들은 이 매커니즘을 따르게 되고, ApplicationContext가 index를 감지하면 scanning하지 않고 해당 것을 사용하게 된다.

index를 생성하기 위해서는 스캔의 대상이 되는 컴포넌트들을 포함하는 각 모듈들에 dependency를 추가하면 된다.

아래는 메이븐 설정

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.0.6.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

-> META-INF/spring.components 파일이 생성되고 jar에 포함되게 됨.