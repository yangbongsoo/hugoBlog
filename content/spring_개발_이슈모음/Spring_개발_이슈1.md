+++
title = "Spring Issue1"
+++

## 1. 통합테스트 애노테이션 정리

### Spring

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Application.class) 

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/app-config.xml")
```

`@ContextConfiguration`은 통합 테스트에서 클래스 레벨 메타데이터(xml 파일 or javaConfig 파일)를 정의한다. 다시 말해, context를 로드하는데 사용되는 annotated class(@Configuration 클래스)나 application context resource locations(classpath에 위치한 XML 설정 파일)들을 선언한다.

또한 `@ContextConfiguration`은 ContextLoader 전략을 사용할 수 있다. 하지만 일반적으로 로더를 직접 명시할 필요는 없다. default loader가 initializers 뿐만 아니라 resource locations 또는 annotated classes를 지원하기 때문이다.

---

**문제 발생**

Spring Boot에서 `@ContextConfiguration(classes = Application.class)`만 설정했더니

```
[main] DEBUG org.springframework.core.type.classreading.AnnotationAttributesReadingVisitor -
Failed to class-load type while reading annotation metadata. 
This is a non-fatal error, but certain annotation metadata may be unavailable.

java.lang.ClassNotFoundException:
org.springframework.data.web.config.EnableSpringDataWebSupport
...
java.lang.ClassNotFoundException:
org.springframework.security.config.annotation.web.configuration.EnableWebSecurity
...
```

위와 같은 에러뿐만 아니라 기본적인 Autowired 설정도 안되고, BeanCreationException도 발생했다.  
원인은 애노테이션이 classpath에 없기 때문에 클래스가 로드될 때 JVM이 drop 시켜서 발생한 문제였다.

**해결 방법1**

Spring Boot에서는 기존의 `@ContextConfiguration` 대신 `@SpringApplicationConfiguration`을 제공한다. ApplicationContext 설정을 `@SpringApplicationConfiguration`으로 사용하면 SpringApplication 으로 생성되고 추가적인 Spring Boot feature들을 얻을 수 있다.

**해결 방법2**

```java
@ContextConfiguration(classes = Application.class, 
loader = SpringApplicationContextLoader.class)
```

그런데 1.4부터 SpringApplicationContextLoader가 Deprecated 되었다.

**해결 방법3**

```java
@ContextConfiguration(classes = Application.class, 
initializers = ConfigFileApplicationContextInitializer.class)
```

ConfigFileApplicationContextInitializer는 Spring Boot application.properties파일을 로드해 테스트 코드에 적용한다. `@SpringApplicationConfiguration`가 제공하는 full feature들이 필요 없을 때 사용된다.

cf) spring boot 1.4

직접적인 Configuration 설정 없이도 `@*Test` 애노테이션이 자동으로 primary configuration을 찾는다(테스트가 포함된 패키지로부터 `@SpringBootApplication`또는 `@SpringBootConfiguration` 애노테이션 클래스를 찾는다).

### Spring Boot

```java
// 방법1
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)

// 방법2 (1.4부터 SpringApplicationContextLoader Deprecated 됌)
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Application.class, loader = SpringApplicationContextLoader.class)

// 방법3
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Application.class, 
initializers = ConfigFileApplicationContextInitializer.class)

// 방법4 (spring boot 1.4부터 가능)
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
```

cf) 방법4에서 `@SpringBootTest`에서 classes 속성을 생략하면 inner-classes에서 `@Configuration`을 제일 먼저 로드하려 시도하고, 없으면 `@SpringBootApplication` class를 찾는다.

**@WebApplicationContext**

WebApplicationContext을 생성할 수 있게 해준다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
public class AccountControllerTest {

  @Autowired
  WebApplicationContext wac;
  MockMvc mockMvc;

  @Before
  public void setUp() {
      mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
  }

  ...
}
```

## 2. dataSource

메이븐 pom.xml에서 에서 spring-boot-starter-data-jpa 를 추가하면 그안에 spring-boot-starter-jdbc가 있고 그 안에 tomcat-jdbc가 있다.  
Spring Boot에서는 DataSource 관리를 위한 구현체로써 tomcat-jdbc(The Tomcat JDBC Pool) 을 default로 제공한다.

