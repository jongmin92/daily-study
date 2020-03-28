# Java, JVM, JRE, JDK

![structure](/java/image/java-jvm-jre-jdk/structure.jpg)

## JVM (Java Virtual Machine)
- 자바 가상 머신으로 자바 바이트 코드(.class 파일)를 OS에 특화된 코드로 변환(인터프리터와 JIT 컴파일러)하여 실행한다.
- 자바 바이트 코드를 실행할 수있는 런타임 환경을 제공하는 사양(스팩)이다.
- 바이트 코드를 실행하는 표준(JVM 자체는 표준)이자 구현체(특정 밴더가 구현한 JVM)이다.
    - JVM 스팩: https://docs.oracle.com/javase/specs/jvms/se11/html/
    - JVM 밴터: 오라클, 아마존, Azul, ...
    - 특정 플랫폼(OS)에 종속적이다.
    > 가상 머신(virtual Machine)이란 여러 가지로 정의할 수 있지만, 프로그램을 실행하기 위해 물리적 머신(즉, 컴퓨터)과 유사한 머신을 소프트웨어로 구현한 것이다. 자바는 원래 WORA(Write Once Run Anywhere)를 구현하기 위해 물리적인 머신과 별개의 가상 머신을 기반으로 동작하도록 설계되었다. 그래서 자바 바이트코드를 실행하고자 하는 모든 하드웨어에 JVM을 동작시킴으로써 자바 실행 코드를 변경하지 않고도 모든 종류의 하드웨어에서 동작한다. (JVM 자체가 특정 하드웨어에 종속적이다.)

## JRE (Java Runtime Environment)
**JVM + 라이브러리**
- 자바 애플리케이션을 실행(Run)할 수 있도록 구성된 배포판.
- JVM과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 갖고 있다.
- 개발 관련 도구는 포함하지 않는다. (JDK에서 제공)

## JDK (Java Development Kit)
**JRE + 개발 툴**
- 소스 코드를 작성할 때 사용하는 자바 언어는 플랫폼에 독립적이다. -> WORA
- 오라클은 자바 11부터는 JDK만 제공하며 JRE는 따로 제공하지 않는다.

## Java
- 프로그래밍 언어
- JDK에 들어있는 자바 컴파일러(javac)를 사용하여 바이트코드(.class 파일)로 컴파일할 수 있다.
- 자바 유료화
    - `오라클`에서 만든 `Oracle JDK 11 버전부터` `상용`으로 사용할 때 유료이다.

## JVM 언어
- JVM 기반으로 동작하는 프로그래밍 언어
- 최초의 JVM은 자바만을 지원하기 위해 만들어졌다. 하지만 자바에 대한 의존성이 타이트하지 않다. 어떠한 다른 프로그래밍 언어로 코딩하더라도 그 언어를 컴파일했을 때, class 파일이 만들어진다거나 java파일이 만들어지면 JVM을 사용할 수 있다.
- 클로저, 그루비, JRuby, Jython, Kotlin, Scala, ...
    - https://en.wikipedia.org/wiki/List_of_JVM_languages

>참고
>- https://howtodoinjava.com/java/basics/jdk-jre-jvm/ 
