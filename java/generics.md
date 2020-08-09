# 제네릭 (Generics)

JDK 1.5에 처음 도입되었다.
- 제네렉은 컴파일 타임에 더 많은 버그를 탐지할 수 있도록하여 코드의 안정성을 높인다.
- 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법을 말한다.
- 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입 체크를 해주는 기능이다.

즉, 런타임에 발생할 수 있는 에러를 컴파일 시의 타입 체크를 통해 사전에 탐지할 수 있도록 한다.

# 제네릭을 사용하지 않는 경우 문제점

```java
public class NoGenerics {

    public static void main(String[] args) {
        final SimpleArrayList list = new SimpleArrayList();

        list.add("1st");
        list.add("2nd");

        final Integer value1 = (Integer) list.get(0);
        final Integer value2 = (Integer) list.get(0);

        System.out.println(value1);
        System.out.println(value2);
    }

    public static class SimpleArrayList {
        private int size;
        private final Object[] element = new Object[5];

        public void add(Object value) {
            element[size++] = value;
        }

        public Object get(int idx) {
            return element[idx];
        }
    }
}
```

```
Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer (java.lang.String and java.lang.Integer are in module java.base of loader 'bootstrap')
	at com.example.test.NoGenerics.main(NoGenerics.java:11)
```

위와 같이 컴파일은 잘 되지만, 실행하면 런타임에 에러가 발생할 수 있다.
이런 문제를 해결하기 위해 Integer 타입만을 갖는 SimpleIntegerArrayList와 String 타입만을 갖는 SimpleStringArrayList를 만들 수 있다.

## SimpleIntegerArrayList
```java
public class SimpleIntegerArrayList {
    private int size;
    private final Integer[] element = new int[5];

    public void add(Integer value) {
        element[size++] = value;
    }

    public Integer get(int idx) {
        return element[idx];
    }
}
```

## SimpleStringArrayList
```java
public class SimpleStringArrayList {
    private int size;
    private final String[] element = new int[5];

    public void add(int value) {
        element[size++] = value;
    }

    public int get(int idx) {
        return element[idx];
    }
}
```

타입을 제외한 모든 부분에서 코드의 중복이 발생한다. 이러한 경우에 제네릭을 사용해서 문제를 해결할 수 있다.

# 제네릭을 사용해서 문제 해결

```java
public static class GenericArrayList<T> {
    private int size;
    private final Object[] element = new Object[5];

    public void add(T value) {
        element[size++] = value;
    }

    public T get(int idx) {
        return (T) element[idx];
    }
}
```

`<T>`로 표현한 것이 제네릭이다. 객체를 생성할 때 타입을 지정하면, 생성되는 객체 안에서는 T의 위치에 지정한 타입이 대체되어서 들어가는 것처럼 컴파일러에 의해 필요한 곳에 형변환 코드가 추가된다.

```java
public class Generics {

    public static void main(String[] args) {
        final GenericArrayList<String> list = new GenericArrayList<>();

        list.add("1st");
        list.add("2nd");

        final String value1 = list.get(0);
        final String value2 = list.get(1);

        System.out.println(value1);
        System.out.println(value2);
    }
}
```

형변환이 필요없다는 것, 지정한 타입과 다른 타입의 참조 변수를 선언하면 컴파일 타임에 오류가 발생한다는 것이 중요 포인트다.

바이트코드로 컴파일된 Generics.class를 Intellij IDEA를 통해 디컴파일해보면 다음과 같다.

```java
public class Generics {
    public Generics() {
    }

    public static void main(String[] args) {
        GenericArrayList<String> list = new GenericArrayList();
        list.add("1st");
        list.add("2nd");
        String value1 = (String)list.get(0);
        String value2 = (String)list.get(1);
        System.out.println(value1);
        System.out.println(value2);
    }
}
```

값을 꺼내 사용하는 곳에 제네릭을 통해 전달받은 타입으로 형변환하는 코드가 추가되었다. 제네릭을 사용하면 컴파일러가 알아서 형변환을 진행하는 것을 확인할 수 있다.

위의 디컴파일된 코드에는 제네릭 타입(String)이 확인 가능하게 보이지만, 실제로 바이트코드에서는 GenericArrayList 관련 부분에 제네릭 타입은 확인할 수 없다. (주석으로만 남아 있는 제네릭(타입) 정보를 Intellij IDEA에서 디컴파일 할 때 참고해서 보기좋게 보여주는 것 같다.)

```java
// class version 52.0 (52)
// access flags 0x21
public class com/example/test/Generics {

  // compiled from: Generics.java

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/example/test/Generics; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x9
  public static main([Ljava/lang/String;)V
    // parameter  args
   L0
    LINENUMBER 6 L0
    NEW com/example/test/GenericArrayList
    DUP
    INVOKESPECIAL com/example/test/GenericArrayList.<init> ()V
    ASTORE 1
   L1
    LINENUMBER 8 L1
    ALOAD 1
    LDC "1st"
    INVOKEVIRTUAL com/example/test/GenericArrayList.add (Ljava/lang/Object;)V
   L2
    LINENUMBER 9 L2
    ALOAD 1
    LDC "2nd"
    INVOKEVIRTUAL com/example/test/GenericArrayList.add (Ljava/lang/Object;)V
   L3
    LINENUMBER 11 L3
    ALOAD 1
    ICONST_0
    INVOKEVIRTUAL com/example/test/GenericArrayList.get (I)Ljava/lang/Object;
    CHECKCAST java/lang/String
    ASTORE 2
   L4
    LINENUMBER 12 L4
    ALOAD 1
    ICONST_1
    INVOKEVIRTUAL com/example/test/GenericArrayList.get (I)Ljava/lang/Object;
    CHECKCAST java/lang/String
    ASTORE 3
   L5
    LINENUMBER 14 L5
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 2
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L6
    LINENUMBER 15 L6
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 3
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L7
    LINENUMBER 16 L7
    RETURN
   L8
    LOCALVARIABLE args [Ljava/lang/String; L0 L8 0
    LOCALVARIABLE list Lcom/example/test/GenericArrayList; L1 L8 1
    // signature Lcom/example/test/GenericArrayList<Ljava/lang/String;>;
    // declaration: list extends com.example.test.GenericArrayList<java.lang.String>
    LOCALVARIABLE value1 Ljava/lang/String; L4 L8 2
    LOCALVARIABLE value2 Ljava/lang/String; L5 L8 3
    MAXSTACK = 2
    MAXLOCALS = 4
}
```

