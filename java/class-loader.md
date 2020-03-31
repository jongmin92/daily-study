# 클래스 로더

![structure](/java/image/class-loader/structure.png)

**로딩, 링크, 초기화 순으로 진행된다.**

## 로딩
- 클래스 로더가 .class 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만들고 "메소드" 영역에 저장한다.
    - FQCN (Fully Qualified Class Name)
    - Class, Interface, Enum
    - 메소드, 변수
- 로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성하여 "힙" 영역에 저장한다.

## 링크
Verify, Prepare, Resolve 세 단계로 나뉘어진다.
- **Verify**: .class 파일 형식이 유효한지 체크한다.
- **Prepare**: 클래스 변수(static 변수)와 기본 값에 필요한 메모리
- **Resolve**: 심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체한다.

## 초기화
Static 변수의 값을 할당한다. (static 블럭이 있다면 이때 실행된다.)

---

- 클래스 로더 계층 구조
    - 클래스 로더는 계층 구조로 이뤄져 있으며, 기볹거으로 세가지 클래스 로더가 제공된다.
    - **부트 스트랩 클래스 로더**: 최상위 우선순위를 가진 클래스로더. JAVA_HOME\lib에 있는 코어 자바 API를 제공한다.
    - **플랫폼 클래스 로더**: 이전에는 extension 클래스 로더라고 불렀다. JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
    - **애플리케이션 클래스 로더**: 애플리케이션 클래스 패스(-classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치)에서 클래스를 읽는다.


```java
public class Ex2 {
    public static void main(String[] args) {
        final ClassLoader classLoader = Ex2.class.getClassLoader();
        System.out.println(classLoader);
        System.out.println(classLoader.getParent());
        System.out.println(classLoader.getParent().getParent());
    }
}

------ 실행 결과 ------
sun.misc.Launcher$AppClassLoader@7852e922
sun.misc.Launcher$ExtClassLoader@5c647e05
null
```

Bootstrap 클래스 로더는 네이티브 코드로 구현되어 있는 클래스 로더라서 VM마다 다 다르고, 자바 코드에서 참조해 출력할 수 없다.

![order](/java/image/class-loader/classloader-order.jpg)

모든 부모 클래스 로더에게 요청하고 못 읽으면 본인이 읽기를 시도한다. 본인도 못읽으면 `ClassNotFoundException`이 발생한다.
