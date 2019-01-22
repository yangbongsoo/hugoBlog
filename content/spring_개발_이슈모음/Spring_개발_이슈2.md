+++
title = "Spring Issue2"
+++

## 22. Spring Security X-Frame-Options 이슈

**Default Security Headers**
```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```
cf) Strict-Transport-Security는 HTTPS 요청일때만 추가된다. 

```
X-Frame-Options: DENY
```
response header에 X-Frame-Options를 갖고 있는 모든 사이트는 iframe 안에서 렌더링 되지 못하도록 브라우저가 막는다.

**customize**
```
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com/
```
SAMEORIGIN : 같은 도메인일때만 ifrmae을 허용한다.
```xml
<http>
	<!-- ... -->

	<headers>
		<frame-options
		policy="SAMEORIGIN" />
	</headers>
</http>
```

```java
http
	// ...
	.headers()
		.frameOptions()
			.sameOrigin();
```
ALLOW-FROM : 구체적인 도메인 지정

**Solution**
```xml
<http>
	<!-- ... -->

	<headers disabled="true" />
</http>

```
```java
http.headers().frameOptions().disable()
```

## 23. HTTP Strict Transport Security (HSTS)

https://docs.spring.io/spring-security/site/docs/4.0.x/reference/html/headers.html#headers-hsts

웹 브라우저가 HTTPS 프로토콜만을 사용해서 서버와 통신하도록 하는 기능을 한다. 만약 HTTPS로 접속에 실패하면 사이트 접근에 실패하게 된다. 서버가 HTTP 응답 헤더에 `Strict-Transport-Security`를 내려주면 브라우저는 그 사이트에 접속할 때 무조건 HTTPS로만 연결한다.

HSTS가 적용되기 위해서는 서버도 헤더를 내려줘야하고 브라우저도 그 헤더에 따른 동작을 해야하는 것이다.
HSTS를 사용하는 대신 서버에서 HTTP 접속을 HTTPS로 redirect 시키는 방법이 있지만 일단 한번 HTTP 연결을 거쳐가는 것이기 때문에 쿠키 정보 탈취 등 보안에 취약하다.

```
Strict-Transport-Security: max-age=31536000 ; includeSubDomains; preload
```
max-age : 지정 시간(단위 초)만큼 HTTPS를 사용

includeSubdomains : HSTS를 서브 도메인에도 적용

preload : 브라우저가 해당 사이트를 HSTS 적용 preload list에 추가하도록 함

**preload list**

HTTPS로 웹 사이트에 접속하기 위해 적어도 한번 웹 서버와 통신을 해야하는데, 통신 해보기 전에 미리 HTTPS로 접속하도록 목록을 미리 만들어둔 것이다.

이는 브라우저가 지원해주는 기능이고 브라우저 안에 기본적으로 내장되어 있는 사이트 목록들이 있다. 그리고 서버가 헤더에 preload값을 내려주면 이 목록에 해당 사이트도 추가하게 되는 것이다(브라우저에 캐시됌). preload에 추가된 사이트는 서버가 HSTS헤더를 삭제해도 브라우저에는 설정이 유지된다.

만약 내장되는 preload list에 추가되면 삭제되기까지(목록 삭제 및 사용자 브라우저 업데이트) 시간이 오래 걸리므로 추가는 신중하게 해야한다. preload list는 크롬이 관리하고 있고 대부분의 브라우저들(파이어폭스, 오페라, 사파리, IE11, Edge)이 이 목록을 같이 사용한다.

만약 수동으로 해제하고자 한다면 크롬은 `chrome://net-internals/#hsts`에 들어가서 Delete Domain에서 삭제할 도메인을 입력하고 삭제하면 된다.

