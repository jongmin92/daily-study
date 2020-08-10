# Dependency Resolution With Gradle
Spring Boot와 Gradle을 함께 사용할 때 의존성 버전을 결정하는 방법에는 여러가지가 있다.

## io.spring.depedency-management
- Gradle 5.0 이전 버전은 Maven BOM을 지원하지 않아서 spring의 depedency-management 플러그인을 사용하면 된다.
- `org.springframework.boot`랑 같이 쓰이면 자동으로 org.springframework.boot의 버전으로 BOM 설정하는 기능
  - 별도의 configuration 없이는 동작 안 함. 근데 그거에 대한 디폴트 설정을 org.springframework.boot 가 가지고 있음. (자기 자신의 버전과 동일한 BOM을 설정함.)
- BOM에 선언된 버전으로 '버전이 설정되지 않은 컴포넌트의 버전을 설정해 준다'
- BOM에 선언된 것보다 낮은 버전으로 내리는 것이 용이
- BOM Property를 지원

## platform
- Gradle 5.0부터 지원한다.
- java-platform 플러그인에서 제공하며, 프로젝트 또는 BOM을 통한 통합 버전 관리
- 의존성 선언에서 버전이 없거나, 낮은 버전이 기록되어 있을 때 BOM에 적힌 버전으로 교체
- 높은 버전으로 의존성의 버전업하는 것은 Gradle의 기본 정책이라 문제 없고, 낮은 버전으로 낮추려면 정책 설정을 하거나 force를 써야 함

## enforcedPlatform 
- 다 platform이랑 똑같은데 얘는 높은 버전이든 낮은 버전이든 BOM에 적인 버전으로 전부 강제


**결과적으로 버전을 바꾸러면 force나 정책으로 해결해거나 platform project에서 적당히 exclude해서 처리해야 함**

## org.springframework.boot
추가적으로 `org.springframework.boot`는 java 플러그인과 연계되어 있는 java-library 플러그인이 설정한 태스크를 꺼내서 다시 고쳐주는 기능을 가지고 있다.
특히 jar, war 태스크를 disable 시키고 평범한 jar/war 작업 후 unzip, spring boot class loader 코드를 주입하여 다시 jar나 war로 묶는 기능을 갖고있다.

따라서 org.springframework.boot는 java, kotlin, java-library, jar, war 보다 아래에 선언되어야 한다. (그렇지 않으면 enabled = true, false를 또 설정해야 한다.)

문제가 되는 구체적인 상황은 spring boot BOM으로 버전 관리가 필요한 일반 jar 파일을 빌드할 때이다
- boot 플러그인을 적용 안 하면 추가 설정이 필요함
- boot 플러그인을 적용하면 jar, war가 비활성화 됨
