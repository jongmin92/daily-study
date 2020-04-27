# Build Cache

**Gradle build cache는 다른 build에서 생성된 output을 재사용해서 시간을 절약하는 것을 목표로 하는 캐시 메커니즘이다.** build cache는 build 출력을 저장하고 input이 변경되지 않은 것으로 판별될 때, 캐시에서 이러한 출력을 fetch해서 다시 빌드하는 것을 피함으로써 동작한다.

build cache를 사용하는 첫 번째 기능은 태스크(task)의 output을 캐싱하는 것이다. 기본적으로 태스크 output 캐싱은 [up-to-date 검사](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:up_to_date_checks) 와 같은 방식(intelligence)를 사용한다.

태스크 이외에 [아티팩트 변환](https://docs.gradle.org/current/userguide/artifact_transforms.html#sec:abm_artifact_transforms) 도 build cache를 활용하고 태스크 output 캐싱과 유사하게 output을 재사용할 수 있다.

## Enable the Build Cache

기본적으로 build cache는 사용되지 않는다. build cache가 활성화되면 Gradle 사용자 홈에 빌드 출력이 저장된다.

다음의 두 가지 방법으로 build cache를 사용할 수 있다.

### Run with --build-cache on the command-line

Gradle은 이 build에만 build cache를 사용한다.

### Put org.gradle.caching=true in your gradle.properties

Gradle은 --no-build-cache로 명시적으로 비활성화 하지 않는 한 모든 빌드에 대해 이전 빌드의 출력을 재사용하려고 시도한다.

## Task Output Caching

up-to-date 검사에서 설명된 incremental build 외에도 Gradle은 이전 태스크 실행의 input이 일치하는 경우 output을 재사용해서 시간을 절약할 수 있다.

빌드가 태스크 output 캐싱에서 제대로 작동하려면 incremental build와 잘 작동해야한다. 예를 들어, 빌드를 연속으로 두 번 실행할 때 output이 있는 태스크는 최산 상태(up-to-date)여야 한다. 이 전제 조건이 충족되지 않는 경우 태스크 output 캐싱을 사용할 때 더 빠른 빌드 또는 올바른 빌드릴 기대할 수 없다.

build cache를 활성화하면 태스크 output 캐싱이 자동으로 활성화된다.

### What does it look like

몇 개의 Java 소스 코드를 갖고 있는 프로젝트를 Java 플러그인을 사용해서 빌드한다.

```bash
> gradle --build-cache compileJava
:compileJava
:processResources
:classes
:jar
:assemble

BUILD SUCCESSFUL
```

clean 하고 다시 빌드를 한다.

```bash
> gradle clean
:clean

BUILD SUCCESSFUL
```

```bash
> gradle --build-cache assemble
:compileJava FROM-CACHE
:processResources
:classes
:jar
:assemble

BUILD SUCCESSFUL
```

compileJava 태스크는 실행되지 않기 캐시된 output이 사용되었음을 알 수 있다. 다른 태스크는 캐싱할 수 없기 때문에 build cache에서 사용되지 않았다.

## Cacheable tasks

태스크는 input과 output을 갖고 있으므로 Gradle은 input을 기반으로 output을 고유하게 정의하는 build cache key를 가질 수 있다. 이 build cache key는 build cache에서 이전 output을 요청하거나 build cache에 새로운 output을 저장하는데 사용된다.

###Built-in cacheable tasks

다음과 같은 build-in Gradle 태스크들은 캐싱될 수 있다.
- Java toolchain: JavaCompile, Javadoc
- Groovy toolchain: GroovyCompile, Groovydoc
- Scala toolchain: ScalaCompile, PlatformScalaCompile, ScalaDoc
- Native toolchain: CppCompile, CCompile, SwiftCompile
- Testing: Test
- Code quality tasks: Checkstyle, CodeNarc, Pmd
- JaCoCo: JacocoMerge, JacocoReport
- Other tasks: AntlrTask, ValidatePlugins, WriteProperties

다른 build-in 태스크는 현재 캐싱할 수 없다.
