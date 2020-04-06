# Creating Your Own Auto-configuration
공유 라이브러리를 개발하는 회사에서 근무하거나 오픈 소스 또는 상업용 라이브러리를 만드는 경우 auto-configuration을 이용할 수 있다.

## Understanding Auto-configured Beans
기본적으로 auto-configuration은 `@Configuration` 클래스로 구현된다. 추가적으로 `@Confiditonal` 애노테이션은 auto-configuration이 적용되는 시기를 제한하는데 사용된다. 일반적으로 auto-configuration 클래스는 `@ConditionalOnClass` 혹은 `@ConditionalOnMissingBean` 애노테이션을 사용한다. 

## Locating Auto-configuration Candidates
스프링 부트는 jar 내에 `META-INF/spring.factories` 파일이 있는지 확인한다. 파일은 다음과 같이 `EnableAutoConfiguration` key 아래에 configuration class를 나열해야 한다.
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

> Auto-configuration은 위와 같은 방법으로만 로드되어야 한다. component scanning의 대상이 되지 않는 특정 패키지에 정의되어야 한다. 또한 auto-configuration 클래스는 추가적인 component를 찾기 위해 component scanning을 enable 할 수 없다. 대신 `@Import`를 사용해야 한다.

configuration을 특정 순서로 적용해야 하는 경우 `@AutoConfigureAfter` 혹은 `@AutoConfigureBefore` 애노테이션을 사용할 수 있다.

서로 직접적인 관련이 없는 auto-configuration에 대해서 특정 순서를 만들고 싶다면 `@AutoConfigureOrder`를 사용할 수 있다. 이 애노테이션은 auto-configuration을 위한 `@Order` 애노테이션이라고 생각하면 된다.

## Condition Annotations
스프링 부트는 여러 `@Conditional` 애노테이션을 제공한다.

### Class Conditions
`@ConditionalOnClass`와 `@ConditionalOnMissingClass` 애노테이션을 사용하면 특정 클래스의 존재 여부에 따라 `@Configuration` 클래스를 포험할 수 있다. 해당 클래스가 실제로 실행중인 애플리케이션 클래스 패스에 없을 수도 있지만 `value
` 속성을 사용하여 참조할 수 있다. 문자열 값을 사용하여 클래스 이름을 지정하려는 경우 `name` 속성을 사용할 수도 있다.
이 메커니즘은 @Bean 메서드에는 적용되지 않는다.

### Bean Conditions
`@ConditionalOnBean`과 `@ConditionalOnMissingBean` 애노테이션은 특정 Bean의 존재 여부에 따라 Bean을 포함할 수 있다. `value` 속성으로 `type`을 지정하거나 `name` 속성으로 이름을 지정할 수 있다. `search` 속성으로는 Bean
을 검색할 때 고려해야할 `ApplicationContext` 계층 구조(hierarchy)를 제한할 수 있다.

> `@ConditionalOnBean`과 `@ConditionalOnMissingBean`은 `@Configuration` 클래스가 생성되는 것을 막지 않는다. 

### Property Conditions
`@ConditionalOnProperty` 애노테이션은 Spring Environment property 기반으로 configuration을 포함할 수 있다. `prefix`와 `name` 속성을 사용해서 체크해야하는 property를 지정할 수 있다.

### Resource Conditions
`@ConditionalOnResource` 애노테이션은 특정 리소스가 있는 경우에만 configuration을 포함할 수 있다. 리소스는 `file:/home/user/test.dat`와 같이 일반적인 스프링 규칙을 사용해서 지정할 수 있다.

### Web Application Conditions
`@ConditionalOnWebApplication` 과 `@ConditionalOnNotWebApplication` 애노테이션을 사용하면 애플리케이션이 "웹 애플리케이션"인지 여부에 따라 configuration을 포함할 수 있다. 

### SpEL Expression Conditions
`@ConditionalOnExpression` 애노테이션은 SpEL 표현식의 결과를 기반으로 configuration을 포함할 수 있다.

## Testing your Auto-configuration
`ApplicationContextRunner`는 `ApplicationContext`를 실행하고 AssertJ 스타일의 assertion을 제공하는 유틸리티 클래스이다. 테스트 클래스의 필드로 정의되어 각 테스트에서 정의되어 사용된다.

ApplicationContextRunner는 각 테스트에 대해 ApplicationContext를 사용자 정의할 수 있는 `withUserConfiguration` 메서드를 제공한다.
```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
    .withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

각 테스트는 runner를 사용해서 특정 use case를 나타낼 수 있다. `run` 메서드는 컨텍스트에 assertion을 적용하는 매개 변수로 ContextConsumer를 사용한다. 테스트가 종료되면 ApplicationContext는 자동으로 닫힌다.
```java
@Test
void defaultServiceBacksOff() {
    this.contextRunner.withUserConfiguration(UserConfiguration.class).run((context) -> {
        assertThat(context).hasSingleBean(UserService.class);
        assertThat(context).getBean("myUserService").isSameAs(context.getBean(UserService.class));
    });
}

@Configuration(proxyBeanMethods = false)
static class UserConfiguration {

    @Bean
    UserService myUserService() {
        return new UserService("mine");
    }

}
```

특정 클래스가 클래스 패스에 없는 경우를 테스트 하고 싶은 경우 `FilteredClassLoader`를 사용한다. 런타임 시점에 클래스 패스에서 특정 클래스를 필러팅하는데 사용한다.

### Overriding the Classpath
런타임에 특정 클래스 및 패키지가 없을 때 발생하는 상황을 테스트할 수도 있다. 스프링 부트든 러너가 쉽게 사용할 수 있는 `FilteredClassLoader`를 제공한다.
```java
@Test
void serviceIsIgnoredIfLibraryIsNotPresent() {
    this.contextRunner.withClassLoader(new FilteredClassLoader(UserService.class))
            .run((context) -> assertThat(context).doesNotHaveBean("userService"));
}
```

## Creating Your Own Starter
라이브러리를 위한 스프링 부트 스타터에는 다음 컴포넌트들이 포함될 수 있다.
- auto-configuration 코드를 포함하고 있는 `autoconfigure` 모듈
- `autoconfigure` 모듈 뿐만 아니라 추가적인 모듈을 포함하고 있는 `starter` 모듈
즉, starter를 추가하면 해당 라이브러리를 사용하는 데 필요한 모든 것이 제공되어야 한다.

> 두 가지 문제를 분리할 필요가 없는 경우에는 auto-configuration 코드와 dependency management(의존성 관리)를 단일 모듈로 결합해서 사용할 수 있다.

## autoconfigure Module
`autoconfigure` 모듈은 라이브러리를 시작하는 데 필요한 모든 것이 포함되어 있다. (ex. `ConfigurationProperties`)

## Starter Module
starter는 실제로 비어있는 jar이다. 오직 라이브러리의 종속성만을 제공하는 것을 목표로한다.
