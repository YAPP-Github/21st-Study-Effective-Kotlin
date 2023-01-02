# 아이템 1 : 생성자 대신 팩토리를 활용하라

**생성자** - 클래스의 인스턴스를 만드는 가장 기본적인 방법 

```kotlin
class LinkedList<T>(
    val head: T,
    val tail: LinkedList<T>?
)

val list = LinkedList(
    head = 1,
    tail = LinkedList(2, null)
)
```

- 디자인 패턴 중, 다양한 생성 패턴이 존재함
- 일반적으로 객체를 직접 생성하지 않고, 별도의 함수를 통해 생성함

**팩토리 함수** - 생성자의 역할을 대신 해주는 함수

```kotlin
fun <T> linkedListOf(
    vararg elements: T
): LinkedList<T>? {
    if(elements.isEmpty()) return null
    val head = elements.first()
    val elementsTail = elements.copyOfRange(1, elements.size)
    val tail = linkedListOf(*elementsTail)
    return LinkedList(head, tail)
}

val list = linkedListOf(1, 2)
```

**생성자와 비교했을 때 여러 장점들이 존재함**

- **함수에 이름을 붙일 수 있음**
    - 객체가 생성되는 방법과 arguments로 필요한 요소를 설명함
    - 동일한 파라미터 타입을 갖는 생성자의 충돌을 줄임
    
- **함수가 원하는 형태의 타입을 리턴할 수 있음**
    - 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용하게 사용
    
    > 저희 예전에 `List<T>`에 대해서 얘기 나눴었는데, List를 사용해서 하는 여러 연산들이 `List 인터페이스`를 반환하는 이유에 대해 나오는데, 여기서 설명하고 있네요!
    > 
    
    ```kotlin
    public inline fun <T> listOf(): List<T> = emptyList()
    ```
    
    - 더 많은 자유도를 가질 수 있음
    
- **호출될 때 마다 새로운 객체를 만들 수 있음**
    - 싱글턴으로 호출하여 하나만 생성하게 강제
    - `createOrNull` 같은 메서드를 만들어  null을 리턴하게 할 수 있음
    
- **아직 존재하지 않는 객체를 리턴할 수 있음**
    - 어노테이션 처리를 기반으로 하는 라이브러리에서 팩토리 메서드를 많이 활용함
    
- **객체 외부에 팩토리 함수를 만들면, 가시성을 원하는 대로 제어할 수 있음**
    - 파일 또는 같은 모듈에서만 접근하게 만들어 관심사의 분리를 이룸

- **인라인으로 작성할 수 있으며, parameter를 refied로 만들 수 있음**

- **생성자로 만들기 복잡한 객체도 만들어 낼 수 있음(ex 맨 위 예제)**

- **원하는 때에 생성자를 호출할 수 있음**

```kotlin
inline fun <T> makeList(config: () -> Config) : ListView{
    val config = config()
    val items = //... 요소 읽어들이기
    return Listview(items)
}
```

### 고려할 사항

- **서브 클래스 생성에는 슈퍼 클래스의 생성자가 필요함**
    - 슈퍼 클래스를 만들려고 할 때, **서브 클래스에도** 만들어주면 됨
- **기본 생성자를 사용하지 말라는 것이 아님**
    - 애초에 팩토리 패턴에서 생성자를 사용함
    - 추가 생성자(secondary constructor)와 경쟁 관계임
    - 근데 보통 팩토리 함수를 많이 사용함
    

## 팩토리 함수의 종류

