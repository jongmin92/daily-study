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

위와 같은 충돌 상황을 해결하는 여러가지 방법이 있다. 의존성을 관리하는 tool 마다 이러한 버전 충돌을 해겨랗는 방법이 다르다.

> Apache Maven은 가장 가까운 첫 번째 전략을 사용한다.  
>  
> Maven은 종속성에 대한 최단 경로(shortest path)와 해당 버전을 사용한다. 같은 길이의 여러 경로가 있는 경우 첫 번째 경로를 사용한다.  
>  
> 그러므로 위의 예에서 `guava`의 버전이 `20.0`이 될 것이다. 프로젝트에서 직접 의존하고 있는 것이 guice에서 의존하고 있는 것보다 경로가 더 짧기(가깝기) 때문이다.  
>  
> 이 방법의 단점은 의존성 순서에 따라 다르다는 것이다. 매우 큰 의존성 그래프로 순서를 유지하는 것은 어려울 수 있다. (ex. 새로운 버전의 의존성이 이전 버전과 다른 순서로 자체 의존성 선언을 갖는 경우)

Gradle은 의존성 그래프에 표시되는 모든 요청 버전을 고려한다. 이 버전 중에서 가장 높은 버전을 선택해서 사용한다.

Gradle은 [rich version declaration](https://docs.gradle.org/current/userguide/rich_versions.html) 개념을 지원하므로 가장 높은 버전은 버전이 선언된 방식에 따라 다르다.

## Implementation conflict resolution
