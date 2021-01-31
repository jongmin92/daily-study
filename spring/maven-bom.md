`POM (Project Object Model)` 개념을 기반으로하는 도구인 Maven이 `BOM (Bill Of Materials)`를 활용하는 방법에 대해서 알아보자.

# Dependency Management Concepts
BOM이 무엇인지, 어디에 사용할 수 있는지 알기 위해서는 먼저 Dependency Management에 대해서 이해해야 한다. 

## Maven POM
Maven POM은 Maven에서 종속성을 가져오고 프로젝트를 빌드하는 데 사용하는 정보 및 구성이 포함된 XML 파일이다.

## Maven BOM
BOM은 특별한 종류의 POM이다. 프로젝트의 종속성 버전을 제어하거나 정의 혹은 업데이트 하기 위한 중앙 위치를 제공하는 데 사용된다.

## Transitive Dependencies
Maven은 pom.xml에서 필요한 종속성들을 확인하고 자동으로 포함시킨다. 이 과정에서 두 가지 종속성이 특정 아티팩트의 다른 버전을 사용하고 있다면 충돌이 발생한다. 이때 Maven은 어떤 버전을 선택할까?

Maven은 "nearest definition"의 버전을 사용한다. "nearest definition"은 프로젝트의 의존성 트리(dependency tree)에서 가장 가까운 버전을 의미한다. (tree의 depth가 가장 짧은 버전이 선택된다고 생각하면 된다.) 이것을 "dependency mediation
", 의존성 중재라고 부른다.

다음의 예시로 확인해보자.
```
A -> B -> C -> D (1.4)
and
A -> E -> D (1.0)
```
프로젝트 A는 B와 E에 의존하고 있으며, B와 E는 각각 서로 다른 버전의 D 아티팩트에 의존하고 있다. 이 경우는 E를 통한 D의 depth가 가장 짧기 때문에 D (1.0)이 프로젝트 빌드에 사용된다.

이때, 사용해야 할 아티팩트 버전을 결정하는 여러 가지 기술이 있다.
- 프로젝트 POM 파일에 아티팩트의 사용하고자 하는 버전을 명시적으로 선언함으로써 버전을 보장할 수 있다. (1.4 버전의 D를 사용하도록 명시적으로 POM 파일에 선언할 수 있다.)
- DependencyManagement를 사용해서 버전을 제어할 수 있다.

## Dependency Management
Dependency Management는 의존성 정보를 중앙 집중화해서 관리하는 메커니즘이다. 여러 프로젝트가 공유하는 종속성 정보가 있다면, 공유 POM 파일인 BOM에 해당 종속성 정보들을 넣어 사용할 수 있다.
다음은 BOM 파일의 예이다.

```xml
<project ...>
     
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Baeldung-BOM</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>BaelDung-BOM</name>
    <description>parent pom</description>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>test</groupId>
                <artifactId>a</artifactId>
                <version>1.2</version>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>b</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>c</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

BOM도 결국 아티팩트 정보 및 버전을 포함할 수 있으며 `dependencyManagement`를 포함하고 있는 일반적인 POM 파일이다.

## Using the BOM File
위의 BOM 파일을 프로젝트에서 사용하면 의존성 선언시 버전 정보에 대해서 생략할 수 있게 된다. BOM 파일을 프로젝트에서 사용할 수 있는 방법으로는 2가지가 있다.

### parent 사용
```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    <parent>
        <groupId>baeldung</groupId>
        <artifactId>Baeldung-BOM</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
</project>
```

### import 사용
프로젝트는 하나의 parent만 상속할 수 있기 때문에 대규모 프로젝트에서는 효율적이지 않을 수 있다. import 방식은 필요한만큼 BOM을 가져올 수 있으므로 이 경우 더 효율적이다.
```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
     
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>baeldung</groupId>
                <artifactId>Baeldung-BOM</artifactId>
                <version>0.0.1-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

## Overwriting BOM Dependency
아티팩트 버전은 다음과 같은 순서로 결정된다.
1. 프로젝트 POM에서 직접 선언된 아티팩트의 버전
2. Parent 프로젝트에 선언된 아티팩트 버전
3. import된 POM에 선언된 버전
4. dependency mediation(의존성 중재)

> 2개의 import된 BOM에서 동일한 아티팩트가 각각 다른 버전으로 정의된 경우, 처음 선언된 BOM 파일의 버전이 우선된다.