근데 실서버에 배포 했을 때 에러가 반복해서 발생했고 그 주기도 일정치 않아서 재연이 쉽지 않았다. `validationQuery: select 1`을 설정했음에도 connection이 자꾸 닫히는 문제가 발생했다.

```java
[2016-05-19 17:37:40.187] boot - 11886 ERROR [http-nio-8080-exec-3] --- SqlExceptionHelper: No operations allowed after connection closed.
[2016-05-19 17:37:40.188] boot - 11886 ERROR [http-nio-8080-exec-3] --- TransactionInterceptor: Application exception overridden by rollback exception
javax.persistence.PersistenceException: org.hibernate.exception.JDBCConnectionException: could not prepare statement
at org.hibernate.jpa.spi.AbstractEntityManagerImpl.convert(AbstractEntityManagerImpl.java:1763)
at org.hibernate.jpa.spi.AbstractEntityManagerImpl.convert(AbstractEntityManagerImpl.java:1677)
at org.hibernate.jpa.internal.QueryImpl.getResultList(QueryImpl.java:458)
at org.hibernate.jpa.criteria.compile.CriteriaQueryTypeQueryAdapter.getResultList(CriteriaQueryTypeQueryAdapter.java:67)
at org.springframework.data.jpa.repository.support.SimpleJpaRepository.findAll(SimpleJpaRepository.java:323)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:497)
at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.executeMethodOn(RepositoryFactorySupport.java:483)
at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.doInvoke(RepositoryFactorySupport.java:468)
at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.invoke(RepositoryFactorySupport.java:440)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
at org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor.invoke(DefaultMethodInvokingMethodInterceptor.java:61)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:281)
at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:136)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
at org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor.invoke(CrudMethodMetadataPostProcessor.java:131)
```

그래서 해결책으로 hikari cp를 사용했다.

```xml
//관련 의존성 추가
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.4.6</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

```java
//설정 추가 
@Configuration
@ConfigurationProperties(prefix = "hikari.datasource")
public class JpaConfig extends HikariConfig {

    @Bean
    public DataSource dataSource() throws SQLException{

        return new HikariDataSource(this);
    }
}
```

```
// yml 추가
hikari:
  datasource:
    jdbcUrl: jdbc:mysql://blabla
    driverClassName: com.mysql.jdbc.Driver
    username: blabla
    password: blabla
    maximum-pool-size: 5
    connection-test-query: select 1
```

**그러나 설정파일에 hikari 의존이 생기는 건 좋지 않다.**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
      <exclusion>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-jdbc</artifactId>
      </exclusion>
    </exclusions>
</dependency>
```

위와 같이 tomcat에서 제공하는 기본 dataSource를 지우면 boot에서 자동으로 hikari cp로 설정된다. 따라서 JpaConfig 파일도 필요 없게 되고 yml에서 hikari를 지워도 된다.

---

**2016.08.12 추가**

Spring Boot 1.4부터는 db 프로파일에 `spring.datasource.hikari`를 명시적으로 작성할 수 있게 됐다.

## 3. request url에 문자열 ".t"을 확장자로 인식

```java
@RequestMapping(value = "/messages/receiverUsername/{receiverUsername:.+}", method = RequestMethod.GET)
```

위와 같이 정규식으로 모든 문자열을 (. 포함) 받겠다고 명시했음에도 .t는 인식이 안되는 문제가 발생했다.

이유는 .t라는 확장자가 있기 때문에 아래와 같이 설정에서 mediaType을 json으로 직접 명시해주면 해결이 가능하다.

```java
@Override
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(true);
    configurer.useJaf(false);
    configurer.ignoreAcceptHeader(true);
    configurer.mediaType("json", MediaType.APPLICATION_JSON);
}
```

## 4. Spring MVC Test

Spring Framework 4.2.6에서 Spring MVC Test를 처음 생성하다 다음과 같은 에러가 발생했다.

```java
java.lang.NoClassDefFoundError: javax/servlet/SessionCookieConfig
```

문제는 org.springframework.mock.web에 있는 mock set들은 Servlet 3.0 API를 기반으로 동작하는데 현재 프로젝트 Servlet 버전은 2.5였다.

