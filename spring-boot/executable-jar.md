# Executable Jar

`spring-boot-loader` 모듈은 스프링 부트가 실행 가능한(executable) jar와 war 파일을 지원한다. Maven 플러그인 또는 Gradle 플러그인을 사용하는 경우 실행 가능한 jar가 자동으로 생성되기 때문에, 실제 작동 방식에 대한 세부적인 내용은 알 필요가 없다.

다른 빌드 시스템을 이용해 실행 가능한 jar 파일을 생성해야 하거나 기반이 되는 기술에 대해 궁금한 경우에는 해당 내용이 도움이 될 것이다.

## Nested JARs
자바는 중첩된(nested)된 jar 파일을 로드(load)하는 표준 방법을 제공하지 않는다. (jar 내에 포함된 jar 파일) 이로 인해 압축을 풀지 않고 command line에서 실행할 수 있는 독립형 애플리케이션을 배포해야 하는 경우 문제가 된다.

이 문제를 풀기 위해 많은 개발자들이 "shaded" jar를 사용했었다. shaded jar는 모든 jar의 클래스들을 하나의 jar인 `"uber jar"`로 패키징한 것을 의미한다. **shaded jar
의 문제는 실제 애플리케이션에 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다는 것이다.** 또한 여러 jar에서 동일한 파일 이름을 사용하지만 내용이 다른 경우에도 문제가 될 수 있다. 스프링 부트는 다른 접근 방식을 이용해 jar를 중첩(nest)해서 사용한다.

### The Executable Jar File Structure
스프링 부트 Loader와 호환되는 jar 파일은 다음과 같은 구조로 만들어져야 한다.

```
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-BOOT-INF
    +-classes
    |  +-mycompany
    |     +-project
    |        +-YourClasses.class
    +-lib
       +-dependency1.jar
       +-dependency2.jar
```

애플리케이션 클래스는 중첩된 `BOOT-INF/classes` 디렉토리에 있어야 한다. 라이브러리는 중첩된 `BOOT-INF/lib` 디렉토리에 있어야 한다.

### The Executable War File Structure
스프링 부트 Loader와 호환되는 war 파일은 다음과 같은 구조로 만들어져야 한다.


```
example.war
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-WEB-INF
    +-classes
    |  +-com
    |     +-mycompany
    |        +-project
    |           +-YourClasses.class
    +-lib
    |  +-dependency1.jar
    |  +-dependency2.jar
    +-lib-provided
       +-servlet-api.jar
       +-dependency3.jar
```

라이브러리는 중첩된 `WEB-INF/lib` 디렉토리에 있어야 한다. embedded로 실행할 때는 필요하지만 기존 웹 컨테이너에 배포할 때는 필요하지 않은 라이브러리는 모두 `WEB-INF/lib-provided` 디렉토리에 있어야 한다.

## Spring Boot’s “JarFile” Class
