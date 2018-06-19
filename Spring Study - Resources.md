###### Spring 5.0 Study

# Chapter 2 - Resources



low-level의 resource에 접근하기 위해 spring이 제공하는 인터페이스에 대한 설명



#### <u>2.2 The Resource interface</u>

Spring’s `Resource` interface is meant to be a more capable interface for abstracting
access to low-level resources.
스프링의 Resource 인터페이스는 로우 레벨의 리소스 접근을 추상화하는데 유용하다.



```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();

}

public interface InputStreamSource {

    InputStream getInputStream() throws IOException;

}
```

주요 메소드 설명

- `getInputStream()`: 리소스의 위치를 찾고 읽어서 inputStream타입으로 리턴해준다.
- `exists()`: 리소스가 실제로 존재하는지 boolean타입 리턴.
- `isOpen()`: 리소스의 
   true - 한번만 읽기 가능하고 읽은 후 close된다.
   fasle - 일반적인  값으로 쓰임
- `getDescription()`: 리소스에 대한 설명을 리턴하여 error output에 사용 - fully qualified file name or actual URL of the resource



#### <u>2.3 Built-in Resource implementations</u>

Resource 인터페이스에 대한 built-in 구현체들이 많이 있다.



1. UrlResource

   `UrlResource` wraps a `java.net.URL`

    URL로 접근가능하는 것들 : 파일들이나 HTTP, FTP 타겟들 등

   모든 URL들은 정규화된 string 포맷이 있다 ex) file:, http:, ftp:

   - UrlResource 생성자로 직접 생성가능

   -  PropertyEditor같은 클래스에서는 String값을 인자로 받아서 내부적으로 UrlResource를 생성하게 된다.

     (다만, classpath:와 같은 몇가지 prefix들이 붙은 것들에 대해서는 다른 Resource구현체들을 사용하고, prefix에 대한 인식을 못하면 standard URL string으로 간주해서 UrlResource로 만든다.)

   ​

2. ClassPathResource

   classpath 경로로 얻어지는 resource들에 쓰이는 구현체

   UrlResource에서 설명한 것과 반대로 됨.

   ​

3. FileSystemResource

   파일 시스템의 특정 파일로부터 정보 읽어옴

   ​

4. ServletContextResource

   This is a `Resource` implementation for `ServletContext` resources, interpreting relative paths within the relevant web application’s root directory

   (웹앱의 루트 디렉토리 기준으로 지정한 상대경로에 위치한 자원으로부터 정보 가져옴)

   ​

5. InputStreamResource

   inputStream으로부터 정보 읽을 때 사용.

   In contrast to other `Resource` implementations, **this is a descriptor for an *already* opened resource** - therefore returning `true` from `isOpen()`. **Do not use it if you need to keep the resource descriptor somewhere, or if you need to read a stream multiple times.**

   ​

6. ByteArrayResource

   useful for loading content from any given byte array, without having to resort to a single-use `InputStreamResource`

   ```java
   //DriveBo.java
   ByteArrayResource resource = new ByteArrayResource(bytes) {
       @Override
       public String getFilename() throws IllegalStateException {
           return originalFilename;
       }
   };
   multiValueMap.add("file", resource);
   ```



#### <u>2.4 The ResourceLoader</u>

Resource 인스턴스를 리턴해주는 인터페이스임.

```java
public interface ResourceLoader {

    Resource getResource(String location);

}
```

**All application contexts implement the `ResourceLoader` interface**, and therefore all application contexts may be used to obtain `Resource` instances.

(모든 어플리케이션 컨텍스트가 ResourceLoader 인터페이스 상속함)

각 application context에 따라 적절한 Resource 구현체로 리턴해준다.

**1. File system - FileSystemResource**

```java
Resource resource = appContext.getResource("file:c:\\testing.txt");
```

**2. URL path  - UrlResource**

```java
Resource resource = appContext.getResource("url:http://www.yourdomain.com/testing.txt");
```

**3. Class path - ClassPathResource**

