# 아이템 7 : compareTo의 규약을 지켜라
**>, ≥, ≤ 등 부등식 연산은 compareTo로 치환된다.**

## Kotlin

```kotlin
data class Example(
    val a: Int,
    val b: List<Int>,
): Comparable<Example> {
    override fun compareTo(other: Example): Int {
        TODO("Not yet implemented")
    }
}

fun main(){
   val b = Example(1, listOf(1)) > Example(1, listOf(1))   
}
```

## Java

```java
public static final void main() {
   boolean b = (new Example(1, CollectionsKt.listOf(1))).compareTo(new Example(1, CollectionsKt.listOf(1))) > 0;
}
```

Comparable<T>를 구현하는 클래스 

→ compareTo를 구현한다

→ **해당 객체가 어떤 순서를 가지고, 비교할 수 있다** 

→ **사용자에 의해 동작이 정해지므로, 규약을 지켜야 한다.**

### compareTo에서 지켜져야 할 규약

- **비대칭적 동작**
    - `a ≥ b` 이고, `b ≥ a`라면 `a == b` 여야 한다.
    - 비교와 동등성 간에 어떠한 일관성 있는 관계가 있어야 한다.
- **연속적 동작**
    - `a ≥ b` 이고 `b ≥ c` 이면 `a ≥ c` 여야 한다.
    - 요소 정렬이 무한 반복에 빠지지 않도록 지켜져야 한다.
- **코넥스적 동작**
    - `a ≥ b` 또는 `b≥a` 중 적어도 하나는 항상 true여야 한다.
    - 두 요소는 확실히 어떤 관계를 가져야 한다.
    - 관계가 없다면, 퀵, 삽입 등의 정렬 알고리즘을 사용할 수 없다.

> 뭔 소린가 해서 검색해보니까, 수학 개념이네요. 요점은 compareTo가 사용자에 의해 정해지니, 순서를 비교할 수 있도록 관계 정의가 잘 되어야 하는 것이 요점 같습니다.
> 

## compareTo를 따로 정의해야 할까?

- **프로퍼티 한개 기준 정렬**
    - Comparable을 따로 구현하지 않고, 아래 메서드를 사용하면 된다.
    - `sortedBy{ property }` ,
- **여러개 기준 정렬**
    - `sortedWith(compareBy({ propery1 }, { property2 })`
    - Comparable을 정의하고 sort
- **String**
    - 내부적으로 Comparable을 구현하고 있음
    - 단점은 **가독성이 조금 떨어짐**
- **날짜, 시간 등 자연스러운 순서를 갖지 않고 애매한 경우라면**
    - 비교기를 companion으로 정의하는 방법 괜찮다
    
    ```kotlin
    class User(
        val name: String,
        val surname: String
    ){
        companion object{
            val DisplayOrder = compareBy(User::surname, User::name)
        }
    }
    ```
    

### compareTo 구현하기

```kotlin
class User(
    val name: String,
    val surName: String
) : Comparable<User> {
    override fun compareTo(other: User): Int = compareValues(surName, other.surName)
}
```

```kotlin
class User(
    val name: String,
    val surName: String
) : Comparable<User> {
    override fun compareTo(other: User): Int = compareValuesBy(this, other, { it.surName }, { it.surName })
}
```

- **Comparator,를 구현하게 된다면 `compareValues`, `compareValuesBy` 를 활용하여 봅시다.**
- **구현 후, 규약을 지키는지 확인하는 것도 잊지 말자!**