HSTS 설정 코드
```xml
<http>
<!-- ... -->

<headers>
	<hsts include-subdomains="true" max-age-seconds="31536000" />
</headers>
</http>

```
```java
@EnableWebSecurity
public class WebSecurityConfig extends
WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) throws Exception {
	http
	// ...
	.headers()
	.httpStrictTransportSecurity()
	.includeSubdomains(true)
	.maxAgeSeconds(31536000);
	}
}

```
HSTS disable 코드
```xml
<http>
	<!-- ... -->

	<headers>
		<hsts disable="true"/>
	</headers>
</http>
```
```java
@EnableWebSecurity
public class WebSecurityConfig extends
		WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			// ...
			.headers()
				.httpStrictTransportSecurity().disable();
	}
}
```

## 24. List에 있는 value를 Mybatis foreach로 insert 하기
```java
public class TestDto {
    private List<String> list;

    // getter & setter
}
```
```xml
    <insert id="insertTest" parameterType="com.test.dto.TestDto">
        INSERT
        INTO test_table (id, order)
        VALUES
        <foreach collection="list" item="value" index="order" open="" separator="," close="">
            (#{value}, #{order})
        </foreach>
    </insert>
```

## 25. 컨트롤러에서 List 객체 받는 방법 정리
**방법1**
```js
$.ajax({
    url: "/test1",
    type: "post",
    contentType: 'application/json;charset=UTF-8',
    data: JSON.stringify(list)
    }).done(function (data, status, xhr) {
    ...
```
```java
@RequestMapping(value = "/test1", produces = {"application/json;charset=UTF-8"}, method = RequestMethod.POST)
public void test1(@RequestBody List<String> list) {
    ...
}
```
**방법2**
```js
$.ajax({
    url: "/test2",
    type: "post",
    contentType: 'application/json;charset=UTF-8',
    data: JSON.stringify({
        name: yangbongsoo,
        age: 29,
        list: list
        })
    }).done(function (data, status, xhr) {
    ...

```
```java
@RequestMapping(value = "/test2", produces = {"application/json;charset=UTF-8"}, method = RequestMethod.POST)
public void test2(@RequestBody TestDto testDto) {
    ...
}
```
```java
public class TestDto {
    private String name;
    private int age;
    private List<String> list;

    //getter and setter
}
```

## 26. Mac에서 Spring Boot의 시작이 느릴 때
Mac에서 Spring Boot의 시작이 느리다면 아래와 같이 hostname을 지정해 본다.
```
sudo scutil --set HostName MyMacBook
```
Spring Boot의 StartupInfoLogger 에서는 InetAddress.getLocalHost().getHostName();를 호출한다.
Mac에서는 Hostname이 지정되어 있지 않을 경우 해당 메서드 호출에 몇초가 걸리는 것으로 파악된다.

## 27. Tomcat8 UMASK 이슈
스프링 이슈는 아니지만 따로 적을 곳이 없어서 여기에 적음

파일 업로드를 할 때 업로드된 파일 권한이 위에서 아래로 바뀌는 현상 발견
```
-rw-r--r--
-rw-r-----
```

원인을 찾아보니 tomcat 7.0.68 catalina.sh에서는 UMASK가 주석처리 되어있었는데
```
#JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"
```

tomcat 8.5.32 catalina.sh에서 UMASK에 대한 부분이 바꼈다.
```
# Set UMASK unless it has been overridden
if [ -z "$UMASK" ]; then
    UMASK="0027"
fi
umask $UMASK

...

# Make the umask available when using the org.apache.catalina.security.SecurityListener
JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"
```

문제가 되는 이유는, 아파치 httpd.conf에서 User nobody를 할 경우에 640이면 파일 read를 못하게 된다.
기존 tomcat7에서는 read에 문제가 되지 않았던 부분이다.
```
file 666 - 022 = 644(-rw-r--r--)
file 666 - 027 = 640(-rw-r-----)
```

cf) 027은 `000 010 111`이고 보수로 전환하면 `111 101 000`이 된다.
UMASK 보수 값과 파일 기본 허가권을 AND 연산하면
```
110 110 110
111 101 000
-----------
110 100 000
```
640이 된다.