```xml
변경 전
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>

변경 후
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

The Spring 4.0.1 reference documentation is now more clear about the Mocks: **Servlet 3.0+ is strongly recommended and a prerequisite in Spring's test and mock packages for test setups in development environments.**

## 5. 날짜

```java
// 기존 날짜 데이터 처리 방식
long plusDay = xxxSchedule.getExecutionDate().getTime() + TimeUnit.DAYS.toMillis(extendDays);

xxxSchedule.setRegisterDate(Date.from(Instant.now()));
xxxSchedule.setExecutionDate(new Date(plugDay));

// 자바8 현재 날짜 구하기
LocalDateTime.now()
LocalDate.now().format(DateTimeFormatter.ofPattern("yyyyMMdd"));

// 타임스탬프에서 날짜 가져오기
Long time = 1470651527000L;
DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
String format = dateFormat.format(time);
```

## 6. Mockito

**verify**

```java
verify(xxxRepository, never()).save(any(xxx.class));
```

`never()` : 어떤 조건에 따라 호출되면 안되는 경우, 진짜 호출이 안되는지 확인.

`<T> T any(Class<T> clazz)`: anyObject와는 다르게 클래스를 지정할 수 있다.

**spy**  
해당 서비스안에 있는 일반 메서드를 `when().thenReturn()`을 통해 제어하고 싶을 때 서비스를 spy로 만들어서 할 수 있다.

```java
xxxService xxxServiceSpy = Mockito.spy(xxxService);
```

**ArgumentCaptor**  
참고 : [http://stackoverflow.com/questions/12295891/how-to-use-argumentcaptor-for-stubbing](http://stackoverflow.com/questions/12295891/how-to-use-argumentcaptor-for-stubbing)

stub을 만들지 않고, 메서드가 호출됐는지 확인하는 것과 인자가 정확한지 확인하고 싶을 때 ArgumentCaptor방식을 사용할 수 있다.

```java
ArgumentCaptor<SomeClass> argumentCaptor = ArgumentCaptor.forClass(SomeClass.class);
verify(someObject).doSomething(argumentCaptor.capture());
assertNull(argumentCaptor.getValue().getXXX());
```

## 7. YAML

기존 spring 프로젝트에서 yml 설정을 하기 위해서 먼저 의존성을 추가해준다.

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.17</version>
</dependency>
```

그리고 서블릿 컨텍스트에 yaml 설정을 추가해준다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd">

<bean id="yamlProperties" class="org.springframework.beans.factory.config.YamlPropertiesFactoryBean">
        <property name="resources" value="classpath:application.yml" />
</bean>

<context:property-placeholder properties-ref="yamlProperties" />
```

cf) spring boot에서는 classpath 에 application.yml 을 추가 하면 자동으로 boot가 스캔하기 때문에 따로 설정이 필요없다.

```
    <build>
        <!-- Turn on filtering by default for application properties -->
        <resources>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/application.yml</include>
                    <include>**/application.properties</include>
                </includes>
            </resource>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
                <excludes>
                    <exclude>**/application.yml</exclude>
                    <exclude>**/application.properties</exclude>
                </excludes>
            </resource>
        </resources>
    </build>
```

**application.yml 작성 예**

```
spring:
  profiles.active: local

---
spring:
  velocity:
    properties.input.encodig: UTF-8
    properties.output.encodig: UTF-8
    cache: false
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL5Dialect
    hibernate:
      ddl-auto: validate
      naming-strategy: org.hibernate.cfg.ImprovedNamingStrategy
    properties:
      hibernate:
        temp:
          use_jdbc_metadata_defaults: false
---
spring:
  profiles: h2-db

  datasource:
    jdbcUrl: jdbc:h2:mem:AZ;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driverClassName: org.h2.Driver
    username: xxx
    password: xxx
    maxWait: 1000

  h2:
    console:
      enabled: true
      path: /h2-console

  jpa:
    show-sql: false
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: create
      naming-strategy: org.hibernate.cfg.ImprovedNamingStrategy
    properties:
      hibernate:
        temp:
          use_jdbc_metadata_defaults: false
logging:
  level: info

