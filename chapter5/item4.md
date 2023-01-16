# 태그 클래스보다는 클래스 계층을 사용하라.

- 클래스에서 상수 ‘모드’를 가진 클래스들이 존재
- 여기서 상수를 태그라고도 하고, 상수 ‘모드’를 가진 클래스를 태그 클래스라고 함.

```kotlin
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
){
    fun match(value: T?) = when(matcher){
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
    }

    enum class Matcher{
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }

    companion object{
        fun <T> equal(value: T) =
            ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)

        fun <T> notEuqal(value: T) =
            ValueMatcher<T>(value = value, matcher = Matcher.NOT_EQUAL)

        fun <T> emptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)

        fun <T> notEmptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
    }
}

fun main(){
    val matcher = ValueMatcher.notEmptyList<List<Int>>()
    println(matcher.match(listOf(1,2,3)))
}
```

- 한 클래스에 여러 모드를 처리하기 위한 보일러 플레이트 코드 존재
- 여러 목적 사용으로 인해 사용되지 않은 프로퍼티 존재할 수도 있음.
    - 위의 예제에서는 value 값이 LIST_EMPTY와 LIST_NOT_EMPTY 모드에는 존재하지 않음
- 상태의 일관성과 정확성 지키기 어려움.

<aside>
💡 따라서 코틀린에서는 한 클래스에 여러 모드를 만드는 방법 대신, 각각의 모드를 클래스로 만들고 타입시스템과 다형성을 활용 → sealed class

</aside>

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean
    
    class Equal<T>(val value: T) : ValueMatcher<T>(){
        override fun match(value: T): Boolean {
            return value == this.value
        }
    }
    
    class NotEquals<T>(val value: T): ValueMatcher<T>(){
        override fun match(value: T): Boolean {
            return value != this.value
        }
    }
    
    class EmptyList<T>() : ValueMatcher<T>(){
        override fun match(value: T): Boolean {
            return value is List<*> && value.isEmpty()
        }
    }
    
    class NotEmptyList<T>(): ValueMatcher<T>(){
        override fun match(value: T): Boolean {
            return value is List<*> && value.isNotEmpty()
        }
    }
}
```

- 태그 클래스가 아닌 sealed 클래스로 구현을 하게 되면 책임이 분산되고 깔끔
- 또한, 각각 클래스에서 필요한 데이터만 있으면 가능하기에 적절한 파라미터만 존재

## sealed 한정자

- sealed 한정자가 필수는 아니지만 사용하게 되면 외부에서 추가적인 서브클래스를 만들 수 없으므로, 타입이 추가되지 않을 거라는게 보장됨.
- 이로 인해 when 구문에 모든 서브타입들에 대한 정의를 하였으면 else 블록 불필요