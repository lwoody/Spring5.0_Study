###### <u>Spring 5.0</u>

## Chapter 5. Aspect Oriented Programming with Spring



### 5.1 소개

- ''관점 지향''으로 '객체 지향' 프로그래밍을 보완한다. 
  -> OOP를 활용한 DI가 의존성(new)에 대한 주입이라면, AOP는 로직(code)의 주입이라고 볼 수 있다.

- 로깅, 보안, 트랜잭션 같은 기능들이 반복적으로 사용됨
  -> 다수의 모듈에 공통적으로 나타나는 부분들 = 횡단 관심사(corss-cutting concern)

- Spring IoC container가 AOP에 의존하지는 않지만(사용안해도 됨), 유용한 미들웨어 솔루션을 제공한다.

- xml schema 기반 / AspertJ 어노테이션 스타일 두가지가 Spring 2.0 이후로  Spring AOP를 보완해 사용되고 있다.

  > Spring AOP will never strive to compete with AspectJ to provide a comprehensive AOP solution. We believe that both proxy-based frameworks like Spring AOP and full-blown frameworks such as AspectJ are valuable, and that they are complementary, rather than in competition. 



##### <u>AOP 컨셉</u>

- Aspect : 다양한 클래스들에 횡단으로 잘라서 들어갈 수 있는 관심사(concern)들이 모듈화 된 것
  -> 기본적인 표준 클래스(xml-based)나 표준 클래스에 @Aspect 어노테이션 사용해서 구현
  ​

- JoinPoint : Aspect 적용이 가능한 어떤 지점
  -> Spring AOP는 Aspect를 메서드에만 적용 가능(In Spring AOP, a *JoinPoint* always represents a method execution)
  -> AspectJ는 메소드 뿐 아니라 속성 등에도 적용가능 함
  ​

- PointCut : 특정 JoinPoint에 Aspect를 적용하게 하는 위치 지정자
  ​

- Advice : PointCut에 적용되는 액션(로직 or 메서드)
  -> '언제'의 개념까지 정의한 메서드(언제 + 무엇을 포인트컷에 적용할지)

  1. Before - 메소드 시작 전
  2. After returning - 메소드 정상 종료 후
  3. After throwing - 메소드에서 예외 발생하고 종료 후
  4. After (finally) - 메소드 종료 후
  5. Around - 메소드의 전 구역

  around advice가 가장 제너럴하지만, 특정 요구되는 행위를 구현하기 위해서는 least powerful advice type을 사용할 것을 권장(구체적인 타입을 사용)
  -> 메소드의 리턴 값을 캐싱할때는 around 보다 after를 사용하는 것이 잠재적 에러를 줄일 수 있다.

  ​

- Target Object( = Advised Object) : 하나 이상의 aspect에 advised된 객체를 말한다.
  ->Spring AOP는 런타임 프록시를 사용해 구현되기 때문에, target 객체는 항상 프록시된 객체이다.

  ​

- Weaving : Advised 객체를 만들기 위해 컴파일 타임, 로드 타임, 런타임에 aspect들을 linking하는 것. = pointCut에 의해 결정된 joinPoint에 지정된 advice를 삽입하는 과정(crosscutting) -> Spring AOP는 런타임에 weaving한다.
  ​


- Introduction : 타겟 클래스에 추가적인 메서드나 필드를 선언하는 것 - 동적으로 인터페이스를 구현
  -> Spring AOP는 Advised된 객체들에 새로운 인터페이스를 introduce 가능하다.(인터페이스 제공하므로써 구현하게끔)
  An introduction is known as an inter-type declaration in the AspectJ community.
  특별한 Advice 중 하나로 생각하면된다. Target 클래스에 완전히 새로운 인터페이스를 추가적으로 구현할 수 있도록 지원한다. - A라는 클래스의 소스를 수정하지 않고 B라는 인터페이스 기능을 추가하는 것

  예 : http://wiki.javajigi.net/pages/viewpage.action?pageId=1084

  ​

- AOP proxy : aspect를 적용하기 위해 AOP 프레임워크에 의해 생성된 객체
  -> JDK dynamic proxy - 인터페이스 기반 / CGLIB proxy - 클래스 기반 둘 중 하나

  (It is important to grasp the fact that Spring AOP is *proxy-based*)

  > **JDK Dynamic proxy** can only proxy by interface (so your target class needs to implement an interface, which is then also implemented by the proxy class).
  >
  > **CGLIB (and javassist)** can create a proxy by subclassing. In this scenario the proxy becomes a subclass of the target class. No need for interfaces.

   JDK *dynamic proxies* 가 디폴트이다. (This enables any interface (or set of interfaces) to be proxied.)

  business object가 interface 상속 받지 않으면 CGLIB 프록시를 사용한다.

  > As it is good practice to program to interfaces rather than classes; business classes normally will implement one or more business interfaces.