---
spring:
  profiles: real-db

  datasource:
    jdbcUrl: xxx
    driverClassName: com.mysql.jdbc.Driver
    username: xxx
    password: xxx
    maximum-pool-size: 5
    connection-test-query: select 1
---
spring:
  profiles: aws-db-test

  datasource:
    jdbcUrl: xxx
    driverClassName: com.mysql.jdbc.Driver
    username: xxx
    password: xxx
    maximum-pool-size: 5
    connection-test-query: select 1
---
spring:
  profiles: local

  datasource:
    url: xxx
    driverClassName: com.mysql.jdbc.Driver
    username: xxx
    password: xxx
    maxWait: 1000
    validationQuery: select 1
```

## 8. 기존 xml 설정을 JavaConfig로 변경하기

먼저 web.xml에서 contextConfigLocation을 xml 파일 위치 대신 AppConfig 위치로 변경해준다.

```xml
<!-- Processes application requests -->
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:spring/servlet-context.xml
        </param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

```xml
<!-- Processes application requests -->
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </init-param>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            (패키지명)xxx.xxx.xxx.xxx.config.AppConfig
        </param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

그리고 ContextLoaderListener가 자동으로 생성하는 컨텍스트의 클래스는 기본적으로 `XmlWebApplicationContext`다. 이를 다른 애플리케이션 컨텍스트 구현 클래스로 변경하고 싶으면 contextClass 파라미터를 이용해 지정해주면 된다.

두번째로 AppConfig 파일에 기존 xml에 있던 설정들을 옮긴다.

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang.StringUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.config.YamlProcessor;
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.yaml.SpringProfileDocumentMatcher;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.client.RestTemplate;

import java.io.IOException;
import java.util.Properties;

@Configuration
@Slf4j
@EnableJpaRepositories(basePackages = "xxx.xxx.xxx.xxx", transactionManagerRef = "xxxTransactionManager")
@EnableScheduling
@EnableAsync
@EnableTransactionManagement
@ComponentScan("xxx.xxx.xxx")
@ImportResource({"classpath:spring/applicationContext-db.xml", "classpath:spring/applicationContext-jpa.xml"})
public class AppConfig implements EnvironmentAware, ApplicationContextAware {
    private ApplicationContext ctx;
    private Environment env;

    @Bean
    RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

    @Bean
    public YamlProcessor.DocumentMatcher documentMatcher() {
        String[] profile = getProfile();
        log.info("ActiveProfile: {}", (Object[]) profile);
        return new SpringProfileDocumentMatcher(profile);
    }

    @Bean(name = "yamlProperties")
    public YamlPropertiesFactoryBean yamlPropertiesFactoryBean(YamlProcessor.DocumentMatcher documentMatcher) throws IOException {
        YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        yamlPropertiesFactoryBean.setResources(ctx.getResources("classpath:application.yml"));
        yamlPropertiesFactoryBean.setDocumentMatchers(documentMatcher);
        return yamlPropertiesFactoryBean;
    }

    @Bean 
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer(@Qualifier("yamlProperties") Properties yamlProperties) {
        PropertySourcesPlaceholderConfigurer p = new PropertySourcesPlaceholderConfigurer();
        p.setProperties(yamlProperties);
        return p;
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.env = environment;
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.ctx = applicationContext;
    }

    private String[] getProfile() {
        String[] activeProfiles = env.getActiveProfiles();
        if (activeProfiles != null) {
            return activeProfiles;
        }

        Properties properties = System.getProperties();
        String profileActive = properties.getProperty("spring.profiles.active");

        return new String[] { StringUtils.defaultString(profileActive, "dev") };
    }
}
```

## 9. Mybatis db 언더바 사용된 컬럼과 자바 카멜케이스 변수 자동 매핑

방법1 : application.yml

