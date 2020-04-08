# Externalized Configuration

스프링 부트는 같은 애플리케이션 코드에 대해서 환경(environments) 설정을 외부로 분리할 수 있도록 한다. properties 파일, YAML 파일, environment 인자, command-line 인자를 사용해서 설정 관련된 부분을 외부로 분리한다. `@Value` 애노테이션을 사용하여 특정 값을 Bean에 직접 주입하거나 Spring의 Environment 추상화를 통해 사용하거나, `@ConfigurationProperties`를 통해 구조화된 오브젝트에 바인딩 할 수 있다.

스프링부트는 값을 재정의 할 수 있도록 특정한 `PropertySource` 순서를 사용한다.

1. devtools가 활성화 된 경우, `$HOME/.config/spring-boot` [폴더의 설정](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools-globalsettings)
2. 테스트에 있는 `@TestPropertySource`
3. `@SpringBootTest` 및 특정 slice test의 `properties`. 
4. Command line arguments
5. `SPRING_APPLICATION_JSON`(environment 변수 혹은 system property)로 부터의 property
6. `ServletConfig` init parameters
7. `ServletContext` init parameters
8. JNDI attributes from `java:comp/env`
9. Java System properties (`System.getProperties()`)
10. OS environment variables
11. `random.*` 속성만 갖는 `RandomValuePropertySource`
12. **패키징된 jar 바깥에 있는 `application-{profile}.properties` 혹은 YAML**
13. **jar와 함께 패키징 된 `application-{profile}.properties` 혹은 YAML**
14. **패키징된 jar 바깥에 있는 `application.properties` 혹은 YAML**
15. **jar와 함께 패키징 된 `application.properties` 혹은 YAML**
16. `@Configuration` 클래스의 `@PropertySource` 애노테이션
17. `SpringApplication.setDefaultProperties`로 설정된 default properties

## Configuring Random Values
`RandomValuePropertySource`는 임의의 값을 주입하는 데 유용하다.
```yaml
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```
`random.int*` 구문에서 `(`는 open, `[`는 close 이다.

## Accessing Command Line Properties
`SpringApplication`은 모든 command line option 인자에 대해서 `property`로 변환하여 스프링 `Enviroment`에 추가한다. (ex. `--server.port=9000`)

## Application Property Files
`SpringApplication`은 다음 위치에 있는 `application.properties` 파일에서 property 들을 읽어 스프링 `Enviroment`에 추가한다.
1. 현재 디렉토리의 `/config` 서브 디렉토리
2. 현재 디렉토리
3. 클래스 패스 `/config` 패키지
4. 루트 클래스 패스

> .properties 파일 대신 .yml 파일을 사용할 수도 있다.

## Profile-specific Properties
`application.properties` 파일 외에도 `application-{profile}.properties` 라는 이름 규칙을 사용하여 profile 기반의 properties를 정의할 수도 있다. `Environment`는 active profile이 설정되지 않은 경우 기본적으로 `default` profile을 가진다. 즉, `application-default.properties`가 사용된다. profile 관련 설정 파일은 항상 `application.properties`를 재정의(overriding)한다.

## Placeholders in Properties
`application.properties`의 값은 존재하는 `Environment`에 의해 필토링되므로 이전에 정의된 값을 다시 참조할 수 있다.
```yaml
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

## Using YAML Instead of Properties
[YAML](https://yaml.org/)은 인간 친화적인 데이터 직렬화에 대한 표준이다. `SpringApplication`은 클래스 경로에 [SnakeYAML](https://bitbucket.org/asomov/snakeyaml/src/master/) 라이브러리가 있는 경우 properties
의 대안으로 YAML을 자동으로 지원한다.

> "Starter"를 사용하면 SnakeYAML은 `spring-boot-starter`에 의해 자동으로 제공된다.

### Loading YAML
스프링 프레임워크는 YAML 문서를 로드하는 데 두가지 편리한 클래스를 제공한다. `YamlPropertiesFactoryBean`은 YAML을 `Properties`로 로드하고, `YamlMapFactoryBean`은 YAML을 `Map`으로 로드한다.

```yaml
environments:
    dev:
        url: https://dev.example.com
        name: Developer Setup
    prod:
        url: https://another.example.com
        name: My Cool App