### 5.2 @AspectJ 지원

**@AspectJ refers to a style of declaring aspects** as regular Java classes annotated with annotations.

어노테이션을 사용해 기본 자바 클래스로 aspect를 정의하는 방식을 말함.(AspectJ 라이브러리 사용해야함)



- #### AspectJ 사용 활성화

  사용하려면 Spring이 @AspectJ 스타일 aspect와 aspect들이 적용된 Bean에 대한 Auto proxy 기능을 가능하게 설정해야한다.

  By autoproxying we mean that if Spring determines that a b**ean is advised by one or more aspects, it will automatically generate a proxy for that bean to intercept method invocations and ensure that advice is executed as needed**.

  ​

  @AspectJ support can be enabled with **XML or Java style configuration.**

  java style

  ```java
  @Configuration
  @EnableAspectJAutoProxy
  public class AppConfig {

  }
  ```

  xml style

  ```xml
  <aop:aspectj-autoproxy/>
  ```

  ​


- #### Aspect 정의

  Any bean defined in your application context with a class that is an @AspectJ aspect (has the `@Aspect` annotation) will be automatically detected by Spring and used to configure Spring AOP. 

  Spring이 정의된 빈들 중 @Aspect 어노테이션 붙은 것들을 찾아서 자동으로 설정함

  빈 정의하고

  ```xml
  <bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
      <!-- configure properties of aspect here as normal -->
  </bean>
  ```

  해당 빈 클래스에 어노테이션 붙여놓기

  ```java
  package org.xyz;
  import org.aspectj.lang.annotation.Aspect;

  @Aspect
  public class NotVeryUsefulAspect {

  }
  ```

  이제 저 안에 pointcut, advice, introduction 같은 것들이 들어가게 된다.

  ​

  컴포넌트 스캐닝으로 설정 가능한데,

  > However, note that the ***@Aspect* annotation is *not* sufficient for autodetection** in the classpath: For that purpose, **you need to add a separate *@Component* annotation**