```
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

방법2 : mybatis-config.xml

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

cf) 다른 설정정보 참고 : [http://www.mybatis.org/mybatis-3/ko/configuration.html](http://www.mybatis.org/mybatis-3/ko/configuration.html)

## 10. @RequestParam값이 Optional일때 DefaultValue 적용

```java
@RequestParam(value = "bongsoo", required = false) int bongsoo
```

int형이기 때문에 값이 없을때 null일 수 없다. 그래서 defaultValue를 미리 지정해주는게 좋다.

```java
@RequestParam(value = "bongsoo", required = false, defaultValue="0") int bongsoo
```

## 11. 세션 스코프 빈 사용

컨트롤러에서 세션 스코프 빈을 사용할 일이 있었고, 컨트롤러는 싱글톤 빈이기 때문에 일반적으로 DI방식을 이용해 주입해서는 방법이 없다. DL 방식을 이용하는 방법도 있지만 애플리케이션 로직에 스프링 코드가 들어간다는 단점이 있다.

그래서 프록시 DI 방식을 택했다.

```java
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
@Component
public class Work {
  ...
}
```

@Scope 애노테이션으로 스코프를 지정했다면 proxyMode 엘리먼트를 이용해서 프록시를 이용한 DI가 되도록 지정할 수 있다. 클라이언트(여기선 컨트롤러 클래스)는 스코프 프록시 오브젝트를 실제 스코프 빈처럼 사용하면 프록시에서 현재 스코프에 맞는 실제 빈 오브젝트로 작업을 위임해준다.

스코프 프록시는 각 요청에 연결된 HTTP 세션정보를 참고해서 사용자마다 다른 Work 오브젝트를 사용하게 해준다. 클라이언트인 컨트롤러 입장에서는 모두 같은 오브젝트를 사용하는 것처럼 보이지만, 실제로는 그 뒤에 사용자별로 만들어진 여러 개의 Work가 존재하고, 스코프 프록시는 실제 Work 오브젝트로 클라이언트의 호출을 위임해주는 역할을 해줄 뿐이다.

프록시 빈이 인터페이스를 구현하고 있고, 클라이언트에서 인터페이스로 DI 받는다면 proxyMode를 ScopedProxyMode.INTERFACES로 지정해주고, 프록시 빈 클래스를 직접 DI 한다면 ScopedProxyMode.TARGET_CLASS로 지정하면 된다(여기서는 Work 클래스로 직접 DI 할 것이므로 ScopedProxyMode.TARGET_CLASS).

```java
스코프 프록시의 DI 사용

@Controller
public void MainController {

  @Autowired
  Work work;
}
```

cf) XML 설정방식

```xml
<bean id="work" class="...Work" scope="session">
  <aop:scoped-proxy proxy-target-class="true"/>
</bean>
```

DI 받을 때 클래스를 이용한다면 proxy-target-class를 true로 설정하고, 인터페이스를 이용한다면 false로 하거나 아예 생략하면 된다.

## 12. Spring Boot war 배포로 변경하기

1. pom.xml
   ```xml
   변경 전 : <packaging>jar</packaging>
   변경 후 : <packaging>war</packaging>
   ```
2. configuration

   ```java
   @SpringBootApplication
   public class Application extends WebMvcConfigurerAdapter {
      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
        }

    ...
   }
   ```

   ```java
   @Configuration
   public class AppConfig extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
   }
   ```

## 13. Spring Boot CORS 처리

대부분의 웹 브라우저는 보안상의 이유로 다른 도메인의 URL을 호출해서 데이터를 가져오는 것을 금지하고 있다. 우리 웹 서비스에서만 사용하기 위해 다른 서브 도메인을 가진 API 서버를 구축했는데, 다른 웹 서비스에서 마음대로 접근해서 사용하면 문제가 되기 때문이다.

그런데 하나의 도메인을 가진 웹 서버에서 모든 처리를 하기에는 효율성이나 성능 등 여러 문제로 각 기능별로 여러 서버를 두는 경우가 많다(API 서버, WAS 서버, 파일 서버 등등). 물리적으로 분리된 서버이고, 다른 용도로 구축된 서버이니 당연히 각각 다른 도메인을 가진 서버들일 텐데, 서로간에 Ajax 통신을 할 수 없는 것일까? 즉 서로 다른 도메인 간의 호출을 의미하는 크로스 도메인 문제를 해결할 수는 없는 것일까?

CORS(Cross Origin Resource Sharing)은 외부 도메인에서의 요청(접근)을 허용해주는 메커니즘이다.

원문 : [http://ooz.co.kr/232](http://ooz.co.kr/232)

```java
public class CORSFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 모든 도메인에 대해 허용하겠다는 의미
        response.addHeader("Access-Control-Allow-Origin", "*");
        response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
        response.addHeader("Access-Control-Allow-Headers", "content-type, accept, api_id, api_key");
        response.addHeader("Access-Control-Max-Age", "1800");
        filterChain.doFilter(request, response);
    }
}
```

위와 같이 필터 클래스를 만들고

```java
    @Bean
    public FilterRegistrationBean registration(){
        FilterRegistrationBean register = new FilterRegistrationBean();
        CORSFilter corsFilter = new CORSFilter();
        register.setFilter(corsFilter);
        register.setEnabled(true);
        register.setUrlPatterns(Collections.singletonList("/*"));
        return register;
    }