my:
   servers:
       - dev.example.com
       - another.example.com
```

위의 yaml 파일은 다음의 properties 파일과 같다.

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App

my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

### Exposing YAML as Properties in the Spring Environment
`YamlPropertySourceLoader` 클래스는 스프링 `Environment`에서 `YAML`을 `PropertySource`로 노출시키는 데 사용할 수 있다. 이렇게하면 `@Value` 애노테이션을 사용해서 YAML properties에 접근할 수 있다.

### Multi-profile YAML Documents
`spring.profiles` 키를 사용해서 하나의 파일에 여러 profile 관련 설정할 수 있다.

```yaml
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production & eu-central
server:
    address: 192.168.1.120
```

> `spring.profiles`는 간단한 profile 이름 또는 profile 표현식을 포함할 수 있다. profile 표현식을 사용하면 `production & (edu-central | eu-west)`와 같은 복잡한 논리식을 사용할 수 있다. 더 자세한 내용은 [reference guide](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)를 참고한다.

### YAML Shortcomings
`@PropertySource` 애노테이션 사용해서 YAML 파일을 로드할 수 없다. 이 경우에는 properties 파일을 사용해야 한다.

profile 별 YAML 파일에서 다중 YAML 문서 구문을 사용하면 예상하지 못한 동작이 발생할 수 있다.

**application-dev.yml**
```yaml
server:
  port: 8000
---
spring:
  profiles: "!test"
  security:
    user:
      password: "secret"

```

`--spring.profile.active=dev` 인자를 주면서 애플리케이션을 실행하면 `security.user.password`가 "secret"으로 설정될 것이라 예상할 수 있지만, 실제 그렇게 동작하지 않는다.

기본 파일의 이름이 `application-dev.yml` 이므로 이미 profile에 특정한 것으로 간주되어 중첩된 문서가 필터링되어 무시된다.

> profile 별 YAML 파일과 다중 YAML 문서 구문을 함께 사용하는 것은 좋지 않다. 

## Type-safe Configuration Properties
`@Value("${property}")` 애노테이션을 사용하여 configuration properties를 주입하는 것은 때때로는 번거로울 수 있다. 특히 여러 properties를 사용하거나 데이터가 계층적인 구조를 갖고 있는 경우 더욱 그렇다. 스프링 부트는 더욱 효율적으로 configuration
 properties를 관리할 수 있는 방법을 제공한다.

### JavaBean properties binding
다음과 같이 표준 JavaBean properties를 선언하는 Bean을 바인딩 할 수 있다.

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }

    }
}
```

표준 JavaBean을 통해 바인딩하기 때문에 default empty constructor, setter, getter는 일반적으로 필수이다. 그러나 다음과 같은 경우 setter를 생략할 수 있다.
- Map이 초기화 되는 한, getter는 필요하지만, setter는 반드시 필요하지 않다.
- Collection과 Array는 index(YAML) 혹은 single comma(properties)로 구분 된 값을 통해 접근할 수 있다. 후자의 경우 setter가 필수 이다.
- 중첩된(nested) POJO가 초기화되면 setter가 필요하지 않다.
- 표준 JavaBean properties만 고려되며, static properties에 대한 바인딩은 지원하지 않는다.

### Constructor binding
이전의 예제는 다음과 같이 변경 불가능한 방식(immutable)으로 다시 작성할 수 있다.
```java
package com.example;

import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() { ... }

    public InetAddress getRemoteAddress() { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password,
                @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() { ... }

        public String getPassword() { ... }

        public List<String> getRoles() { ... }

    }

}
```

이 설정에서 `@ConstructorBinding` 애노테이션은 생성자 바인딩을 사용해야 함을 나타내는 데 사용된다. 바인더는 바인딩하려는 매개 변수가 있는 생성자를 찾을 것이다.