- #### PointCut 정의

  advice가 실행되는 시점(지점)을 정하는 것이다.

  > *Spring AOP only supports method execution join points for Spring beans*, so you can think of a pointcut as matching the execution of methods on Spring beans

  ​

  두 부분으로 나누어 정의한다.

  1. **a pointcut signature** comprising a name and any parameters -> regular method definition

  2. **a pointcut expression** that determines *exactly* which method executions we are interested in -> `@Pointcut`annotation + regular AspectJ 5 pointcut expression

     ([AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html) 에 표현식 설명 있음)

  ```java
  @Pointcut("execution(* transfer(..))")// the pointcut expression
  private void anyOldTransfer() {}// the pointcut signature
  ```

  ​

  <u>포인트컷 지정자들</u>(PCD - pointCut designators) - <u>keyword telling Spring AOP what to match</u>

  - ***execution*** - for matching method execution join points, this is the primary pointcut designator you will use when working with Spring AOP
    ​

  - ***within*** - limits matching to join points **within certain types** (simply the execution of a method declared within a matching type when using Spring AOP)

    ```java
    @Pointcut("within(org.baeldung.dao.FooDao)")
    ```

    ​

  - ***this***, ***target*** - *<u>this</u>* limits matching to join points where the bean reference (Spring AOP proxy) is an instance of the given type, while <u>*target*</u> limits matching to join points where the target object (application object being proxied) is an instance of the given type. **The former works when Spring AOP creates a CGLIB-based proxy, and the latter is used when a JDK-based proxy is created.**

    ​

    예시)

    ```java
    public class FooDao implements BarDao {
        ...
    }
    ```

    ```java
    @Pointcut("target(org.foopack.dao.BarDao)")
    ```

    위와 같이 인터페이스 상속했을 때는 *target*을 써야한다.

    FooDao가 인터페이스를 상속받지 않거나 proxyTargetClass 속성이 false로 되어있으면 proxy된 target 객체가 FooDao의 subclass이기 때문에 *this*를 사용해야한다.

    > AspectJ itself has type-based semantics and at an execution join point both `this` and `target` refer to the same object - the object executing the method. Spring AOP is a proxy-based system and differentiates between the proxy object itself (bound to `this`) and the target object behind the proxy (bound to `target`).

    ​

  - ***args*** - limits matching to join points (the execution of methods when using Spring AOP) where the **arguments are instances of the given types**
    ​

  - ***@target*** - limits matching to join points (the execution of methods when using Spring AOP) where the **class of the executing object has an annotation of the given type**

    ​

  - ***@args*** - limits matching to join points (the execution of methods when using Spring AOP) where the **runtime type of the actual arguments passed have annotations of the given type**(s)

    메소드 파라미터 중 지정한 타입의 어노테이션 붙은 파라미터가 있을 경우

    ```java
    @Pointcut("@args(org.foopack.aop.annotations.Entity)")
    public void methodsAcceptingEntities() {}

    @Before("methodsAcceptingEntities()")
    public void logMethodAcceptionEntityAnnotatedBean(JoinPoint jp) {
        logger.info("Accepting beans with @Entity annotation: " + jp.getArgs()[0]);
    }
    ```

    ​

  - ***@within*** - limits matching to join points within types that have the given annotation (the execution of methods declared in types with the given annotation when using Spring AOP)

    ​

  - ***@annotation*** - limits matching to join points where the subject of the join point (method being executed in Spring AOP) has the given annotation

    ​

  - Spring AOP also supports an additional PCD named `bean` - allows you to limit the matching of join points to a particular named Spring bean

    ```java
    bean(idOrNameOfBean)
    ```

    > The `bean` PCD operates at the ***instance* level** (building on the Spring bean name concept) rather than at the **type level** only (which is what weaving-based AOP is limited to).
    >
    > … where it is **natural and straightforward to identify specific beans by name.**

    ​

    예시 참고 : http://www.baeldung.com/spring-aop-pointcut-tutorial

  ​

  <u>포인트컷 표현식</u>	

  `execution` pointcut designator를 가장 많이 사용한다. 

  ```
  execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
              throws-pattern?)
  ```

  **접근자제한 - 리턴타입제한 - 패키지&클래스패턴 - 메소드이름패턴(파라미터패턴) - throws예외패턴**

  ​

  Pointcut expressions can be **combined** using '&&', '||' and '!'. It is also possible to refer to pointcut expressions by name.

  ```java
  @Pointcut("execution(public * *(..))")
  private void anyPublicOperation() {}

  @Pointcut("within(com.xyz.someapp.trading..*)")
  private void inTrading() {}

  @Pointcut("anyPublicOperation() && inTrading()")
  private void tradingOperation() {}
  ```

  Example shows three pointcut expressions: `anyPublicOperation` (which matches if a method execution join point represents the execution of any public method); `inTrading` (which matches if a method execution is in the trading module), and `tradingOperation` (which matches if a method execution represents any public method in the trading module).

  ​

  그 이외에 세부적인 표현식 쓰기 방식은 메뉴얼 참고!

  ​

  <u>Writing good pointcuts</u>

  On first encountering a pointcut declaration, **AspectJ will rewrite it into an optimal form** for the matching process. What does this mean? Basically pointcuts are rewritten in DNF (Disjunctive Normal Form) and the components of the pointcut are sorted such that those components that are cheaper to evaluate are checked first. This means **you do not have to worry about understanding the performance of various pointcut designators and may supply them in any order in a pointcut declaration.**

  각 지정자나 지정자의 순서에 대한 성능은 걱정할 필요x

  그러나 지정자의 설명을 세부적으로 제공할수록 당연히 빠르고, 메모리 효율을 높여 최적의 성능을 낼 수 있음.
  아래 3가지 그룹으로 지정자를 구분할 수 있음.

  - **Kinded** designators are those which select **a particular kind** of join point. For example: execution, get, set, call, handler
  - **Scoping** designators are those which select **a group** of join points of interest (of probably many kinds). For example: within, withincode
  - **Contextual** designators are those that match (and optionally bind) based on context. For example: this, target, @annotation

  잘쓰여진 pointCut은 Scoping을 포함해 2개 이상을 사용한다.(combining)

  > Supplying either just a kinded designator or just a contextual designator will work but could affect weaving performance (time and memory used) due to all the extra processing and analysis. Scoping designators are very fast to match and their usage means AspectJ can very quickly dismiss groups of join points that should not be further processed 



