# 애노테이션 프로세서

**애노테이션 프로세싱은 컴파일 타임에 애노테이션들을 스캐닝하고 처리하는 javac에 속한 빌드툴이다.** 특정 애노테이션들을 위해 애노테이션 프로세서를 만들어서 등록할 수 있다.
애노테이션 프로세싱은 자바 1.6부터 사용 가능하다.

[Processor](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Processor.html) 인터페이스를 직접 구현해도 되지만, 기본으로 제공되는 [AbstractProcessor](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/AbstractProcessor.html) 추상 클래스를 사용하자.

## 주요 메서드
- getSupportedAnnotationTypes: 프로세서가 처리할 애노테이션들을 지정한다.
- getSupportedSourceVersion: 소스코드 버전을 몇까지 지원할 것인지 지정한다.
- process: 특정 애노테이션을 가진 엘리먼트(소스코드의 구성요소)를 찾아 처리한다.
    - 애노테이션 프로세서는 라운드라는 개념으로 처리를 한다.
    - 여러 라운드에 거쳐 처리한다.
    - 각 라운드마다 프로세서에게 특정 애노테이션을 갖고 있는 엘리먼트를 찾으면 처리를 요청한다.
    - 처리된 결과가 다음 라운드에게 전달될 수 있다.
    - true를 리턴하면, 다른 프로세서가 이를 처리하지 않는다.

## JAR 패키징
- JAR 패키징을 위해서는 MANIFEST 정보를 생성해야한다.
- src/main/resource/services 하위에 javax.annotation.processing.Processor 파일을 생성한다.
- 생성한 프로세서의 풀 패키지 경로를 적는다.

>AutoService  
>AutoService를 사용하면 자동적으로 MANIFEST 파일을 생성해준다.
>컴파일 시점에 애노테이션 프로세서를 사용하여 META-INF/services/javax.annotation.processor.Processor 파일을 자동으로 생성한다.











