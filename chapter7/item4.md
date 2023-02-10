# 아이템 4 : 더 이상 사용하지 않는 객체의 레퍼런스를 제거하라

- 일반적으로 스코프를 벗어나면서 객체가 자동으로 해제된다. 이를 위해 변수를 지역 스코프에 잘 정의하고, 톱레벨 프로퍼티나 컴패니언으로 큰 데이터를 저장하지 않는 정도는 잘 지켜야 한다.
- 그래도 Garbage Collector를 너무 맹신하지 말고, 어떤 객체의 메모리가 많이 차지하거나 많이 생성될 때에는 신경을 쓰자

## 메모리 누수의 예

- 액티비티 객체에 대한 참조를 companion으로 유지하면, 가비지 컬렉터가 해당 액티비티의 메모리를 회수 할 수 없게됨
- 배열에서 사용하지 않는 요소를 해제하지 않는 경우
- 초기화를 위한 initialLized 람다를 프로퍼티로 갖을 때, 초기화 이후 해제하지 않는 경우
    - 코틀린 stdlib의 lazy 예 → 초기화 이후 initialzier를 null로 변경하여 참조를 없앰
        
        ```kotlin
        private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
            private var initializer: (() -> T)? = initializer
            @Volatile private var _value: Any? = UNINITIALIZED_VALUE
            // final field is required to enable safe publication of constructed instance
            private val lock = lock ?: this
        
            override val value: T
                get() {
                    val _v1 = _value
                    if (_v1 !== UNINITIALIZED_VALUE) {
                        @Suppress("UNCHECKED_CAST")
                        return _v1 as T
                    }
        
                    returnsynchronized(lock){
        val _v2 = _value
                        if (_v2 !== UNINITIALIZED_VALUE) {
                            @Suppress("UNCHECKED_CAST") (_v2 as T)
                        } else {
                            val typedValue = initializer!!()
                            _value = typedValue
                            initializer = null
                            typedValue
                        }
        }
        }
        
            override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE
        
            override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."
        
            private fun writeReplace(): Any = InitializedLazyImpl(value)
        }
        
        ```
        
- 절대 사용되지 않는 객체를 캐시해서 저장해 두는 경우
    - `SoftReference` 를 사용하여, 객체가 부족한 경우 가비지 컬렉터가 이를 알아서 해제하도록 해야함
- 다이얼로그 같은 경우
    - `WeakReference` 를 사용하여 **다이얼로그가 출력되는 동안**만 메모리를 유지하고, 이후 메모리에 대한 참조를 유지할 필요가 없도록 설정

## 메모리 누수 예측하기

- 힙 프로파일러
- LeakCanary

## 클린코드와 성능의 트레이드오프?

- 일반적으로 클린코드를 잘 지킨경우, 메모리와 성능적으로 좋음
- 만약 기회비용을 계산해야 한다면?
    - 일반적인 경우 - **가독성과 확장성**
    - 라이브러리 - **메모리와 성능**