```

빈으로 등록만 해주면 url 패턴에 따라 필터가 작동된다.

### CORS support in Spring Framework

[http://spring.io/blog/2015/06/08/cors-support-in-spring-framework](http://spring.io/blog/2015/06/08/cors-support-in-spring-framework)

@CrossOrigin 애노테이션을 추가해줌으로써 CORS를 가능하게 해준다.

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @RequestMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

컨트롤러 전체에 적용도 가능하다.

```java
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @RequestMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

**Java Config**

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(false).maxAge(3600);
    }
}
```

**XML namespace**

```xml
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="http://domain1.com, http://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="false"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="http://domain1.com" />

</mvc:cors>
```

## 14. @JsonInclude @Pattern

```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
    private Long id;

    @NotEmpty
    @Pattern(regexp = "[a-zA-Z\\d._-]{6,100}")
    private String name;

    @NotEmpty
    @Length(min = 6, max = 100)
    @Pattern(regexp = "^[a-zA-Z\\d.-_]+@[a-zA-Z\\d-]+(.[a-z]+)+$")
    private String email;

    @NotEmpty
    @Length(min = 6)
    private String password;

    private Date regDt;
}
```

`@JsonInclude(JsonInclude.Include.NON_NULL)` 객체를 Json으로 변환할 때, null값 필드는 아예 Json으로 매핑 안되게 해주는 설정이다.

## 15. Swagger를 조회하는 URL 바꾸고 싶을 때

```java
@Controller
public class ApiDocController {
    @GetMapping("/swagger-ui")
    public void apiDocs (OutputStream response) throws IOException {
        InputStream source = new ClassPathResource("/META-INF/resources/swagger-ui.html").getInputStream();
        StreamUtils.copy(source, response);
    }
}
```

## 16. WebApplicationInitializer를 이용한 컨텍스트 등록

서블릿 3.0 이상부터 사용 가능하다. web.xml 파일만을 단독으로 사용하던 것에서 탈피해, 설정 방식을 모듈화해서 관리하는 방법을 도입했다.

```java
@Order(value = 1)
public class WebAppIntializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        //parent
        AnnotationConfigWebApplicationContext rootContext = new AnnotationConfigWebApplicationContext();
        rootContext.register(AppConfig.class);

        servletContext.addListener(new ContextLoaderListener(rootContext));
//        new ContextLoader(rootContext).initWebApplicationContext(servletContext);

        //child
        AnnotationConfigWebApplicationContext dispatcherServletContext = new AnnotationConfigWebApplicationContext();
        dispatcherServletContext.register(WebConfig.class);

        ServletRegistration.Dynamic dispatcher = servletContext.addServlet("dispatcher", new DispatcherServlet(dispatcherServletContext));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");

        //Encoding Filter 등록
        FilterRegistration.Dynamic encodingFilter = servletContext.addFilter("CharacterEncodingFilter", new CharacterEncodingFilter());
        encodingFilter.setInitParameter("encoding", "UTF-8");
        encodingFilter.setInitParameter("forceEncoding", "true");
        encodingFilter.addMappingForUrlPatterns(null, true, "/*");
```

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <configuration>
   <webappDirectory>deploy</webappDirectory>
   <failOnMissingWebXml>false</failOnMissingWebXml>
  </configuration>
</plugin>
```

### 17. maven-clean-plugin