## 28. Maven에서 dependency 의 scope 설정
compile(default) : 컴파일 시 라이브러리를 사용한다. <br>
runtime : 실행 시 라이브러리를 사용한다. <br>
provided : 외부에서 라이브러리가 제공된다. 컴파일 시 사용하지만 빌드에 포함하지 않는다. 보통 JSP, Servlet 라이브러리들에 사용한다. <br>
test : 테스트 코드에만 사용한다. <br>

```xml
<mvc:annotation-driven/>
<context:component-scan base-package=""/>
```

base-package포함, 하위의 클래스들 중 @Controller, @Repository, @Service, @Component가 붙어 있는 클래스들을 자동으로 스프링 빈으로 등록한다.

```xml
<tx:annotation-driven/>
<context:component-scan base-package=""/>
```
bace-package포함, 하위의 클래스들 중 @Transcational이 붙은 곳에 트랜잭션을 적용한다.

`<context:component-scan> / <mvc:annotation-driven> / <context:annotation-config> 차이점`

1. context:component-scan
    1. 특정 패키지안의 클래스들을 스캔하고, 빈 인스턴스를 생성한다.
    2. @Component @Controller @Service @Repository 애노테이션이 존재해야 빈을 생성할 수 있다.
    3. 이것의 장점 중 하나는 @Autowired 와 @Qualifier 애노테이션을 이해한다는 것인데 component-scan을 선언했다면 context:annotation-config를 선언할 필요가 없다.
2. mvc:annotation-driven
    1. 스프링 MVC 컴포넌트들을 그것의 디폴트 설정을 가지고 활성화 하기위해 사용된다. 만약 context:component-scan을 XML 파일에서 빈을 생성하기 위해 사용하면서 mvc:annotation-driven을 포함시키지 않아도 MVC 애플리케이션은 작동할 것이다. 그러나 mvc:annotation-driven은 특별한 일들을 하는데 이 태그는 당신의 @Controllers에게 요청을 전파하기위해 요구되는 HandlerMapping과 HandlerAdapter를 등록한다. 게다가, 클래스패스상에 존재하는 디폴트 작업을 수행한다.
3. context:annotation-config
    1. context:annotation-config은 애플리케이션 컨텍스트안에 이미 등록된 빈들의 애노테이션을 활성화하기 위해 사용된다.(그것들이 XML로 설정됐는지 혹은 패키지스캐닝을 통한건지는 중요하지 않다.) 그 의미는 이미 스프링 컨텍스트에 의해 생성되어 저장된 빈들에 대해서 @Autowired와 @Qualifier 애노테이션을 해석할거란 얘기다. component-scan 또한 같은일을 할 수 있는데, 추가적으로 애플리케이션 컨텍스트에 빈을 등록하기위한 패키지들을 스캔한다. context:annotation-config는 빈을 등록하기 위해 검색할 수 없다.
    2. context:annotation-config 태그를 설정하면 @Required @Autowired @Resource @PostConstruct @PreDestroy @Configuration 기능을 각각 설정하는 수고를 덜게 해준다.

**mvc:resources**

```xml
<mvc:default-servlet-handler default-servlet-name="default"/>
```
DispatcherServlet이 처리하지 못한 요청을 서블릿 컨테이너의 DefaultServlet에게 넘겨주는 역할을 하는 핸들러이다.

/js/jquery.js 처럼 컨트롤러에 매핑안되는 URL같은 경우는 DefaultServletHttpRequestHandler가 담당한다. 이 핸들러는 매핑 우선순위가 가장 낮아서 애노테이션 매핑 등등을 거쳐서 다 실패한 URL만 넘어온다. 그리고 요청을 자신이 직접 읽어서 처리하는 것이 아니라, 원래 서버가 제공하는 디폴트 서블릿으로 넘겨버린다. 그러면 서버의 기본 디폴트 서블릿이 동작해서 스태틱리소스를 처리하는 것이다. 다시말해 일단 스프링이 다 받고 스프링이 처리 못하는 건 다시 서버의 디폴트 서블릿으로 넘긴다는 아이디어이다.

