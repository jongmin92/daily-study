# Use bill of materials (BOM) As Dependency Constraints
Gradle 5부터 빌드 파일에서 BOM을 사용해 쉽게 권장되는 의존성 버전을 얻을 수 있다. BOM에 정의된 의존성 버전은 Gradle의 의존성 제한 조건이다. 즉, BOM에 정의된 의존성 제약 조건을 통해 버전이 해결(resolve)되므로 BOM에 포함된 의존성에 대해서는 빌드 파일에 버전을 명시할 필요 없다. 전이 의존성(transitive dependency) 버전 또한 BOM을 사용해서 해결된다.

BOM 파일을 import 하기 위해서 의존성 핸들러 메서드인 `platform`을 사용할 수 있다. BOM에 정의된 버전이 권장사항이지만, BOM에 정의된 버전을 빌드 파일에서 명시적(explicit) 버전으로 지정함으로써 대체할 수 있다.

# Example
- Spring Boot 2.1.4.RELEASE 용 BOM 파일을 사용한다.
- BOM에 정의된 두 가지 의존성을 추가한다.
  - `commons-codec:commons-codec`: 버전을 명시적으로 지정하지 않음않음. BOM에 정의된 버전 사용.
  - `org.yaml:snakeyaml`: 버전을 명시적으로 지정.

```kotlin
// File: build.gradle.kts
plugins {
    java
}
 
repositories {
    jcenter()
}
 
dependencies {
    // Load bill of materials (BOM) for Spring Boot.
    // The dependencies in the BOM will be
    // dependency constraints in our build.
    implementation(platform("org.springframework.boot:spring-boot-dependencies:2.1.4.RELEASE"))
 
    // Use dependency defined in BOM.
    // Version is not needed, because the version
    // defined in the BOM is a dependency constraint
    // that is used.
    implementation("commons-codec:commons-codec")
 
    // Override version for dependency in the BOM.
    // Version in BOM is 1.23.
    implementation(group = "org.yaml",
                   name = "snakeyaml",
                   version = "1.24")
}
```

`dependencies` task를 실행하면 의존성이 어떻게 해결되는지 확인할 수 있다. `snakeyaml`의 경우 BOM에 정의된 버전은 1.23이지만 1.24버전이 사용된다.

```bash
$ gradle -q dependencies --configuration compileClasspath
------------------------------------------------------------
Root project
------------------------------------------------------------
 
compileClasspath - Compile classpath for source set 'main'.
+--- org.springframework.boot:spring-boot-dependencies:2.1.4.RELEASE
|    +--- commons-codec:commons-codec:1.11 (c)
|    \--- org.yaml:snakeyaml:1.23 -> 1.24 (c)
+--- commons-codec:commons-codec -> 1.11
\--- org.yaml:snakeyaml:1.24
 
(c) - dependency constraint
A web-based, searchable dependency report is available by adding the --scan option.
```

의존성 핸들러 메서드 `enforcedPlatform`도 사용 가능하다. 이 경우 사용하는 의존성의 버전은 import된 BOM에 정의된 버전을 이용해서 강제로 적용된다. (빌드 파일에서 의존성을 명시적으로 지정하더라도 enforcedPlatform으로 import된 BOM에 정의된 버전이 강제된다.)

```kotlin
// File: build.gradle.kts
plugins {
    java
}
 
repositories {
    jcenter()
}
 
dependencies {
    // Load bill of materials (BOM) for Spring Boot.
    // The dependencies in the BOM will be
    // dependency constraints in our build, but
    // the versions in the BOM are forced for
    // used dependencies.
    implementation(enforcedPlatform("org.springframework.boot:spring-boot-dependencies:2.1.4.RELEASE"))
 
    // Use dependency defined in BOM.
    // Version is not needed, because the version
    // defined in the BOM is a dependency constraint
    // that is used.
    implementation("commons-codec:commons-codec")
 
    // Version in BOM is 1.23 and because
    // we use enforcedPlatform the version
    // will be 1.23 once the dependency is resolved,
    // even though we define a newer version explicitly.
    implementation(group = "org.yaml",
                   name = "snakeyaml",
                   version = "1.24")
}
```

아래와 같이 `snakeyaml` 의존성 버전이 1.23으로 강제된다.

```bash
$ gradle -q dependencies --configuration compileClasspath
------------------------------------------------------------
Root project
------------------------------------------------------------
 
compileClasspath - Compile classpath for source set 'main'.
+--- org.springframework.boot:spring-boot-dependencies:2.1.4.RELEASE
|    +--- commons-codec:commons-codec:1.11 (c)
|    \--- org.yaml:snakeyaml:1.23 (c)
+--- commons-codec:commons-codec -> 1.11
\--- org.yaml:snakeyaml:1.24 -> 1.23
 
(c) - dependency constraint
A web-based, searchable dependency report is available by adding the --scan option.
$
```
