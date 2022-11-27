# item 6: 프로퍼티는 동작이 아니라 상태를 나타내야 한다.

## 개요

### 프로퍼티, 필드 (loughly)

프로퍼티와 필드의 공통점: 데이터를 저장하는데 **사용될 수 있음** 

- 번역판에 그냥 “저장한다”라고 오역되어 있어서, 블로그에 무분별하게 전파중
- 프로퍼티는 “속성” 이다.

프로퍼티는 데이터를 저장하는 backing field에 대한 레퍼런스인 field를 가질 수 있음

var 을 이용하면 프로퍼티는 게터와 세터를 만들어 파생 프로퍼티(derived property)라고 하는데 별로 중요한 내용은 아닌 것 같다. 공식 문서에 없는 용어이고 책에만 나오는 내용 

### backing field

필요하면 코틀린이 자동으로 만들어준다. 

필요 없으면 안 만들어진다

기본 Accessor 들이 field를 사용하기 때문에, 보통은 backing field가 존재한다.

물론 custom accessor가 field를 사용해도 backing field가 존재한다.

프로퍼티가 데이터를 저장하지 않으면 당연히 backing field도 필요 없고 만들어지지도 않는다. 

```kotlin
class User(val age = 0){
	val isAdult: Boolean
		get() this.age >= 19  // 저장할 값이 없어서 필드가 필요 없음
	val name = "sejin" // 저장할 값이 있어서 필드가 필요함 
}

참고: backing properties
private val _liveData1 = LiveData<Some>() // 필요해서 백킹 프로퍼티 만듬
val liveData1: MutableLiveData<Some> = _liveData1

private val viewModelAge: Int = 0  // 필요 없어서 백킹 프로퍼티 안만듬

```

backing field: 필요하면 **코틀린이** 만들어준다

backing property: 필요하면 **사람이** 선언한다. 

### 프로퍼티의 캡슐화와 활용

프로퍼티는 본질이 Accessor라서 기본적으로 캡슐화 되어있다.

캡슐화 자체에 의미를 둔다기 보다는,  Accessor를 사용하므로 (캡슐화가 되므로) 이를 유연하게 사용해 간단하게 다양한 상황에 대처할 수 있다고 알면 되겠습니다.

프로젝트에서 Date 클래스를 적극 사용하고 있던 도중, 더 이상 Date클래스에 시간 정보를 저장하고 싶지 않다고 가정합시다.

```kotlin

class SomeClass {
		var someVariable: Int = 0
    var millis: Long = 0
        get() = date.time
        private set

    var date: Date
        get() = Date(millis) // Date를 원한다면 매번 생성해줍니다.
        set(value) {
            millis = value.time
        }
}
```

프로퍼티는 필드가 (데이터가) 필요 없기 때문에 interface에도 정의할 수 있고, 확장 프로퍼티도 만들 수 있음 

따라서 프로퍼티 위임도 자연스럽게 Accessor를 만드는 동작으로 이해할 수 있음

그냥 특수한 방식으로 getter 를 만들어주는 것

```kotlin
val db: Database by lazy {connectToDb()}
---
private final Lazy db$delegate;
public 생 성 자 (){
	this.db$delegate = LazyKt.lazy(~~~)
}
public Database getDb(){
	val a = this.db$delegate
	return a.value()
}
```

### When to / not to use properties

프로퍼티를 함수의 대용으로 사용할 수는 있지만, 완전히 대체해서  사용하면 안된다. 

상태를 나타내거나 설정하기 위한 목적으로만 사용해야한다. 

getter나 setter로 기능하지 않는 경우 프로퍼티로 만드는 것은 좋지 않다.

함수의 대용으로 사용하면 안되는 경우 

1. O(1) 이상의 복잡도 이거나 기타 연산 비용이 많은 경우. 
    - 복잡한 연산은 관습적으로 함수가 처리하고, 따라서 다른 개발자의 최적화를 기대할 수도 있음
    - 프로퍼티를 호출했을 때 무거운 연산이 실행될 것을 아무도 기대하지 않음
2. 비즈니스 로직을 표현하는 경우 
    - 프로퍼티가 로깅, notify, 바인딩 값 바꾸기 정도 이상은 기대하지 않는다.
3. 결정적이지 않은 경우 (not deterministic)
    - 프로퍼티는 호출 결과가 deterministic 해야함. 외부로부터의 변경이 없는 한, 같은 동작 혹은 연속적인 동작에도 동일한 값이 나와야 한다.
    - 이런 경우 Iterator.next() 등의 함수로 구현되어 있음.
4. conversion
    - 관습적으로 Int.toDouble() 같은 함수를 사용한다.
    - 다른 state에 대한 reference로 오해할 수 있음
5. 게터에서 프로퍼티 상태 변경이 일어나는 경우

프로퍼티를 써야하는 경우

- java에서처럼 getName, setName을 함수로 설정하는 경우, 프로퍼티로 고친다.