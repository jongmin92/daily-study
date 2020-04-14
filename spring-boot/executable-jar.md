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
중첩된 jar 로딩을 지원하는데 사용되는 핵심 클래스는 `org.springframework.boot.loader.jar.JarFile`이다. 처음 로드할 때, 다음과 같이 각 `JarEntry`의 위치는 외부(outer) jar의 물리적인 file offset과 맵핑된다.

```
myapp.jar
+-------------------+-------------------------+
| /BOOT-INF/classes | /BOOT-INF/lib/mylib.jar |
|+-----------------+||+-----------+----------+|
||     A.class      |||  B.class  |  C.class ||
|+-----------------+||+-----------+----------+|
+-------------------+-------------------------+
 ^                    ^           ^
 0063                 3452        3980
```

위의 예제에서 `0063`의 위치는 myapp.jar에서 `/BOOT-INF/classes`에서 `A.class`를 찾는 방법을 보여준다. 중첩된 jar(mylib.jar)의 `B.class`는 `3452`에 `C.class`는 `3980`에 위치한다.

위의 정보를 바탕으로 외부 jar(myapp.jar)의 특정 부분을 탐색해서 중첩된 jar를 로드할 수 있다. 아카이브의 압축을 풀 필요가 없고, 모든 데이터를 읽어 메모리로 가져올 필요도 없다.

### Compatibility with the Standard Java “JarFile”
스프링 부트 Loader는 기존 코드 및 라이브러리와의 호환성을 유지한다. `org.springframework.boot.loader.jar.JarFile`은 `java.util.jar.JarFile`을 확장하고 있으며 대체 가능하다. `getURL()` 메서드는 `java.net.JarURLConnection`과 호환되는 connection을 제공하며 Java의 `URLClassLoader`와 함께 사용할 수 있는 URL을 리턴한다.

## Launching Executable Jars
`org.springframework.boot.loader.Launcher` 클래스는 실행 가능한 jar의 main entry point로 사용되는 특별한 bootstrap 클래스이다. jar 파일의 실제 Main 클래스이며 적절한 `URLClassLoader`를 설정하고 궁극적으로 `main()` 메서드를 호출하는데 사용된다.

3가지의 launcher sub 클래스가 있다. (JarLauncher, WarLauncher, PropertiesLauncher)

## Launcher Manifest
`META-INF/MANIFEST.MF`의 `Main-Class` 속성으로 적절한 `Launcher`를 지정해야 한다. main 메서드를 포함하고 있는 클래스는 `Start-Class` 속성으로 지정되어야 한다.

다음은 실행 가능한 jar 파일에 대한 일반적인 `MANIFEST.MF`의 예제이다.

```
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.mycompany.project.MyApplication
```

war 파일의 경우는 다음과 같다.

```
Main-Class: org.springframework.boot.loader.WarLauncher
Start-Class: com.mycompany.project.MyApplication
```

> manifest 파일에서 `Class-Path`를 지정할 필요는 없다. classpath는 중첩된 jar에서 추론된다.

## Executable Jar Restrictions
스프링 부트 Loader 패키지 애플리케이션으로 작업할 때 다음 제한 사항을 고려해야한다.
- System classLoader: 시작된 애플리케이션은 클래스를 로드할 때 `Thread.getContextClassLoader()`를 사용해야 한다. (대부분의 라이브러리 및 프레임워크에 해당한다.) `ClassLoader.getSystemClassLoader()`를 사용해서 중첩된 jar 클래스를 로드하려고하면 실패한다. `java.util.Logging`은 항상 system classloader를 사용한다. 이런 이유로 다른 로깅 구현을 고려해야한다.
