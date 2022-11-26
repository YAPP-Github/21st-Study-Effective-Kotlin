# 2. 연산자를 오버로드를 할 때는 의미에 맞게 사용하라
> &nbsp; 연산자 오버로딩을 사용하면 강력한 도구로 활용할 수 있지만 때로는 중복된 의미로 인해 오용될 가능성도 존재합니다. 

```kotlin
    fun Int.factorial(): Int = (1..this).product()
    
    fun Iterable<Int>.product(): Int = 
    	fold(1) { acc, i -> acc * i }
        
    operator fun Int.not() = factorial()
```
팩토리얼을 ``! `` 연산자로 나타내기 위해서 not 연산자를 오버로딩하였습니다. 기능 동작을 할지라도 기존 사용자에게는 논리 연산으로 사용될지 새로 정의한 연산자로 사용될지 혼란스럽고 오해의 소지가 있습니다.
<br>

```kotlin
    operator fun Int.times(operation: () -> Unit): () -> Unit =
    	{ repeat(this) { operation() } }
        
    val tripledHello = 3 * { print("Hello") }
    
    tripleHello()	// HelloHelloHello
```
또한 대략적으로 유추는 가능하지만 확실하지 않을 경우가 있습니다. 위 예시는 times 연산자를 오버로드했고 특정 사용자는 함수를 세 번 반복하여 호출한다고 생각할 수도 있고 아닐 수도 있습니다. 이처럼 의미가 명확하지 않은 경우에는 ``infix``를 활용한 확장 함수를 사용하는 것이 좋습니다.
<br>

```kotlin
    infix fun Int.timesRepeated(operation: () -> Unit): () -> Unit =
    	{ repeat(this) { operation() } }
        
    val tripledHello = 3 timesRepeated { print("Hello") }
    
    tripleHello()	// HelloHelloHello
```
infix를 사용하여 이항 연산자처럼 사용할 수 있으며, 사실 가장 좋은 것은 top-level function을 사용하는 것이 좋습니다. n번 호출하는 함수에는 ``repeat(3) { } `` 가 이미 stdlib에 구현되어 있습니다.

> 연산자를 오버로딩을 할 때는 의미에 맞게 사용하되 의미가 모호할 경우에는 infix를 활용하라고 하였습니다. 하지만 DSL(Domain-Specific Language)를 정의할 때는 규칙을 무시해도 괜찮습니다. 😎
