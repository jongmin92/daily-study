# 리플렉션

**자바 리플랙션은 런타임(Runtime) 시점에 객체를 통해 클래스의 정보를 분석하고 수정할 수 있는 기능이다.**

리플랙션 API를 사용하면 런타임에 필드, 메서드, 생성자 등을 포함하는 클래스 및 해당 멤버를 조작할 수 있다. (private 접근 지시자를 가진 멤버까지 조작이 가능하다.)

[java.lang.reflect](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html) 패키지는 리플랙션을 구현하기 위한 클래스들을 제공한다.

[java.lang.Class](https://docs.oracle.com/javase/8/docs/api/index.html) 클래스의 메소드는 특정 클래스의 메타 데이터를 수집하는데 사용한다.
> 실행중인 Java 프로세스의 클래스와 인터페이스를 나타내는 객체이다. (enum은 클래스의 종류이고, annotation은 인터페이스의 한 종류이다.)  
Class<T>의 객체는 JVM 기동시, 클래스 로더에 의해 클래스가 로딩될 때, 힙 영역에 생성된다.

## Class<T>에 접근하는 방법
- `타입.class`를 사용
- `인스턴스.getClass()`를 사용
- `Class.forName("FQCN")`를 사용
    - ClassPath에 해당 클래스가 없다면 ClassNotFoundException이 발생한다.

## Class<T>를 통해 할 수 있는 것
- 필드 (목록) 가져오기
- 메소드 (목록) 가져오기
- 상위 클래스 가져오기
- 인터페이스 (목록) 가져오기
- 애노테이션 (목록) 가져오기
- 생성자 가져오기
- ...

> 예제 코드: https://github.com/jongmin92/code-examples/tree/master/java/reflection
