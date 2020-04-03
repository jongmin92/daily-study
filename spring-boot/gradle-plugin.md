# Spring Boot Gradle Plugin

> document: https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/

스프링 부트 그레이들 플러그인은 그레이들에서 스프링 부트 지원을 제공한다. executable jar 혹은 war를 패키징하고 스프링 부트 애플리케이션을 실행하며 의존성 관리(dependency management)를 제공한다. 그레이들 5.x 이상의 버전이 필요하다.

## Getting started
```kotlin
plugins {
    id("org.springframework.boot") version "2.2.6.RELEASE"
    java
}
```
플러그인은 다른 특정 플러그인이 추가될 때 상호작용할 수 있다. (ex. 자바 플러그인이 추가되면 executable jar 빌드 태스크가 자동으로 추가된다.)

## Managing dependencies
```kotlin
plugins {
    id("org.springframework.boot") version "2.2.6.RELEASE"
    id("io.spring.dependency-management") version "1.0.9.RELEASE"
    java
}
```

`io.spring.dependency-management` 플러그인을 적용하면 스프링 부트 플러그인은 자동으로 사용중인 스프링 부트 버전에서 [spring-boot-dependencies bom](https://github.com/spring-projects/spring-boot/blob/v2.2.6.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) 을 import 한다.
그러므로 bom에서 관리되는 종속성을 선언 할 때 버전을 생략할 수 있다.
> dependency-management 플러그인은 BOM(Bill Of Materials) 파일을 제공한다. BOM은 의존성 버전 제어를 위한 POM이다.

의존성 관리(dependency-management)는 어떤 아티팩트의 버전을 사용할지만 기술하고, 실제 의존성이 추가되는 것은 dependencies에 명시적으로 추가했을 때다.

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
```

### Customizing managed versions
```kotlin
extra["slf4j.version"] = "1.7.20"
```
의존성 관리에 의해 관리되는 버전이 아닌 특정 버전을 설정할 수도 있다.

### Using Spring Boot’s dependency management in isolation
스프링 부트 플러그인을 적용하지 않고도 스프링 부트의 의존성 관리를 사용할 수 있다. `SpringBootPlugin` 클래스는 그룹 ID, 아티팩트 ID 혹은 버전을 몰라도 bom을 가져 오는 데 사용할 수 있는 `BOM_COORDINATES` 상수를 제공한다.

먼저 스프링 부트 플러그인이 적용되지 않도록 변경한다.

```kotlin
plugins {
    id("org.springframework.boot") version "2.2.6.RELEASE" apply false
}
```

스프링 부트 의존성 관리 플러그인을 적용하고 스프링 부트의 bom을 가져오도록 설정한다.

```kotlin
the<DependencyManagementExtension>().apply {
    imports {
        mavenBom(org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES)
    }
}
```

혹은 다음과 같이 작성할 수도 있다.

```kotlin
dependencyManagement {
    imports {
        mavenBom(org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES)
    }
}
``` 

## Packaging executable archives
플러그인은 애플리케이션의 모든 종속성을 포함하는 실행 가능한 아카이브(jar, war)를 생성한 후 `java -jar`로 실행할 수 있다.

### Packaging executable jars
`bootJar` 태스크를 이용해서 실행 가능한 jar를 빌드할 수 있다. bootJar 태스크는 Java 플러그인이 적용될 때 자동으로 생성되며 `BootJar`의 인스턴스이다. `assemble` 태스크는 `bootJar` 태스크에 의존성을 가지므로 `assemble` 태스크를 실행하면 `bootJar
` 태스크도 함께 실행된다. (`build` 태스크는 `assemble` 태스크에 의존성을 가진다.)

### Packaging executable wars
`bootWar` 태스크를 이용해서 실행 가능한 war를 빌드할 수 있다. bootWar 태스크는 war 플러그인이 적용될 때 자동으로 생성되며 `BootWar`의 인스턴스이다.

#### Packaging executable and deployable wars
`java -jar`로 실행하고 외부 컨테이너에 배포 할 수 있도록 war 파일을 패키징할 수 있다. 이를 위해서는 임베디드 서블릿 컨테이너 종속성이 `providedRuntime` 설정으로 추가되어야 한다.

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
}
```

이렇게 하면 외부 컨테이너의 자체 클래스와 충돌하지 않는 war 파일의 `WEB-INF/lib-provided` 에 패키징된다.

>`providedRuntime` 설정은 `compileOnly` 설정보다 선호된다. `compileOnly` 종속성은 테스트 클래스 패스에 없으므로 웹 기반의 integration 테스트가 실패한다.
>`providedCompile`은 프로젝트가 컴파일 될 때 클래스 패스에서 확인 가능하다. (`compile`을 확장(extends) 하고 있다.) `providedRuntime`은 `runtime`을 확장하고 있으며, 컴파일 되는 클래스 패스가 아니다.

### Packaging executable and normal archives
기본적으로 `bootJar` 혹은 `bootWar` 태스크가 설정되면 `jar` 또는 `war` 태스크는 disabled 된다. jar 또는 war 태스크를 활성화해서 실행 가능한 아카이브와 일반 아카이브를 동시에 빌드하도록 프로젝트를 설정할 수 있다.

```kotlin
tasks.getByName<Jar>("jar") {
    enabled = true
}
```

### Configuring executable archive packaging
`BootJar`와 `BootWar` 태스크는 각각 `Jar`와 `War` 태스크의 하위 클래스이다. 따라서 `BootJar`의 경우 `Jar`와 같은 설정 뿐만 아니라 별도의 설정도 갖고 있다.

#### Configuring the main class
기본적으로 실행 가능한 아카이브의 main 클래스는 클래스 패스 내에서 `public static void main(String[])` 메서드가 있는 클래스를 찾아 자동으로 설정된다.

물론 main 클래스를 명시적으로 설정할 수도 있다.

```kotlin
tasks.getByName<BootJar>("bootJar") {
    mainClassName = "com.example.ExampleApplication"
}
```

또는, 스프링 부트 DSL의 `mainClassName` 속성을 사용해서 설정할 수도 있다.

```kotlin
springBoot {
    mainClassName = "com.example.ExampleApplication"
}
```

#### Making an archive fully executable
스프링 부트는 완전히 실행 가능한 아카이브를 지원한다. 애플리케이션을 시작하는 방법을 알고 있는 쉘 스크립트를 추가함으로써 아카이브를 완전히 실행가능하게 만든다. 이 기능을 사용하기 위해서는 launch script가 활성화 되어야 한다.

```kotlin
tasks.getByName<BootJar>("bootJar") {
    launchScript()
}
```

스프링 부트의 기본 시작 스크립트(launch script)가 추가된다. 기본 실행 스크립트에는 기본 값을 가진 여러 속성이 포함되어 있다. `properties` 속성을 사용해서 값을 정의할 수 있다.

```kotlin
tasks.getByName<BootJar>("bootJar") {
    launchScript {
        properties(mapOf("logFilename" to "example-app.log"))
    }
}
```

직접 작성한 launch script를 설정할 수도 있다.

```kotlin
tasks.getByName<BootJar>("bootJar") {
    launchScript {
        script = file("src/custom.script")
    }
}
```

## Publishing your application

### Publishing with the `maven` plugin
`maven` 플러그인이 적용되면, `uploadBootArchives` 이름을 가진 bootArchives 업로드 태스크가 자동으로 생성된다. 기본적으로 bootArchives에는 bootJar 또는 bootWar 태스크로 생성된 아카이브가 포함된다. `uploadBootArchives` 태스크는 이 아카이브를 Maven 저장소에 publish 하도록 설정될 수 있다.

```kotlin
tasks.getByName<Upload>("uploadBootArchives") {
    repositories.withGroovyBuilder {
        "mavenDeployer" {
            "repository"("url" to "https://repo.example.com")
        }
    }
}
```

### Publishing with the `maven-publish` plugin
스프링 부트 jar 또는 war를 publish 하려면 `MavenPublication`의 `artifact` 메서드를 사용한다. 

```kotlin
publishing {
    publications {
        create<MavenPublication>("bootJava") {
            artifact(tasks.getByName("bootJar"))
        }
    }
    repositories {
        maven {
            url = uri("https://repo.example.com")
        }
    }
}
```

## Running your application with Gradle
아카이브를 빌드하지 않고 `bootRun` 태스크로 애플리케이션을 실행할 수 있다.

```bash
$ ./gradlew bootRun
```

`bootRun` 태스크는 `JavaExec` 서브 클래스인 `BootRun`의 인스턴스이다. 따라서 그레이들에서 Java 프로세스를 실행하기 위한 [일반적인 설정](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.JavaExec.html)을 사용할 수 있다.

기본적으로 main 클래스는 클래스 패스에서 `public static void main(String[])` 메서드가 있는 클래스를 찾아 설정된다.

main 클래스를 명시적으로 설정할 수도 있다.

```kotlin
tasks.getByName<BootRun>("bootRun") {
    main = "com.example.ExampleApplication"
}
```

또는, 스프링 부트 DSL의 `mainClassName` 속성을 사용해서 설정할 수도 있다.

```kotlin
springBoot {
    mainClassName = "com.example.ExampleApplication"
}
```

### Passing arguments to your application
그레이들 4.9 이상을 사용하는 경우, `JavaExec` 태스크와 마찬가지로 `--args='<arguments>'`를 사용해 arguments를 bootRun으로 전달할 수 있다.

```bash
$ ./gradlew bootRun --args='--spring.profiles.active=dev'
```

## Integrating with Actuator

### Generating build information
스프링 부트 액츄에이터의 `info` endpoint는 `META-INF/build.info.properties` 파일이 있는 경우 빌드에 대한 정보를 자동으로 제공한다. 이 파일 생성을 위한 `BuildInfo` 태스크가 있다.

```
springBoot {
    buildInfo()
}
```

`bootBuildInfo` 라는 `BuildInfo` 태스크가 구성되고 Java 플러그인의 `classes` 태스크가 의존하게 된다. output directory는 main source set directory인 `build/resources/main/META-INF`가 된다.

기본적으로 생성된 빌드 정보는 프로젝트로부터 전달된다.

| Property | Default value |
| -------- | ------------- |
| build.artifact | bootJar 혹은 bootWar 태스크의 기본 이름 |
| build.group | 프로젝트 그룹 |
| build.name | 프로젝트 이름 |
| build.version | 프로젝트 버전 |
| build.time | 프로젝트가 빌드되는 시간 |

위의 정보는 DSL을 이용해서도 설정이 가능하다.

```kotlin
springBoot {
    buildInfo {
        properties {
            artifact = "example-app"
            version = "1.2.3"
            group = "com.example"
            name = "Example application"
        }
    }
}
```

`build.time`의 기본값은 프로젝트가 빌드되는 순간이다. 그러나 side-effect로 태스크가 `up-to-date`가 되지 않는다는 것이다. 결과적으로 프로젝트 테스트를 포함해서 더 많은 작업을 실행해야하므로 빌드 시간이 더 오래 걸린다. 또 다른 side-effect
로 작업의 출력이 항상 변경된다는 것이다. 그러므로 build.time의 값 보다 빌드 성능 혹은 반복이 중요하다면 build.time의 값을 null 혹은 특정 값으로 고정되도록 설정하는 것이 좋다.

빌드 정보에 특정 프로퍼티를 추가할 수도 있다.

```kotlin
springBoot {
    buildInfo {
        properties {
            additional = mapOf(
                "a" to "alpha",
                "b" to "bravo"
            )
        }
    }
}
```

## Reacting to other plugins
다른 플러그인이 적용되면 스프링 부트 플러그인은 프로젝트의 설정을 다양하게 변경한다.

### Reacting to the Java plugin
그레이들의 [Java 플러그인](https://docs.gradle.org/current/userguide/java_plugin.html)이 적용되면 스프링 부트 플러그인의 변경은 다음과 같다.

1. 실행 가능한 fat jar를 생성하는 `bootJar`라는 `BootJar` 태스크를 생성한다. jar는 main source set의 런타임 클래스 패스에 있는 모든것을 포함한다. main source set의 클래스는 `BOOT-INF/classes`로 패키지되고, jar는 `BOOT-INF/lib`로 패키지된다.
2. `assemble` 태스크가 `bootJar` 태스크에 의존성을 갖도록 설정된다.
3. `jar` 태스크를 disable 한다.
4. 애플리케이션을 실행하는데 사용할 수 있는 `bootRun`이라는 `BootRun` 태스크를 생성한다.
5. `bootJar` 태스크에 의해서 생성된 아티팩트를 포함하는 `bootArchives` 설정을 생성한다.
6. `UTF-8`을 사용하도록 `JavaCompile` 태스크를 설정한다.
7. `-parameters` 컴파일러 인자(argument)를 사용하도록 태스크를 설정한다.

### Reacting to the Kotlin plugin
그레이들의 [Kotlin 플러그인](https://kotlinlang.org/docs/reference/using-gradle.html)이 적용되면 스프링 부트 플러그인의 변경은 다음과 같다.

1. 스프링 부트 의존성 관리에 사용된 Kotlin 버전을 플러그인의 버전과 일치시킨다. `kotlin.version` 프로퍼티의 값을 Kotlin plugin 버전과 일치하는 값으로 설정하면 된다. 
2. `-java-parameters` 컴파일러 인자(argument)를 사용하도록 `KotlinCompile` 태스크를 설정한다.

### Reacting to the war plugin
그레이들의 [War 플러그인](https://docs.gradle.org/current/userguide/war_plugin.html)이 적용되면 스프링 부트 플러그인의 변경은 다음과 같다.

1. 실행가능한 fat war를 생성하는 `BootWar` 태스크를 생성한다. `providedRuntime` 설정의 디펜던시는 `WEB-INF/lib-provided`로 패키징된다.
2. `assemble` 태스크가 `bootWar` 태스크에 의존하도록 설정한다.
3. `war` 태스크를 disable 한다.
4. `bootWar` 태스크에서 생성된 아티팩트를 포함하도록 `bootArchives` 설정을 구성한다.

### Reacting to the dependency management plugin
[io.spring.dependency-management 플러그인](https://github.com/spring-gradle-plugins/dependency-management-plugin)이 적용되면, 스프링 부트 플러그인은 자동으로 `spring-boot-dependencies` bom을 import 한다.
