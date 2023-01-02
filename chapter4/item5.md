# 아이템 5 :요소의 가시성을 최소화하라

### 왜 가시성을 최소화해야 하는가?

- **변경을 가할 때 기존의 것을 숨기는 것 보다 새로운 것을 노출하는 것이 쉽다.**
    - 공개되어있는 경우 이미 **외부에서 사용중** 이기에, 관련된 제한을 변경하는 것이 어려움
- **클래스의 상태를 나타내는 프로퍼티를 보장할 수 없음**
    - 클래스 상태에 대한 **규약을 모르는 사람이 상태를 변경하여, 클래스의 불변성을 해침**
    - 아래와 같이 setter만을 private으로 설정하는 방식으로 **프로퍼티를 캡슐화** 하는 방식을 사용함
    
    ```kotlin
    class CounterSet<T>(
        private val innerSet: MutableSet<T> = mutableSetOf()
    ) : MutableSet<T> by innerSet {
        var elementsAdded: Int = 0
            private set
    
        override fun add(element: T): Boolean {
            elementsAdded++
            return innerSet.add(element)
        }
    
        override fun addAll(elements: Collection<T>): Boolean {
            elementsAdded += elements.size
            return innerSet.addAll(elements)
        }
    }
    ```
    
    - 의존하는 프로퍼티가 있을 경우, 객체 상태를 보호하는 것이 더 중요함
        - 아래와 같이 value와 initialized가 의존하는 경우, initialized가 노출된다면 **예상하지 못한 변경에 의해 예외가 발생하고, 코드 신뢰성이 떨어질 수 있음**
    
    ```kotlin
    class MutableLazyHolder<T>(val initializer: () -> T) {
        private var value: Any? = Any()
        private var initialized = false
    
        @Suppress("UNCHECKED_CAST")
        operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
            if (!initalized) {
                value = initializer()
                initialized = true
            }
            return value as T
        }
    
        operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
            this.value = value
            initialized = true
        }
    }
    ```
    
- 상태 변경을 막는 것은 병렬 프로그래밍을 할 때 안전성을 보장함

## 가시성 한정자 사용하기

클래스와 요소를 외부에 노출할 필요가 없다면, 가시성을 제한하여 외부에서 접근할 수 없도록 만들어야 함

### 클래스 멤버

- **public(default) :** 어디에서나 볼 수 있음
- **private :** 클래스 내부에서만 볼 수 있음
- **protected :** 클래스와 서브클래스 내부에서만 볼 수 있음
- **internal** : 모듈 내부에서만 볼 수 있음

### **톱 레벨 요소**

- **public** : 어디에서나 볼 수 있음
- **private** : 같은 파일 내부에서만 볼 수 있음 (동일한 파일 또는 클래스 에서만 사용할 때 사용)
- **internal** : 모듈 내부에서만 볼 수 있음 (다른 모듈에 의해 사용될 가능성이 있을 때 사용)

<br>

**모듈과 패키지를 혼동하지 말자**
```kotlin
모듈 : 함께 컴파일 되는 코틀린 소스 ex) 그레이들 소스 세트, 메이븐 프로젝트, 인텔리제이 IDEA 모듈
```

### data class에는 적용하지 않는 것이 좋음

- 프로퍼티가 노출되어야 하므로, **public으로 지정**하는 것이 좋음

```kotlin
class User(
    val name: String,
    val age: Int,
)
```

## 정리

요소의 가시성은 최대한 제한적인 것이 좋다.

- **인터페이스가 작을수록 이를 공부하고 유지하는 것이 쉬움**
- **최대한 제한이 되어있어야 변경하기 쉬움**
- 클**래스의 상태를 나타내는 프로퍼티가 노출되어 있다면, 클래스가 자신의 상태를 책임질 수 없음**
- **가시성이 제한되면 API 변경을 쉽게 추적할 수 있음**