mvn clean을 수행하면 기본적으로 target 디렉토리 밑에 있는 내용을 삭제한다. 그런데 target 디렉토리 외에 다른 디렉토리도 지우고 싶다면, pom.xml에 직접 설정할 수도 있다.

```xml
<plugin>
  <artifactId>maven-clean-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <filesets>
      <fileset>
        <directory>deploy/WEB-INF/lib</directory>
      </fileset>
      <fileset>
        <directory>deploy/WEB-INF/classes</directory>
      </fileset>
    </filesets>
  </configuration>
</plugin>
```

### 18. maven profiles

```xml
    <properties>
        <java-version>1.8</java-version>
        <env>local</env>
    </properties>

    ...


    <profiles>
        <profile>
            <id>local</id>
            <properties>
                <env>local</env>
            </properties>
        </profile>
        <profile>
            <id>dev</id>
            <properties>
                <env>dev</env>
            </properties>
        </profile>
    </profiles>

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
            <resource>
                <directory>src/main/resources-${env}</directory>
            </resource>
        </resources>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>${java-version}</source>
                    <target>${java-version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webappDirectory>target/deploy</webappDirectory>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 19. HandlerMethodArgumentResolver를 이용한 사용자 검증

HandlerMethodArgumentResolver 인터페이스를 구현하여 컨트롤러의 메서드 파라미터를 검증, 수정 할 수 있다.

```java
/**
 * Strategy interface for resolving method parameters into argument values in
 * the context of a given request.
 *
 * @author Arjen Poutsma
 * @since 3.1
 * @see HandlerMethodReturnValueHandler
 */
public interface HandlerMethodArgumentResolver {

    /**
     * Whether the given {@linkplain MethodParameter method parameter} is
     * supported by this resolver.
     * @param parameter the method parameter to check
     * @return {@code true} if this resolver supports the supplied parameter;
     * {@code false} otherwise
     */
    boolean supportsParameter(MethodParameter parameter);

    /**
     * Resolves a method parameter into an argument value from a given request.
     * A {@link ModelAndViewContainer} provides access to the model for the
     * request. A {@link WebDataBinderFactory} provides a way to create
     * a {@link WebDataBinder} instance when needed for data binding and
     * type conversion purposes.
     * @param parameter the method parameter to resolve. This parameter must
     * have previously been passed to {@link #supportsParameter} which must
     * have returned {@code true}.
     * @param mavContainer the ModelAndViewContainer for the current request
     * @param webRequest the current request
     * @param binderFactory a factory for creating {@link WebDataBinder} instances
     * @return the resolved argument value, or {@code null}
     * @throws Exception in case of errors with the preparation of argument values
     */
    Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
            NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception;

}
```

쿠키로 인증 구현 예

```java
public class CookieHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

    private XxxService xxxService;

    public CookieHandlerMethodArgumentResolver(XxxService xxxService) {
        this.xxxService = xxxService;
    }

    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.hasParameterAnnotation(Authentication.class)
                && methodParameter.getParameterType().equals(User.class);
    }

    @Override
    public Object resolveArgument(MethodParameter methodParameter,
                                  ModelAndViewContainer modelAndViewContainer,
                                  NativeWebRequest nativeWebRequest,
                                  WebDataBinderFactory webDataBinderFactory) throws Exception {

        HttpServletRequest servletRequest = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
        javax.servlet.http.Cookie cookie = WebUtils.getCookie(servletRequest, "loginCookie");
        if (StringUtils.isNotEmpty(cookie)) {
            String EncryptVal = cookie.getValue();
            String userId = JavaEnCrypto.Decrypt(EncryptVal);
            if (isVaild(userId)) {
                User user = new User();
                user.setUserName(userId);
                return user;
            }
        }
        return null;
    }

    private boolean isVaild(String userId) {
        if (xxxService.checkEmail(userId).isDuplicated()){
            return true;
        }
        else {
            return false;
        }
    }
}
```

설정은 다음과 같이 한다.

```java
//boot config
@SpringBootApplication
public class Application extends WebMvcConfigurerAdapter {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(authenticationResolver(xxxService));
    }

    @Bean
    public CookieHandlerMethodArgumentResolver authenticationResolver(xxxService xxxService) {
        CookieHandlerMethodArgumentResolver cookieHandlerMethodArgumentResolver = new CookieHandlerMethodArgumentResolver(xxxService);
        return cookieHandlerMethodArgumentResolver;
    }
}
```

```xml
//xml config
<mvc:annotation-driven>
    <mvc:argument-resolvers>
        <bean class="xxx.xxx.CookieHandlerMethodArgumentResolver">
            <constructor-arg>
                <bean class="xxx.xxx.xxxService"/>
            </constructor-arg>
        </bean>
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```

```java
//Controller class 