1. [companion 객체 팩토리 함수](https://www.notion.so/1-dbd24e2589ff4f56a7867e041183cd48)
2. [확장 팩토리 함수](https://www.notion.so/1-dbd24e2589ff4f56a7867e041183cd48)
3. [톱레벨 팩토리 함수](https://www.notion.so/1-dbd24e2589ff4f56a7867e041183cd48)
4. [가짜 생성자](https://www.notion.so/1-dbd24e2589ff4f56a7867e041183cd48)
5. [팩토리 클래스의 메서드](https://www.notion.so/1-dbd24e2589ff4f56a7867e041183cd48)

### Companion 객체 팩토리 함수

**가장 일반적인 방법**으로, Companion 객체를 사용하는 방법임

```kotlin
class LinkedList<T>(
    val head: T,
    val tail: LinkedList<T>?
){
    companion object{
        fun <T> of(vararg elements: T): LinkedList<T>? {
            if(elements.isEmpty()) return null
            val head = elements.first()
            val elementsTail = elements.copyOfRange(1, elements.size)
            val tail = linkedListOf(*elementsTail)
            return LinkedList(head, tail)
        }
    }
}
```

- 이름을 가진 생성자(Named Constructor Idiom) 이라고도 부름

**다양한 팩토리 함수 이름**

- `from` : 인자 받아서 반환
- `of` : 여러 개의 인자를 받아 통합 및 반환
- `valueOf` : 위 둘과 비슷하나, 좀 더 친절한 의미의 이름
- `instance` or `getInstance`  : 싱글턴
- `createInstance` or `newInstance` : 싱글턴 X
- `getType` : `getInstance` 와 비슷하나, 팩토리가 다른 클래스에 있을 때
- `newType` : `newInstance` 와 비슷하나, 팩토리가 다른 클래스에 있을 때

**`Companion` 객체의 더 많은 기능**

```kotlin
//**Companion 객체를 단순한 정적 멤버로 취급하지 말자.**
**//Companion 객체는 인터페이스를 구현할 수 있으며, 클래스를 상속받을 수도 있음**
abstract class ActivityFactory{
    abstract fun getIntent(context: Context): Intent
}

class MainActivity: ComponentActivity(){
    
    companion object: ActivityFactory(){
        override fun getIntent(context: Context) : Intent = Intent(context, MainActivity::class.java)
    }
}
```

### 확장 팩토리 함수

- **이미 `Companion` 객체가 존재할 때** 사용할 수 있는 팩토리 함수
- **확장 함수를 활용하여 구현**

```kotlin
interface Tool {
    companion object { }
}

fun Tool.Companion.createBigTool() : BigTool{
    
}

fun main(){ 
    Tool.createBigTool() 
}
```

- 이를 활용하여 외부 라이브러리를 확장할 수 있음
- 단, 적어도 비어있는 `Companion` 객체가 필요

### 톱 레벨 팩토리 함수

- ex) `listOf, setOf, mapOf`
- **광범위하게 사용됨**

```kotlin
class MainActivity: ComponentActivity(){
    
    companion object: ActivityFactory(){
        override fun getIntent(context: Context) : Intent = Intent(context, MainActivity::class.java)
    }
}
```

- `listOf`이 `List.of` 보다 **가독성이 좋기**에 톱 레벨 함수로 사용함
    - **모든 곳에서 사용할 수 있기 때문에, 이름을 잘 지정해야 하고 신중하게 사용해야 함**
    

### 가짜 생성자

생성자 처럼 보이고, 생성자 처럼 작동하지만 팩토리 함수의 장점을 가지는 함수

`MutableList`는 인터페이스인데, 어떻게 `MutableList(4) {}` 형식으로 사용할 수 있었을까?

```kotlin
public inline fun <T> MutableList(size: Int, init: (index: Int) -> T): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

클래스의 이름을 가진 톱 레벨 함수로 위 코드처럼 구현되어 있다. 

- 많은 개발자들이 톱 레벨 함수인지 잘 모름 - 가짜 생성자라고 부르는 이유

**가짜 생성자를 만드는 이유**

- **인터페이스를 위한 생성자를 만들고 싶을 때**(기본 생성자를 만들 수 없는 상황)
- reified 타입 argument를 갖고 싶을 때

를 제외하고는, 진짜 생성자처럼 동작해야 함

**또 다른 방법?**

```kotlin
class Tree<T>{
    companion object {
        operator fun <T> invoke(size: Int, generator: (Int) -> T): Tree<T>{
            
        }
    }
}

fun main(){
    Tree(10) { "it" }
}
```

> **다만, 연산자 오버로드를 사용할 경우 의미에 맞게 사용을 해야하는데, 이에 위배되므로 위 방법은 좋지 않은 방법임
따라서 톱 레벨 함수를 사용하는 것이 좋음**
> 

### 팩토리 클래스의 메서드

- 상태를 가질 수 있는 특성으로 인해 더 다양한 기능을 가짐

```kotlin
data class Student(
    val id: Int,
    val name: String,
)

class StudentFactory{
    private var nextId = 0
    fun next(name: String) = Student(nextId++, name)
}

fun main(){
    val factory = StudentFactory()
    val student1 = factory.next("rok")
    val student2 = factory.next("yoon")
}
```

- **캐싱을 활용하거나, 이전에 만든 객체를 복사하여 사용하는 등, 객체의 생성 속도를 높일 수 있음**

## 정리

- **객체를 생성할 때 각각의 팩토리 함수의 특징을 잘 파악하고 사용해야 함**
- **가짜 생성자, 톱레벨 팩토리, 확장 팩토리 함수는 신중하게 사용해야 함**
- **가장 일반적인 방법은 companion을 이용하는 것**
    - 가장 관습적이고 안전함
