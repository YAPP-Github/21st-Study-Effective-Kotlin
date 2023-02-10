# 아이템 3 : 인라인 클래스의 사용을 고려하라

## Inline Class

```kotlin
@JvmInline
value class Password(private val s: String)
```

- 다른 변수와 혼동되지 않기 위해 값을 래핑할 수 있는 클래스
- value로 선언함으로써 컴파일러가 객체를 생성하지 않고 값을 매핑해 줌

`@JvmInline` 어노테이션과 `value` 키워드를 붙이면 value(inline) 클래스를 선언할 수 있음

코틀린의 1.3 버전에서는 inline class가 alpha 버전으로 사용되었는데, 1.5.0 이후부터는 `value`와 `@JvmInline`을 붙여서 사용하도록 권장하고 있습니다.

> @JvmInline 은 KotlinJs, KotlinNative 등 다른 버전과 호환을 위해 존재하는 어노테이션
> 

## 왜 인라인 클래스를 사용하는가?

- **비즈니스 로직**을 위해 특정 type을 **Wrapper**로 감싸줘야 할 때가 있는데, 이 때 **객체를 생성하기 위해 힙 할당을 하게 됨으로써 런타임 오버헤드가 발생**하고 성능 저하가 일어나기 때문에
- 측정 단위를 표현할 때

이런 것들을 해결하기 위해 **inline 클래스**를 사용할 수 있습니다.

```kotlin
fun alarm(millis: Long){
	print("$millis 밀리 후에 알람이 울립니다.")
}

fun test(){
	alarm(2000L)
}

```

여기서 2000L이라는 parameter를 넣어 호출한다면?

별 문제 없어 보이는 코드일 수 있습니다. 하지만 `2000ms`가 2초라는 사실을 잘 모르는 경우도 있을 것이고 , **다른 비즈니스 로직을 적용하더라도 소수만 알아 들을 수 있는 어떠한 값이 있을 것입니다.**

이러한 문제를 Wrapper Class를 작성함으로써 해결할 수 있습니다.

```kotlin
fun alarm(duration: Duration){
		print("{$duration.millis} 밀리 후에 알람이 울립니다.")
}

fun test(){
		alarm(Duration.seconds(2))
}

data class Duration private constructor(
        val millis: Long,
) {
    companion object {
        fun millis(millis: Long) = Duration(millis)
        fun seconds(seconds: Long) = Duration(seconds * 1000)
    }
}

```

이런 식으로 `Duration class`를 통해 2초라는 사실을 바로 확인할 수 있습니다.

**그러나**, `data class`를 사용한다면 **매번 alarm을 호출할 때 마다, Duration이라는 객체를 생성하여 할당해 줘야 합니다.**

추가 힙 할당으로 생기는 런타임 오버헤드를 방지하기 위해, **value**라는 modifier를 앞에 붙여주어 inline class를 선언해 보겠습니다.

```kotlin
fun alarm(duration: Duration){
		print("{$duration.millis} 밀리 후에 알람이 울립니다.")
}

fun test(){
		alarm(Duration.seconds(2))
}

@JvmInline
value class Duration private constructor(
        val millis: Long,
) {
    companion object {
        fun millis(millis: Long) = Duration(millis)
        fun seconds(seconds: Long) = Duration(seconds * 1000)
    }
}

```

value class로 선언된 Duration은, 파라미터로 들어올 때, **객체의 프로퍼티로 대체됩니다.**

- 위 코드를 아래와 같이 인터페이스로 선언할 수도 있는데, inline의 장점이 사라져서 쓸모 없게 되어버림
    ![Untitled 1](https://user-images.githubusercontent.com/70064912/218126498-872ebcb2-c4bc-411f-80fd-2bd0c0bad44d.png)

자바로 디컴파일 하면서 무슨일이 일어나고 있는지 확인해 보겠습니다.

### data class

```
public static final void alarm(@NotNull Duration duration) {
      Intrinsics.checkNotNullParameter(duration, "duration");
      String var1 = duration.getMillis() + " 밀리 후에 알람이 울립니다.";
      System.out.print(var1);
}

```

자바로 디컴파일 했을 때, `data class`로 선언된 `Duration` 의 경우, 객체 자체를 받아서 객체 내의 프로퍼티를 가져오고 있습니다.

### value class

```
public static final void alarm_ZlhV6Ys(long duration) {
      String var2 = duration + " 밀리 후에 알람이 울립니다.";
      System.out.print(var2);
   }

```

`fun alarm(duration: Duration)` 분명히 코틀린 코드는 `Duration`을 받고 있으나, 자바로 디컴파일 했을 때 위와 같이 `Duration` 내의 프로퍼티를 가져오고 있습니다.

여기서 디컴파일 시 **`_ZlhV6TYs` 처럼 특정한 문자열을 붙이는 것을 맹글링이라고 합니다.**

```
@JvmInline
value class People(val name: String){
	fun f(people: People) {}
	fun f(name: String) {}
}

```

위와 같이 메서드 오버로딩을 적용했을 때, 맹글링이 적용되지 않는다면 디컴파일 하였을 때 메서드 이름과 parameter가 중복이 됩니다.

이처럼, 컴파일러가 자동으로 최적화를 진행해 주기 때문에, 객체를 할당하지 않아도 Wrapper 클래스를 통해 가독성을 높일 수 있습니다.

## value class의 특징

- **프로퍼티를 하나만 가질 수 있음**

![https://velog.velcdn.com/images/evergreen_tree/post/d4c9a7bc-bfe3-46eb-8dd4-e5a9148dcd16/image.png](https://velog.velcdn.com/images/evergreen_tree/post/d4c9a7bc-bfe3-46eb-8dd4-e5a9148dcd16/image.png)

- `equals()`, `toString()`, `hashCode()` 메서드만 생성(data class는 `copy()`, `componentN()` 까지 생성)
- === 연산을 지원하지 않음
- val 프로퍼티만 허용

## typealias 대신에 inline class를 활용하자

- 아래처럼, Millis에 Seconds를 넣어도 문제되지 않음
![Untitled](https://user-images.githubusercontent.com/70064912/218126531-25afdd66-4688-42d0-b0db-b2d1ca97def7.png)


## 정리

- 객체 생성 오버헤드 없이 타입 래핑 가능
- 휴먼 에러를 방지하고, 코드 안정성을 향상시켜줌
- 의미가 명확하지 않는 타입, 측정 단위들에 사용하자