public ResponseEntity<ResultResponse> logout(@Authentication User user, HttpServletResponse response) {
    if (user != null) {}
    else {}
}
```

### 20. @JsonView

출처 url : [http://www.concretepage.com/spring-4/spring-mvc-4-rest-jackson-jsonview-annotation-integration-example](http://www.concretepage.com/spring-4/spring-mvc-4-rest-jackson-jsonview-annotation-integration-example)

```java
public class Profile {
   public interface PublicView {}
   public interface FriendsView extends PublicView {}
   public interface FamilyView extends FriendsView {}
}
```

```java
@RestController
@RequestMapping("/app")
public class UserController {
   @Autowired
   private UserService userService;

   @JsonView(Profile.PublicView.class)
   @RequestMapping(value = "/publicprofile", produces = MediaType.APPLICATION_JSON_VALUE)
   public List<User> getAllPublicProfile() {
      return userService.getAllUsers();
   }

   @JsonView(Profile.FriendsView.class)
   @RequestMapping(value = "/friendprofile", produces = MediaType.APPLICATION_JSON_VALUE)
   public List<User> getAllFriendsProfile() {
      return userService.getAllUsers();
   }

   @JsonView(Profile.FamilyView.class)
   @RequestMapping(value = "/familyprofile", produces = MediaType.APPLICATION_JSON_VALUE)
   public List<User> getAllFamilyProfile() {
      return userService.getAllUsers();
   }

}
```

```java
public class User {
   @JsonView(Profile.PublicView.class)
   private String userId;

   private String password;
   private int age;

   @JsonView(Profile.FamilyView.class)
   private long mobnum;
   @JsonView(Profile.FriendsView.class)
   private String mailId;
   @JsonView(Profile.PublicView.class)
   private String name;
   @JsonView(Profile.PublicView.class)
   private College college;
   @JsonView(Profile.PublicView.class)
   private Address address;

    ...
}
```

```java
public class College {
   @JsonView(Profile.PublicView.class)
   private String colName;
   @JsonView(Profile.FriendsView.class)
   private String colLocation;

    ...
}
```

```java
public class Address {
   @JsonView(Profile.FamilyView.class)
   private String houseNo;
   @JsonView(Profile.FriendsView.class)
   private String city;
   @JsonView(Profile.PublicView.class)
   private String country;

    ...
}
```

**결과 /app/publicprofile**  
![](/publicprofile.png)

**결과 app/friendprofile**  
![](/friendprofile.png)

**결과 app/familyprofile**  
![](/familyprofile.png)

### 21. Spring Boot Embedded Tomcat AJP 연동

Spring Boot의 Embeded WAS로 Tomcat을 쓸 경우 아래와 같은 설정으로 AJP 연동을 할 수 있다.

```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
    tomcat.addContextCustomizers((context) -> {
        StandardRoot standardRoot = new StandardRoot(context);
        standardRoot.setCacheMaxSize(100 * 1024);
        standardRoot.setCacheObjectMaxSize(4 * 1024);
    });
    if (tomcatAjpEnabled) {
        Connector ajpConnector = new Connector("AJP/1.3");
        ajpConnector.setProtocol("AJP/1.3");
        ajpConnector.setPort(ajpPort);
        ajpConnector.setSecure(false);
        ajpConnector.setAllowTrace(false);
        ajpConnector.setScheme("http");
        tomcat.addAdditionalTomcatConnectors(ajpConnector);
    }

    return tomcat;
}
```

참고로 AJP를 쓸 경우 HTTP 메서드 등 PATCH를 쓸 수 없다는 제약이 있다.