GenericArrayList.class의 바이트코드를 보면 조금 더 정확하게 확인할 수 있다.

```java
// class version 52.0 (52)
// access flags 0x21
// signature <T:Ljava/lang/Object;>Ljava/lang/Object;
// declaration: com/example/test/GenericArrayList<T>
public class com/example/test/GenericArrayList {

  // compiled from: GenericArrayList.java

  // access flags 0x2
  private I size

  // access flags 0x12
  private final [Ljava/lang/Object; element

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
   L1
    LINENUMBER 5 L1
    ALOAD 0
    ICONST_5
    ANEWARRAY java/lang/Object
    PUTFIELD com/example/test/GenericArrayList.element : [Ljava/lang/Object;
    RETURN
   L2
    LOCALVARIABLE this Lcom/example/test/GenericArrayList; L0 L2 0
    // signature Lcom/example/test/GenericArrayList<TT;>;
    // declaration: this extends com.example.test.GenericArrayList<T>
    MAXSTACK = 2
    MAXLOCALS = 1

  // access flags 0x1
  // signature (TT;)V
  // declaration: void add(T)
  public add(Ljava/lang/Object;)V
    // parameter  value
   L0
    LINENUMBER 8 L0
    ALOAD 0
    GETFIELD com/example/test/GenericArrayList.element : [Ljava/lang/Object;
    ALOAD 0
    DUP
    GETFIELD com/example/test/GenericArrayList.size : I
    DUP_X1
    ICONST_1
    IADD
    PUTFIELD com/example/test/GenericArrayList.size : I
    ALOAD 1
    AASTORE
   L1
    LINENUMBER 9 L1
    RETURN
   L2
    LOCALVARIABLE this Lcom/example/test/GenericArrayList; L0 L2 0
    // signature Lcom/example/test/GenericArrayList<TT;>;
    // declaration: this extends com.example.test.GenericArrayList<T>
    LOCALVARIABLE value Ljava/lang/Object; L0 L2 1
    // signature TT;
    // declaration: value extends T
    MAXSTACK = 5
    MAXLOCALS = 2

  // access flags 0x1
  // signature (I)TT;
  // declaration: T get(int)
  public get(I)Ljava/lang/Object;
    // parameter  idx
   L0
    LINENUMBER 12 L0
    ALOAD 0
    GETFIELD com/example/test/GenericArrayList.element : [Ljava/lang/Object;
    ILOAD 1
    AALOAD
    ARETURN
   L1
    LOCALVARIABLE this Lcom/example/test/GenericArrayList; L0 L1 0
    // signature Lcom/example/test/GenericArrayList<TT;>;
    // declaration: this extends com.example.test.GenericArrayList<T>
    LOCALVARIABLE idx I L0 L1 1
    MAXSTACK = 2
    MAXLOCALS = 2
}
```

제네릭 타입이 선언됐던 add의 파라미터와 get의 리턴 타입이 전부 Object인 것을 확인할 수 있다.

# 제네릭의 타입 소거 (Type Erasure)

위의 바이트코드에서 확인한 것처럼 바이트코드 상에서는 제네릭 타입이 전부 Object로 변경되어 있는 것을 확인할 수 있다. 즉 런타임 시점에 실행되는 이 바이트코드로는 제네릭 타입 정보를 알 수 없는 것이다.

이를 제네릭의 타입 소거라고 한다. (타입을 컴파일 타임에만 검사하고 런타임에는 해당 타입 정보를 알 수 없다.)

Java 컴파일러는 타입 소거를 아래와 같이 적용한다.
- 제네렉 타입 파라미터(T)를 Object로 변경한다.
    - Object로 변경하는 경우는 unbounded된 경우를 의미하며, <T extends Integer>와 같이 bound가 되어 있지 않은 경우를 의미한다.
    - 이 소거 규칙은 제네릭을 적용할 수 있는 일반 클래스, 인터페이스, 메서드에 해당된다.
- 타입 안정성 보존을 위해 필요하다면 type casting을 넣어준다.
- 확장된 제네릭 타입에서 다형성을 보존하기 위해 bridge method를 생성한다.

> 제네릭을 도입한 JDK 1.5는 기존의 코드에 대해서 모두 호환성을 유지하면서 제네릭을 사용해야 했다. 따라서 코드의 호환성 때문에 로타입의 지원 + 제네릭을 구현할 때 소거하는 방식을 이용했다.  
>
> 로타입: 제네릭 타입에서 타입 매개변수를 사용하지 않은 경우. GenericArrayList<T>에서 GenericArrayList로만 선언해서 사용하는 경우를 말한다. 
