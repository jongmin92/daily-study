# reified를 사용하는 이유

## 사용하는 이유
```kotlin
fun <T> genericFunc(clazz: Class<T>)
```
일반적인 제네릭 함수 body에서 타입 `T`는 컴파일 타임에는 존재하지만 런타임에는 [Type erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html) 때문에 접근할 수 없다. 일반적인 클래스에서 작성뙨 함수 body
에서 제네릭 타입에 접근하고 싶다면 genericcFunc 처럼 명시적으로 타입을 파라미터로 전달해주어야 한다.

하지만 **reified type parameter**와 함께 **inline 함수**를 만들면 추가적으로 `Class<T>`를 파라미터로 넘겨줄 필요없이 런타임에 타입 `T`에 접근할 수 있다.

```kotlin
inline fun <reified T> genericFunc()
```

## 동작 방식
`reified`는 `inline` 함수와 조합해서만 사용할 수 있다. inline 함수는 컴파일러가 함수의 바이트코드를 함수가 사용되는 모든 곳에 복사하도록 만든다. reified type과 함께 인라인 함수가 호출되면 컴파일러는 type argument
로 사용된 실제 타입을 앙ㄹ고 만들어진 바이트코드를 직접 클래스에 대응되도록 바꿔준다.

## 예시
[kotlin jackson moodule](https://github.com/FasterXML/jackson-module-kotlin) 프로젝트에서 예를 찾아볼 수 있다.

### reified type 없이 접근하기 
```kotlin
// 컴파일 실패
fun <T> String.toKotlinObject(): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(this, T::class.java)
}
```

`readValue` 메서드는 `JsonObject`를 파싱하는데 사용하기 위해 타입 하나를 받는다. 타입 파라미터 `T`의 Class를 얻으려고 하면 컴파일 에러가 발생한다. **"Can not use 'T' as reifiedd type parameter. Use a class instead."**

### 명시적으로 Classs 파라미터를 전달하기
```kotlin
fun <T : Any> String.toKotlinObject(clazz: KClass<T>): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(this, clazz.java)
}
```

메서드 파라미터로 전달된 `T`의 `Class`는 readValue의 argument로 사용된다. 일반적인 제네릭 자바 코드와 같은 형태이고 올바르게 동작한다. 그리고 다음과 같이 사용할 수 있다.

```kotlin
data class Foo(val name: String)

val json = """{"name":"example"}"""
json.toKotlinObject(Foo::class)
```

## reified 사용하기
`reified` 타입 파라미터 `T`와 함께 `inline` 함수를 사용하면 같은 기능을 다른 방식으로 구현할 수 있다.

```kotlin
inline fun <reifiedd T : Any> String.toKotlinObject(): T {
    val mapper = jacksonObjectMapper()
    return mapper.readValue(this, T:class.java)
}
```

위 코드에서는 추가적으로 `T`의 `Class`를 받을 필요도 없고, `T`는 일반적인 클래스로 사용될 수 있다. 그리고 다음과 같이 사용할 수 있다.

```kotlin
json.toKotlinObject<Foo>()
```

## 주의
`reified` 타입 파라미터로 작성된 인라인 함수는 **자바 코드에서 호출 할 수 없다.**