`@ConstructorBinding` 클래스의 중첩 멤버도 생성자를 통해 바인딩된다. `@DefaultValue`를 사용해서 기본 값을 지정할 수 있다.

> 생성자 바인딩을 사용하려면 `@EnableConfigurationProperties` 또는 configuration property scanning을 사용해야 한다. 일반 스프링 메커니즘에 의해 생성된 Bean(@Component, @Bean method, @Import)에는 생성자 바인딩을 사용할 수 없다.
>
> 클래스에 대한 생성자가 둘 이상인 경우 바인딩해야하는 생성자에 `@ConstructorBinding`을 직접 사용할 수 있다.

### Enabling `@ConfigurationProperties`-annotated types
스프링 부트는 `@ConfigurationProperties` 타입을 바인딩하고 Bean으로 등록하기위한 수단을 제공한다. 클래스별로 configuration properties를 활성화 하거나 component scanning과 비슷한 방식으로 scanning 방식을 활성화 할 수 있다.

경우에 따라 `@ConfigurationProperties` 애노테이션이 달린 클래스는 auto-configuration 혹은 conditional로 제공하려는 경우 scanning 방식이 적합하지 않을 수 있다. 이 경우, `EnableConfigurationProperties` 애노테이션을 사용해서 처리 할 타입을 지정할 수 있다.

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

configuration property scanning 방식을 사용하려면 `@ConfigurationPropertiesScan` 애노테이션을 애플리케이션에 추가한다. 일반적으로 `@SpringBootApplication` 애노테이션이 달린 main application
 클래스에 추가하지만, `@Configuration` 클래스에도 추가 할 수 있다. 기본적으로 애노테이션을 선언하는 클래스 패키지에서 scan이 일어나지만 특정 패키지를 지정할 수도 있다. 

```java
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
public class MyApplication {
}
```

### Relaxed Binding
스프링 부트는 `Environment` properties를 `@ConfigurationProperties` bean에 바인딩하기 위해 다소 완화된 규칙을 사용한다. 따라서 `Environment` property 이름과 bean property 이름이 정확히 일치할 필요는 없다.

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
}
```

위의 코드를 사용하면 다음의 property 이름을 모두 사용할 수 있다.

| Property | Note |
| -------- | ---- |
| acme.my-project.person.first-name | `.properties`와 `.yml` 파일에 권장되는 Kebab case 형식 |
| acme.myProject.person.firstName | 표준 camel case 형식 |
| acme.my_project.person.first_name | underscore case 형식 |
| ACME_MYPROJECT_PERSON_FIRSTNAME | 시스템 환경 변수를 사용할 때 권장되는 Upper case 형식 |

**property source 마다의 relaxed 바인딩 규칙**은 다음과 같다.

| Property Source | Simple | List |
| --------------- | ------ | ---- |
| Properties Files | Kebab case, Camel case, underscore case | `[]` 또는 `,`로 구분 된 값을 사용 |
| YAML Files | Kebab case, Camel case, underscore case | 표준 YAML list 문법 또는 `,`로 구분 된 값을 사용 |
| Environment Variables | underscoe를 구분자로 사용하는 Upper case | MY_ACME_1_OTHER = my.acme\[1].other와 같이 underscore로 둘러싸인 숫자 값 |
| System properties | Kebab case, Camel case, underscore case | `[]` 또는 `,`로 구분 된 값을 사용 |

> 공식 문서에서는 가능한 경우 lower-case kebab 형식을 권장한다. (ex. my.property-name=acme))

`Map` property에 바인딩 할 때는 키에 lowercase alpha-numeric 문자 또는 `-`의 경우, 원래 값이 유지되도록 대괄호 표기법을 사용해야 한다.

```yaml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

위의 property는 `/key1`, `/key2`, `/key3`를 `Map`의 키로 사용해서 바인딩된다.

> YAML 파일의 경우 키를 올바르게 구문 분석하려면 괄호를 `"`로 묶어야 한다.

### Converting durations
