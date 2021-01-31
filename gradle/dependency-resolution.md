# Dependency Resolution

의존성 해결(Dependency Resolution)은 의존성 그래프(Dependency Graph)가 완료 될 때까지 반복되는 두 단계로 구성되는 프로세스이다.
- 그래프에 새로운 의존성이 추가되면, 그래프에 추가될 버전을 결정하기 위해 의존성 해결을 수행한다.
- 특정 의존성이 버전과 함께 그래프의 일부로 식별되면, 의존성을 차례로 추가할 수 있도록 메타 데이터를 검색한다.

## How Gradle handles conflicts?

의존성 해결을 수행할 때 Gradle은 두 가지 유형의 충돌을 처리한다.
- Version conflicts
    - 두 개 이상의 의존성이 특정 의존성을 요구하지만 버전이 다른 경우
- Implementation conflicts
    - 의존성 그래프에 같은 implementation 또는 기능을 제공하는 모듈이 포함된 경우

의존성 해결 프로세스는 enterprise 요구 사항을 충족할 수 있도록 사용자가 직접 정의할 수도 있다. 
자세한 내용은 [Controlling transitive dependencies](https://docs.gradle.org/current/userguide/dependency_constraints.html) 를 참고한다.

## Version conflict resolution

두개의 컴포넌트가 다음과 같은 겅우 버전 충돌이 발생한다.
- 같은 모듈에 의존한다. ex) `com.google.guava:guava`
- 그러나 서로 다른 버전을 사용한다. ex) `20.0`, `25.1-android`
    - 프로젝트는 `com.google.guava:guava:20.0`에 직접 의존하고 있다.
    - 프로젝트에서 사용하는 `com.google.inject:guice:4.2.2`는 `com.google.guava:guava:25.1-android`에 의존하고 있다.

### Resolution strategy

위와 같은 충돌 상황을 해결하는 여러가지 방법이 있다. 의존성을 관리하는 tool 마다 이러한 버전 충돌을 해결하는 방법이 다르다.

> Apache Maven은 가장 가까운 첫 번째 전략을 사용한다.  
>  
> Maven은 종속성에 대한 최단 경로(shortest path)와 해당 버전을 사용한다. 같은 길이의 여러 경로가 있는 경우 첫 번째 경로를 사용한다.  
>  
> 그러므로 위의 예에서 `guava`의 버전이 `20.0`이 될 것이다. 프로젝트에서 직접 의존하고 있는 것이 guice에서 의존하고 있는 것보다 경로가 더 짧기(가깝기) 때문이다.  
>  
> 이 방법의 단점은 의존성 순서에 따라 다르다는 것이다. 매우 큰 의존성 그래프로 순서를 유지하는 것은 어려울 수 있다. (ex. 새로운 버전의 의존성이 이전 버전과 다른 순서로 자체 의존성 선언을 갖는 경우)

Gradle은 의존성 그래프에 표시되는 모든 요청 버전을 고려한다. 이 버전 중에서 가장 높은 버전을 선택해서 사용한다.

Gradle은 [rich version declaration](https://docs.gradle.org/current/userguide/rich_versions.html) 개념을 지원하므로 가장 높은 버전은 버전이 선언된 방식에 따라 다르다.

## The Dependency Cache

Gradle에는 매우 정교한 캐싱 메커니즘이 포함되어 있어서 의존성 해결 과정에서 remote request 수를 최소화한다.

Gradle 의존성 캐시는 `GRADLE_USER_HOME/caches`에 있는 두 가지 storage 유형으로 구성된다.
- POM 파일과 같은 metadata를 포함하는 jar 같은 바이너리 아티팩트의 파일 기반 저장소. 다운로드된 아티팩트의 storage 경로에는 SHA1 checksum을 포함한다. 이는 동일한 이름이지만 컨텐츠가 다른 2개의 아티팩트를 캐시 할 수 있음을 의미한다.
- dynamic 버전, module descriptor, 아티팩트 해결 결과를 포함한 해결 된 module metadata의 바이너리 저장소.

### Chache Cleanup

Gradle은 어떤 의존성 캐시를 사용했는지에 대한 정보를 유지한다. 이 정보를 사용해서 30일 이상 사용되지 않은 아티팩트에 대해 캐시를 정기적으로 최대 24시간 마다 스캔한다. 사용되지 않는 아티팩트는 삭제해서 캐시가 무한하게 커지지 않도록한다.