```java
Resource resource = appContext.getResource("classpath:com/mkyong/common/testing.txt");
```



#### <u>2.5 The ResourceLoaderAware interface</u>

Bean은 application context를 통한 접근이 불가능하기 떄문에, ResourceLoader를 통해 resource에 접근하기 위한 방법으로 ResourceLoaderAware 인터페이스를 사용한다.

Implement the **ResourceLoaderAware** interface and create setter method for **ResourceLoader** object. Spring will DI the resource loader into your bean.

```java
public class CustomerService implements ResourceLoaderAware
{
	private ResourceLoader resourceLoader;
	
	public void setResourceLoader(ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}
		
	public Resource getResource(String location){
		return resourceLoader.getResource(location);
	}
}

...

public static void main( String[] args )
    {
    	ApplicationContext appContext = 
    	   new ClassPathXmlApplicationContext(new String[] {"Spring-Customer.xml"});
	
    	CustomerService cust = 
           (CustomerService)appContext.getBean("customerService");
    	
    	Resource resource = 
            cust.getResource("classpath:com/mkyong/common/testing.txt");
}
```

물론 ApplicationContextAware 인터페이스를 상속해도 가능하지만, resource load 목적에만 쓰인다면 ResourceLoaderAware를 상속하는 것이 리소스 로딩 기능에만 커플링 되도록 하므로 더 깔끔하다.

-> The code would just be coupled to the resource loading interface, which can be considered a utility interface, and not the whole Spring `ApplicationContext` interface. 



```java
@Component
 public class MyBean{
    @Autowired
    private ResourceLoader resourceLoader;
      ......
 }
```

위와같이 autowiring도 가능하다.



#### <u>2.6 Resource as dependencies</u>

위 Resourceloader는 동적인 resource path 처리를 위해 필요하다면, 정적인 경로의 경우 그냥 xml 설정으로 처리할 수 있다.

prefix가 없으면 application context에 맞게 알아서 들어감

아래는 myBean 클래스에 Resource타입인 template이라는 프로퍼티에 주입되는 경우이다.

```xml
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```



#### <u>2.7 Application contexts and Resource paths</u>

1. Constructing application contexts

   application context 생성자에서 string을 resource의 location path로 받아들인다.(빈 정의되어있는 xml file)

   prefix가 없으면 해당 application context에 맞추어 Resource 타입이 정해진다.

   ```java
   ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
   ```

   위는 빈 정의 들이 ClassPathResource가 사용되어 클래스패스로부터 로드될것임.

   ```java
   ApplicationContext ctx =
       new FileSystemXmlApplicationContext("conf/appContext.xml");
   ```

   위는 파일 시스템 경로로부터 로드될 것임.

   ​

   A `ClassPathXmlApplicationContext` instance composed of the beans defined in the`'services.xml'` and `'daos.xml'` could be instantiated like so…

   ```java
   ApplicationContext ctx = new ClassPathXmlApplicationContext(
       new String[] {"services.xml", "daos.xml"}, MessengerService.class);
   ```

   위처럼 string array로 xml file들의 이름을 열거하고, 클래스를 명시하면 알아서 경로정보를 찾아 인스턴스화하기도 한다.

   ​

2. WildCards in application context constructor resource paths(특별규칙)

   The resource paths in application context constructor values may be a simple path which has a one-to-one mapping to a target Resource, or **alternately may contain the special "classpath*:" prefix and/or internal Ant-style regular expressions (matched using Spring’s `PathMatcher` utility). Both of the latter are effectively wildcards**

   ##### Ant-style Patterns

   ```
   /WEB-INF/*-context.xml
   com/mycompany/**/applicationContext.xml
   file:C:/some/path/*-context.xml
   classpath:com/mycompany/**/applicationContext.xml
   ```

   **classpath*: prefix**

   ```java
   ApplicationContext ctx =
       new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
   ```

   The `classpath*:` prefix can also be combined with a `PathMatcher` pattern in the rest of the location path, for example `classpath*:META-INF/*-beans.xml`

   이하 경로 설정에 관한 설명들..