- #### Adivce 정의

  포인트컷 표현식으로 매칭되는 메서드 excution에 before, after, around 방식으로 적용한다.

  문서로 고고

  https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-advice

  어드바이스 메소드의 파라미터 타입으로 구체적인 포인트컷 제한 가능. -> 아래 passing parameter 설명.

  - Around advice

    ```java
    @Aspect
    @Component
    public class PerformanceAspect {
     
        private Logger logger = Logger.getLogger(getClass().getName());
     
        @Pointcut("within(@org.springframework.stereotype.Repository *)")
        public void repositoryClassMethods() {};
     
        @Around("repositoryClassMethods()")
        public Object measureMethodExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
            long start = System.nanoTime();
            Object retval = pjp.proceed();// start
            long end = System.nanoTime();
            String methodName = pjp.getSignature().getName();
            logger.info("Execution of " + methodName + " took " + 
              TimeUnit.NANOSECONDS.toMillis(end - start) + " ms");
            return retval; // stop
        }
    }
    ```

    메소드 실행 시간 재는 예시.

    아래 인용 :  결과값은 캐싱되고 proceed는 여러번 호출될 수 있다는 내용

    > The value returned by the around advice will be the return value seen by the caller of the method. A simple caching aspect for example could return a value from a cache if it has one, and invoke proceed() if it does not. Note that proceed may be invoked once, many times, or not at all within the body of the around advice, all of these are quite legal.

  ​

  - Advice Parameters

    1. access to current JoinPoint

       Any advice method may declare as its first parameter, a parameter of type`org.aspectj.lang.JoinPoint`

       `JoinPoint` interface provides a number of useful methods such as `getArgs()` (returns the method arguments), `getThis()` (returns the proxy object), `getTarget()` (returns the target object), `getSignature()` (returns a description of the method that is being advised) and `toString()` (prints a useful description of the method being advised).

       첫번째 파라미터로 JoinPoint 타입을 받으면 해당 JoinPoint 인터페이스의 api들을 사용할 수 있음.

       ​

    2. passing parameters

       ```java
       @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
       public void validateAccount(Account account) {
           // ...
       }
       ```

       위와 같이 어드바이스에 파라미터를 명시해서 Account 객체를 첫째 파라미터로 받는 dao operation만 advising가능하다.

       > An example should make this clearer. Suppose you want to advise the execution of dao operations that take an Account object as the first parameter, and you need access to the account in the advice body.
       >
       > ...
       >
       > The `args(account,..)` part of the pointcut expression **serves two purposes**: **firstly**, it restricts matching to only those method executions where the method takes at least one parameter, and the argument passed to that parameter is an instance of `Account`; **secondly**, it makes the actual `Account` object available to the advice via the `account`parameter.

       ​

    3. Advice parameters and generics

       위 2번의 기능은 제네릭도 커버한다.

       ```java
       public interface Sample<T> {
           void sampleGenericMethod(T param);
           void sampleGenericCollectionMethod(Collection<T> param);
       }
       ```

       아래와 같이 하나는 가능 o

       ```java
       @Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
       public void beforeSampleMethod(MyType param) {
           // Advice implementation
       }
       ```

       콜렉션의 제네릭은 불가능(각 element들을 일일히 다루는게 비합리적임 - null다루는것같은)

       ```java
       @Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
       public void beforeSampleMethod(Collection<MyType> param) {
           // Advice implementation
       }
       ```

       ​

    4. Determining argument names

       both the advice and the pointcut annotations have an optional **"argNames" attribute** which can be used to specify the argument names of the annotated method 

       ```java
       @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
               argNames="bean,auditable")
       public void audit(JoinPoint jp, Object bean, Auditable auditable) {
           AuditCode code = auditable.value();
           // ... use code, bean, and jp
       }
       ```

       argNames로 bean, auditable을 명시하여 매칭하여 사용한다. - 런타임에 사용가능

       JoinPoint, ProceedingJoinPoint, JoinPoint.StaticPart 객체의 경우는 그냥 쓰면 됨.

       'argNames' 속성을 지정하지 않아도 되고, 그러면 스프링 AOP가 해당 클래스의 디버그 정보를 검사하고 로컬변수 테이블에서 파라미터 이름을 결정하려고 시도한다. 디버그 정보 없이 코드 컴파일하면 spring aop가 변수와 파라미터를 바인딩하는 연결을 추론한다.

       -> 바인딩이 모호하거나 실패하게 되면 AmbiguousBindingException or IllegalArgumentException을 던진다.

       ​

    5. 어드바이스 순서

       여러 어드바이스들이 같은 joinpoint에서 실행되려고 하면, Aspectj의 우선순위 규칙과 같게 Spring Aop가 순서를 정한다.

       before -> 높은 우선순위가 먼저

       after -> 높은 우선순위가 더 나중에

       `rg.springframework.core.Ordered` interface를 구현하거나, Order어노테이션 사용해서 순서를 제어할 수도 있다.

       ​

    ​

