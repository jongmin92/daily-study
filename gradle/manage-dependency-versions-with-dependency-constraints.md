# Manage Dependency Versions With Dependency Constraints
Maven 빌드를 위한 POM 파일에는 `dependencyManagement` 섹션을 정의할 수 있다. 이 섹션에는 의존성에 대한 버전을 선언할 수 있다. 선언된 의존성 버전은 `dependencies` 섹션에서 버전 없이 의존성을 사용하는 경우 참조된다.

Gradle에서도 `의존성 제약(dependency constraints)` 조건을 사용해서 같은 작업을 수행할 수 있다. **의존성 제약 조건은 빌드 파일에 정의된 의존성에 대한 버전 혹은 전이 의존성에 대한 버전을 정의하기 위해 사용할 수 있다.** 

multi-project 빌드에서 의존성 제약 조건을 사용한다면 root 빌드 파일에 의존성 버전을 정의하고 각 프로젝트 빌드 파일에서는 의존성에 대한 버전을 명시하지 않고 사용할 수 있다.

# Example
```kotlin
// File: build.gradle.kts
plugins {
    groovy
}
 
repositories {
    jcenter()
}
 
// In a multi-project build, this dependencies
// block with constraints could be in the
// root build file, so all versions are
// defined in one place.
dependencies {
    constraints {
        // Define dependency with version to be used.
        // This version is used when we define a dependency
        // for guava without a version.
        implementation("com.google.guava:guava:27.1-jre")
 
        // Constraints are scoped to configurations,
        // so we can make specific constraints for a configuration.
        // In this case we want the dependency on Spock defined
        // in the testImplementation configuration to be a specific version.
        // Here we use named arguments to define the dependency constraint.
        testImplementation(group = "org.spockframework",
                           name = "spock-core",
                           version = "1.3-groovy-2.5")
    }
}
 
// In a multi-project build this dependencies block
// could be in subprojects, where the dependency
// declarations do not need a version, because the
// versions are defined in the root build file using
// constraints.
dependencies {
    // Because of the dependency constraint,
    // we don't have to specify the dependency version here.
    implementation("com.google.guava:guava")
 
    // Another dependency without version for the
    // testImplementation configuration.
    testImplementation("org.spockframework:spock-core")
}
```

`compileClasspath`에 대한 dependencies task로 의존성이 어떻게 resolve되는지 확인할 수 있다.

```bash
$ gradle -q dependencies --configuration compileClasspath
------------------------------------------------------------
Root project
------------------------------------------------------------
 
compileClasspath - Compile classpath for source set 'main'.
+--- com.google.guava:guava -> 27.1-jre
|    +--- com.google.guava:failureaccess:1.0.1
|    +--- com.google.guava:listenablefuture:9999.0-empty-to-avoid-conflict-with-guava
|    +--- com.google.code.findbugs:jsr305:3.0.2
|    +--- org.checkerframework:checker-qual:2.5.2
|    +--- com.google.errorprone:error_prone_annotations:2.2.0
|    +--- com.google.j2objc:j2objc-annotations:1.1
|    \--- org.codehaus.mojo:animal-sniffer-annotations:1.17
\--- com.google.guava:guava:27.1-jre (c)
 
(c) - dependency constraint
A web-based, searchable dependency report is available by adding the --scan option.
```

Guava 의존성 버전이 27.1-jre로 resolve된 것을 확인할 수 있다. 위의 dependencies 결과에서 `(c)`는 의존성 제약(dependency constraint) 조건을 사용해서 의존성을 resolve한 것을 의미한다.

`testCompileClasspath`에 대한 dependencies task는 테스트 의존성이 어떻게 resolve되는지 확인할 수 있다.

```bash
$ gradle -q dependencies --configuration testCompileClasspath
> Task :dependencies
 
------------------------------------------------------------
Root project
------------------------------------------------------------
 
testCompileClasspath - Compile classpath for source set 'test'.
+--- com.google.guava:guava -> 27.1-jre
|    +--- com.google.guava:failureaccess:1.0.1
|    +--- com.google.guava:listenablefuture:9999.0-empty-to-avoid-conflict-with-guava
|    +--- com.google.code.findbugs:jsr305:3.0.2
|    +--- org.checkerframework:checker-qual:2.5.2
|    +--- com.google.errorprone:error_prone_annotations:2.2.0
|    +--- com.google.j2objc:j2objc-annotations:1.1
|    \--- org.codehaus.mojo:animal-sniffer-annotations:1.17
+--- com.google.guava:guava:27.1-jre (c)
+--- org.spockframework:spock-core:1.3-groovy-2.5
|    +--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-json:2.5.4
|    |    \--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-nio:2.5.4
|    |    \--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-macro:2.5.4
|    |    \--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-templates:2.5.4
|    |    +--- org.codehaus.groovy:groovy:2.5.4
|    |    \--- org.codehaus.groovy:groovy-xml:2.5.4
|    |         \--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-test:2.5.4
|    |    +--- org.codehaus.groovy:groovy:2.5.4
|    |    \--- junit:junit:4.12
|    |         \--- org.hamcrest:hamcrest-core:1.3
|    +--- org.codehaus.groovy:groovy-sql:2.5.4
|    |    \--- org.codehaus.groovy:groovy:2.5.4
|    +--- org.codehaus.groovy:groovy-xml:2.5.4 (*)
|    \--- junit:junit:4.12 (*)
\--- org.spockframework:spock-core -> 1.3-groovy-2.5 (*)
 
(c) - dependency constraint
(*) - dependencies omitted (listed previously)
 
A web-based, searchable dependency report is available by adding the --scan option.
```
