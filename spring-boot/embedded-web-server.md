# Embedded Web Servers
스프링 부트 웹 애플리케이션은 내장 웹 서버(embedded web server)를 포함하고 있다.

## Use Another Web Server
많은 스프링 부트 스타터에는 기본 내장 컨테이너가 포함되어 있다.
- servlet을 사용하는 애플리케이션의 경우, `spring-boot-starter-web`에는 `spring-boot-starter-tomcat`을 포함하고 있다. 대신 `spring-boot-starter-jetty` 또는 `spring-boot-starter-undertow`를 사용할 수도 있다.
- reactive를 사용하는 애플리케이션의 경우, `spring-boot-starter-webflux`에는 `spring-boot-starter-reactor-netty`를 포함하고 있다. 대신 `spring-boot-starter-tomcat`, `spring-boot-starter-jetty
` 또는 `spring-boot-starter-undertow`를 사용할 수도 있다.

다른 HTTP 서버로 전환 할 때는 기존의 dependency를 exclude하고 사용하고자 하는 starter를 추가하면 된다.

> `WebClient` 클래스를 사용하려면 `spring-boot-starter-reactor-netty`가 필요하므로 다른 HTTP 서버를 포함해야 하는 경우에도 Netty에 대한 dependency를 유지해야 한다.

## Disabling the Web Server
클래스 패스에 웹 서버를 시작하기 위한 의존성이 추가되어 있으면 스프링 부트는 자동으로 웹 서버를 시작한다. 웹 서버를 비활성화 하려면 다음과 같이 설정한다.

```yaml
spring.main.web-application-type=none
```

## Change the HTTP Port
standalone 애플리케이션에서 기본 HTTP 포트는 `8080` 이지만, `server.port` 를 사용해서 변경할 수 있다.

HTTP endpoint를 완전히 off 하고 사용하려면 `server.port=-1`을 사용하면 된다.

## Use a Random Unassigned HTTP Port
현재 사용 가능한 포트를 스캔해서 랜덤으로 사용하려면 `server.port=0`을 사용하면 된다.

## Discover the HTTP Port at Runtime
로그 출력 혹은 Webserver를 통해 `ServletWebServerApplicationContext`에서 사용중인 port 정보를 얻을 수 있다.

위의 방법도 있지만 사용중인 port 정보를 확인하는 가장 좋은 방법은 `ApplicationListener<ServletWebServerInitializedEvent>`의 `@Bean`을 추가해서 이벤트를 받아 사용하는 것이다.

`@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`를 사용하는 테스트에서는 `@LocalServerPort`를 사용해서 사용중인 포트를 주입받을 수도 있다.

> `@LocalServerPort`는 `@Value("${local.server.port}")`에 대한 meta-annotation이다.

## Enable HTTP Response Compression
HTTP response compression은 `Jetty`, `Tomcat` 그리고 `Undertow`에서 지원된다. 

```yaml
server.compression.enabled=true
```

압축을 수행하려면 기본적으로 response 길이가 2048 바이트 이상이어야 한다.
해당 기본 설정 값은 `server.compression.min-response-size` 속성 값을 이용해 변경 가능하다.

기본적으로 response 압축은 다음의 content-type의 경우 적용된다.
- text/html
- text/xml
- text/plain
- text/css
- text/javascript
- application/javascript
- application/json
- application/xml

위의 content-type 역시 `server.compression.mime-types` 속성 값으로 변경 가능하다.

## Configure SSL
SSL은 `application.properties` 또는 `application.yml`에서 `server.ssl.*` 속성 값을 설정해서 적용할 수 있다.

```yaml
server.port=8443
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=secret
server.ssl.key-password=another-secret
```

위의 설정을 적용하면 애플리케이션은 더 이상 8080 포트에서 plain HTTP connector를 지원하지 않는다. **스프링 부트는 `application.properties`를 통한 HTTP connector와 HTTPS connector를 동시에 지원하지 않는다.** 두 가지를 모두 사용하려면 하나의 connector를 직접 선언해야 한다. HTTP connector가 조금 더 설정하기 쉽기 때문에 `application.properties`를 사용해서 HTTPS connector를 설정하는 것이 좋다.

## Configure HTTP/2
`server.http2.enable` 속성을 통해서 스프링 부트 애플리케이션에서 HTTP/2 지원을 가능하게 설정할 수 있다. 이 설정은 사용하고 있는 웹 서버 및 애플리케이션 환경에 의존하고 있다. (JDK8에서는 기본적으로 지원하지 않는 프로토콜 이다.)

