# Gradle VS Maven Comparison

## Flexibility
Google은 Android의 공식 build tool로서 Gradle을 선택했다.
Gradle과 Maven 모두 convention over configuration을 제공하지만, Maven은 configuration을 customization 하게 만들기에는 쉽지 않은 모델 구조를 갖고 있다. 이런 이유로 Maven build는 쉽게 이해할 수 있지만 특별한 요구 사항에 기반한 자동화 관련 문제를 해결하기 위해서 사용하기에는 부적합하다.

## Performance
build time을 단축시키는 것은 배포 관련 작업에서 가장 중요한 것 중 하나이다. Gradle과 Maven 모두 parallel project build와 parallel dependency resolution을 지원한다. 이 과정에서 Gradle은 work avoidance와 incrementality 메커니즘을 기반으로 Maven보다 훨씬 빠른 성능을 제공한다.
- [Incrementality](https://blog.gradle.org/introducing-incremental-build-support): Gradle은 task의 Input과 Output을 tracking 함으로써 가능한 경우 변경된 파일만 처리함으로써 불필요한 작업을 피한다.
- [Build Cache](https://blog.gradle.org/introducing-gradle-build-cache): 같은 input에 대한 build output을 재사용한다.
- [Gradle Daemon](https://docs.gradle.org/current/userguide/gradle_daemon.html): build 정보를 메모리에서 "hot" 상태로 오래 유지하는 프로세스이다.
 
![build-time](/gradle/image/gradle-vs-maven-comparison/build-time.png)

거의 모든 시나리오에서 Gradle이 Maven에 비해 최소 두 배 더 빠르다. (빌드 캐시를 사용하는 대규모 빌드의 경우에는 100배 더 빠르다.)

## User Experience
Maven은 오랫동안 사용되어 오면서 IDE를 통한 지원이 잘 제공되고 있다. 그러나 Gradle의 IDE 지원 또한 빠르게 발전하고 있다. Gradle은 Kotlin 기반의 DSL을 제공하면서 더 나은 IDE 지원 환경을 제공하고 있다.

IDE 지원도 중요하지만 여전히 많은 사용자는 command-line interface를 통해서 빌드 작업을 실행한다. Gradle은 더 나은 로깅과 'gradle tasks'와 같은 discoverability(검색 기능)이 있는 modern한 CLI를 제공한다.

마지막으로 Gradle은 디버깅 및 최적화를 위한 interactive web-based UI를 제공한다. -> [build scans](https://gradle.com/build-scans/?_ga=2.25537149.1344853913.1587296453-3879792.1583141850)

## Dependency Management
Gradle과 Maven 모두 dependency를 로컬에 캐시하고 병렬로 다운로드 할 수 있다.

Maven은 build-in dependency scope이 없기 때문에 unit test와 integration test를 분리하기 위해서는 다소 어색한 모듈 아키텍처가 필요하다. Gradle은 custom dependency scope을 이용해서 더 나은 모델링과 빠른 빌드를 제공한다.

Maven은 dependency conflict를 해결 할 때, 가장 짧은 경로로 작동하기 때문에 선언 순서에 영향을 받는다. Gradle은 dependency graph에서 발견 된 가장 높은 dependency version을 선택해서 해결한다. 또한 Gradle을 사용해서 transitive version 보다 우선해서 dependency 버전을 downgrade할 수 있다.