- #### Intorduction

  advised objects implement a given interface, and to provide an implementation of that interface on behalf of those objects.

  @DeclareParents 어노테이션으로 사용 가능하다.

  <u>예시</u>

  Basic interface

  ```java
  package pl.beans;

  public interface Performance {
      public void perform();
  }
  ```

  ```java
  @Component
  public class Actor implements Performance {

  	private static final String WHO = "Actor: ";

  	@Override
  	public void perform() {
      	System.out.println(WHO+"Making some strange things on scene");
     	}

  }
  ```

  New interface

  ```java
  package pl.introduction;

  public interface Crazy {

      public void doSomeCrazyThings();

  }
  ```

  ```java
  package pl.introduction;

  public class CrazyActor implements  Crazy {

  	@Override
  	public void doSomeCrazyThings() {
      	System.out.println("Ługabuga oooo  'Performer goes crazy!'");
      }
  }
  ```

  Aspect

  ```java
  package pl.introduction;

  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.DeclareParents;

  @Aspect
  public class CrazyIntroducer {

      @DeclareParents(value="pl.beans.Performance+", defaultImpl=pl.introduction.CrazyActor.class)
      public static Crazy shoutable;

  }
  ```

  위와 같이 작성하면 Perfomance 인터페이스를 구현하는 Bean들(Actor)은 CrazyActor클래스(디폴트 구현체)를 상속하게 되어 Crazy 인터페이스의 기능도 사용할 수 있게 되는것.

  -> 코드 내용 변경없이 메소드 및 필드 추가 가능한것임.

  ​

- Aspect instantiation models

  고급 토픽이라 나중에 봐도 된다는 이야기 있었음.

  ​

  > By default there will be a single instance of each aspect within the application context. AspectJ calls this the singleton instantiation model.

  디폴트로 각각의 aspect는 application context내에 하나의 인스턴스로 존재할 것임.

  -> 이걸 singleton instantiation model이라 함.

  ​

  싱글톤 말고 다른 생명주기를 가질 수 있도록 할 수 도 있음.

  > Spring supports AspectJ’s `perthis` and `pertarget` instantiation models

  perthis / pertarget으로!

  ```java
  @Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
  public class MyAspect {

    private int someState;
      
    @Before(com.xyz.myapp.SystemArchitecture.businessService())
    public void recordServiceUsage() {
      // ...
    }    
  }
  ```

  - aspect는 businessService 메소드(aspect내의 포인트컷 메소드)가 호출될때 최초로 생성된다.
  - 호출 될 때 마다 생성되므로 someState값을 각 호출때마다 독립적으로 사용가능할듯?

  pertarget도 똑같이 동작함



- Example

  비즈니스 서비스 실행이 동시성 이슈 때문에 실패하는 경우가 많음.
  -> 작업이 재시도되면 성공할 가능성이 높은 작업들이 있음(멱등작업)

  ```java
  @Aspect
  public class ConcurrentOperationExecutor implements Ordered {

      private static final int DEFAULT_MAX_RETRIES = 2;

      private int maxRetries = DEFAULT_MAX_RETRIES;
      private int order = 1;

      public void setMaxRetries(int maxRetries) {
          this.maxRetries = maxRetries;
      }

      public int getOrder() {
          return this.order;
      }

      public void setOrder(int order) {
          this.order = order;
      }

      @Around("com.xyz.myapp.SystemArchitecture.businessService()")
      public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
          int numAttempts = 0;
          PessimisticLockingFailureException lockFailureException;
          do {
              numAttempts++;
              try {
                  return pjp.proceed();
              }
              catch(PessimisticLockingFailureException ex) {
                  lockFailureException = ex;
              }
          } while(numAttempts <= this.maxRetries);
          throw lockFailureException;
      }

  }
  ```

  모든 businessService들의 작업이 실패하면 PessimisticLockingFailureException을 받고 최대 시도 횟수 동안 계속 재시도 하는 코드임.

  aspect는 Ordered 인터페이스 구현하므로 아래와 같이 우선순위 설정 가능

  ```xml
  <aop:aspectj-autoproxy/>

  <bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
      <property name="maxRetries" value="3"/>
      <property name="order" value="100"/>
  </bean>
  ```

  그리고 멱등성이 필요한 작업만 명시해주고 싶으면 아래와 같은 어노테이션을 붙이고 advice에 적용하면 된다.

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Idempotent {
      // marker annotation
  }

  -------

  @Around("com.xyz.myapp.SystemArchitecture.businessService() && " +
          "@annotation(com.xyz.myapp.service.Idempotent)")
  public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
      ...
  }
  ```



그리고 5.3은 schema-based AOP 방식 설명..