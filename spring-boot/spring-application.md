# SpringApplication

## Lazy Initialization
`SpringApplication`은 application이 lazy하게 초기화 되는 것을 허용한다. **lazy 초기화가 활성화되고 애플리케이션이 시작되면 빈은 애플리케이션 시작 시점이 아닌 필요한 시점에 생성된다.** 결과적으로 lazy 초기화를 활성화하면 애플리케이션이 시작하는 데 걸리는 시간을 줄일 수 있다. (웹 애플리케이션인 경우, lazy 초기화를 사용하면 HTTP 요청이 들어오기 전까지 많은 웹 관련 빈이 초기화되지 않는다.)
 
**lazy 초기화의 단점은 애플리케이션의 문제 발견 시점을 늦출 수 있다는 것이다.** 잘못 구성된 빈이 lazy 하게 초기화되면 애플리케이션 시작 시에 문제를 발견할 수 없다. lazy 초기화를 사용하는 경우 JVM의 힙 크기도 조정하는 것이 필요하다. (애플리케이션 기동시가 아닌 그 이후에 빈이 생성되기 때문이다.)

lazy 초기화는 `SpringApplicationBuilder`의 `lazyInitialization` 메서드 또는 `SpringApplication`의 `setLazyInitialization` 메서드를 사용해서 설정할 수 있다. 또는 `spring.main.lazy-initialization` 프로퍼티 설정으로 활성화 할 수 있다.

```yaml
spring.main.lazy-initialization=true
```

## Application Events and Listeners
몇 가지 이벤트는 `ApplicationContext`가 생성되기 전에 트리거(trigger)되기 때문에 리스너를 `@Bean`으로 등록할 수 없다. `SpringApplication.addListeners()` 메서드 또는 `SpringApplicationBuilder.listeners()` 메서드로 등록할 수 있다.

애플리케이션 생성과 상관없이 리스너를 동록하려면 `org.springframework.context.ApplicationListener`를 키로 사용하여 `META-INF/spring.factories` 파일을 프로젝트에 추가해서 리스너를 사용할 수 있다.

```yml
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

애플리케이션이 실행될 때, 애플리케이션 이벤트는 다음 순서로 발생한다.
1. `ApplicationStartingEvent`는 애플리케이션 시작시 리스너 등록 및 초기화를 제외하고 별도의 로직을 처리하기 전에 발생한다. 
2. `ApplicationEnvironmentPreparedEvent`는 컨텍스트에서 사용될 Environment를 알고 있지만 컨텍스트가 생성되기 전에 발생한다.
3. `ApplicationContextInitializedEvent`는 `ApplicationContext`가 준비되고 `ApplicationContextInitializer`가 호출됐지만 빈 정의가 로드 되기전에 발생한다.
4. `ApplicationPreparedEvent`는 refresh가 발생하기 전이나 빈이 로드 된 후 발생한다.
5. `WebServerInitializedEvent`는 `WebServer`가 준비된 후 발생한다. (`ServletWebServerInitializedEvent`, `ReactiveWebServerInitializedEvent`)
6. `ApplicationStartedEvent`는 context가 refresh된 후 application과 command-line runner가 실행되기 전에 발생한다.
7. `ApplicationReadyEvent`는 application과 command-line runner가 실행된 후 발생한다. 애플리케이션이 요청을 처리할 준비가 되었음을 의미한다.
8. `ApplicationFailedEvent`는 startup 과정에서 exception이 발생하는 경우 발생한다.













