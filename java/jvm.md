# JVM 구조

![structure](/java/image/jvm/structure.png)

## 클래스 로더 시스템
- .class에서 바이트코드를 읽고 메모리에 저장
- **로딩**: 클래스를 읽어오는 과정
- **링크**: 바이트코드가 올바른지 검증. 클래스에서 찾은 static 변수 및 메소드에 메모리를 할당
- **초기화**: static block을 실행

## 메모리
- 모든 스레드에서 공유 하는 영역
    - **메소드**: 메타 데이터(클래스 이름, 부모 클래스 이름, 메소드, 변수), 런타임 상수 풀, 메소드 코드를 저장
    - **힙**: 객체를 저장
- 각 스레드마다 별도로 가지는 영역
    - **스택**: 스레드 마다 런타임 스택을 만들고, 그 안에 메소드 호출 스택을 스택 프레임이라 부르는 블럭으로 쌓는다. 메소드 호출에서 사용되는 지역 변수를 저장한다. 이러한 모든 로컬 변수를 스레드 로컬 변수라고 한다. 스레드가 종료되면 런타임 스택도 사라진다.
    - **PC(Program Counter) 레지스터**: 스레드 마다 스레드 내 현재 실행할 명령어의 실제 메모리 주소를 저장한다. (포인터)
    - **네이티브 메소드 스택**

## 실행 엔진
![interpreter](/java/image/jvm/interpreter.png)

- **인터프리터**: 바이트코드를 한 줄씩 실행
- **JIT 컴파일러**: 인터프리터의 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 네이티브 코드로 바꿔둔다. 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
- **GC**: 더 이상 참조되지 않는 객체를 모아서 정리한다.

## JNI (Java Native Interface)
- 자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법을 제공
- Native 키워드를 사용한 메소드 호출

## 네이티브 메소드 라이브러리
- C, C++로 작성된 라이브러리


> 참고
>- https://www.geeksforgeeks.org/jvm-works-jvm-architecture/
>- https://dzone.com/articles/jvm-architecture-explained
>- https://blog.jamesdbloom.com/JVMInternals.html