# Dependency Resolution

의존성 해결(Dependency Resolution)은 의존성 그래프(Dependency Graph)가 완료 될 때까지 반복되는 두 단계로 구성되는 프로세스이다.
- 그래프에 새로운 의존성이 추가되면, 그래프에 추가될 버전을 결정하기 위해 의존성 해결을 수행한다.
- 특정 의존성이 버전과 함께 그래프의 일부로 식별되면, 의존성을 차례로 추가할 수 있도록 메타 데이터를 검색한다.

## How Gradle handles conflicts?