> 스프링 부트는 `h2c`(HTTP/2 cleartext version)를 지원하지 않는다. 따라서 먼저 SSL을 설정해야 한다.

### HTTP/2 with Undertow
Undertow 1.4.0 이상에서는 JDK8에 대한 추가 요구사항 없이 HTTP/2가 지원된다.

### HTTP/2 with Jetty
Jetty 9.4.8 부터 Conscrypt 라이브러리에서 HTTP/2가 지원된다. HTTP/2를 사용하기 위해서는 다음의 종속성이 추가되어야 한다.
- `org.eclipse.jetty:jetty-alpn-conscrypt-server`
- `org.eclipse.jetty.http2:http2-server`

### HTTP/2 with Tomcat
스프링 부트는 JDK 9 이상, Tomcat 9.0.x를 사용하는 경우 HTTP/2를 지원한다. JDK 8에서 사용하고자 하는 경우 `libtcnative` 라이브러리가 추가되어야 한다.

### HTTP/2 with Reactor Netty
`spring-boot-webflux-starter`는 기본적으로 Reactor Netty를 서버로 사용한다. JDK 9 이상에서 HTTP/2로 설정이 가능하다. JDK 8에 대해서도 런타임에서의 최적화된 성능을 위해 기본적으로 제공하는 라이브러리와 함께 HTTP/2
로 사용 가능하다. 이를 위해서는 `io.netty:netty-tcnative-boringssl-static` 라이브러리가 필요하다.

## Configure the Web Server
일반적으로 `application.properties`를 이용해 웹 서버 관련된 설정을 할 수 있다. 그러나 별도의 설정 관련 property를 제공해지 않는다면 `WebServerFactoryCustomizer`를 사용해야 한다.

| Server | Servlet stack | Reactive stack |
| ------ | ------------- | -------------- |
| Tomcat | TomcatServletWebServerFactory | TomcatReactiveWebServerFactory |
| Jetty | JettyServletWebServerFactory | JettyReactiveWebServerFactory |
| Undertow | UndertowServletWebServerFactory | UndertowReactiveWebServerFactory |
| Reactor | N/A | NettyReactiveWebServerFactory |

`WebServerFactory`를 통해서 connector, server resource와 같은 것들을 설정할 수 있다.

## Add a Servlet, Filter, or Listener to an Application
Servlet 스택의 애플리케이션인 경우 `Servlet`, `Filter`, `ServletContextListener`를 추가하는 두 가지 방법이 있다.

### Add a Servlet, Filter, or Listener by Using a Spring Bean
`@Bean`을 사용해서 `Servlet`, `Filter`, 또는 `Servlet *Listener`를 추가 할 수 있다. 그러나 애플리케이션 초기화 단계에 Bean이 생성되기 컨테이너에 추가되기 때문에 주의해야 한다. (초기화 단계에서 너무 많은 Bean을 생성할 수 있기 때문이다.) 초기화 단계가 아닌 Bean의 초기화를 lazy하게 변경해서 이러한 문제를 해결할 수 있다.

### Add Servlets, Filters, and Listeners by Using Classpath Scanning
`@WebServlet`, `@WebFilter` 그리고 `@WebListener` 애노테이션이 있는 클래스는 embeded servlet container 의해서 자동으로 등록된다. (embeded servlet container는 `@ServletComponentScan`과 `@Configuration` 애노테이션을 달고 있다. `@ServletComponnentScan`은 애노테이션이 있는 클래스의 패키지에서 스캔한다.)

## Configure Access Logging
Tomcat, Undertow 그리고 Jetty에 대한 Access Log를 설정할 수 있다.

```yaml
server.tomcat.basedir=my-tomcat
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```

> 로그의 기본 위치는 Tomcat의 기본 디렉토리와 관련된 `logs` 디렉토리이다. `logs`는 임시 디렉토리이므로 Tomcat의 기본 디렉토리를 수정하거나 로그의 절대 경로를 설정할 수 있다.

## Enable Tomcat’s MBean Registry
내장 Tomcat의 MBean registry는 기본적으로 disabled되어 있다. Tomcat의 MBean을 사용해서 Micrometer를 통해 metric을 노출하는 데 사용할 수 있도록 하려면 `server.tomcat.mbeanregistry.enabled` 속성을 사용해야 한다.

```yaml
server.tomcat.mbeanregistry.enabled=true
```