`*.ico`파일 처리 못하는 현상은 `web.xml`에 ico의 MIME타입을 지정해주면 된다.
```xml
<mime-mapping>
    <extension>ico</extension>
    <mime-type>image/vnd.microsoft.icon</mime-type>
</mime-mapping>
```

**정적자원 설정하기**
CSS,JS,이미지 등의 자원은 거의 변하지 않기 때문에, 웹 브러우저에 캐시를 하면 네트워크 사용량, 서버 사용량, 웹 브라우저의 반응 속도 등을 개선할 수 있다. 스프링 MVC를 이용하는 웹 애플리케이션에 정적 자원 파일이 함께 포함되어 있다면 웹 서버 설정을 사용하지 않고 캐시를 사용하도록 지정할 수 있다.

```xml
<mvc:resources mapping="/resources/**" location="/resources/" cache-period="60"/>
```

mapping : 요청 경로 패턴을 설정한다. (컨텍스트 경로를 제외한 나머지 부분의 경로)
location : 웹 애플리케이션 내에서 요청 경로 패턴에 해당하는 자원의 위치를 지정한다. 위치가 여러곳일 경우 각 위치를 콤마로 구분한다.
cache-period: 웹 브라우저에 캐시 시간 관련 응답 헤더를 전송한다. 초 단위로 캐시 시간을 지정하며 이 값이 0이면 웹 브라우저가 캐시하지 않도록 한다.

위 설정의 경우 요청 경로가 /resources/로 시작하면, 그에 해당하는 자원을 /resources/나 /WEB-INF/resources/ 디렉토리에서 검색한다.

**빈 설정방식의 변화**
![](/bean설정방식의변화.PNG)

1.x : 모든걸 `<bean> </bean>`으로 <br>
2.0.x : `<tx:annotation-driven` <br>
2.5x : @Service, @Repository 같이 설정 `<context:component-scan base-package="~"` <br>
3.0.x : @Configuration, @Bean <br>
3.1.x : @Enable~ ex) @EnableTransactionManagement <br>

![](/bean설정방식의변화2.PNG)

![](/ooo.PNG)

@EnableWebMvc는 `<mvc:annotation-driven>`과 똑같다. 따라서 애노테이션 드리븐을 설정하면 위의 주석에 해당하는 것들이 자동으로 등록이 된다. 기본적으로 Http메세지 컨버터도 등록이 되지만, JSON을 위한 MappingJackson2HttpMessageConverter는 직접 등록해줘야 사용할 수 있다.

---
**웹 환경에서 스프링 애플리케이션이 기동하는 방식**
![](/webapplicationcontext.PNG)
서블릿 컨테이너는 브라우저와 같은 클라이언트로부터 들어오는 요청을 받아서 서블릿을 동작시켜주는 일을 맡는다. 서블릿은 웹 애플리케이션이 시작될 때 미리 만들어둔 웹 애플리케이션 컨텍스트에게 빈 오브젝트로 구성된 애플리케이션의 기동 역할을 해줄 빈을 요청해서 받아둔다. 그리고 미리 지정된 메서드를 호출함으로써 스프링 컨테이너가 DI 방식으로 구성해둔 애플리케이션의 기능이 시작되는 것이다.

스프링은 이런 웹 환경에서 애플리케이션 컨텍스트를 생성하고 설정 메타정보로 초기화해주고, 클라이언트로부터 들어오는 요청마다 적절한 빈을 찾아서 이를 실행해주는 기능을 가진 DispatcherServlet이라는 이름의 서블릿을 제공한다. DispatcherServlet은 서블릿이 초기화 될때 자신만의 컨텍스트를 생성하고 초기화한다.
