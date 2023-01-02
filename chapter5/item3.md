## 아이템 3: 복잡한 객체를 생성하기 위한 DSL을 정의하라

`DSL : Domain Specific Language`

- 복잡한 객체, 계층 구조를 갖고 있는 객체를 정의할 때 유용
- 설계는 어렵지만, **보일러 플레이트와 복잡성을 숨길 수 있어 개발자의 의도를 명확히 표현함**

Ktor의 예

```kotlin
HttpClient(engineFactory = CIO) {
        expectSuccess = true
        engine {
            endpoint {
                connectTimeout = MaxTimeoutMillis
                connectAttempts = MaxRetryCount
            }
        }
        defaultRequest {
            url(BaseUrl)
            headers {
                append(HttpHeaders.ContentType, ContentType.Application.Json)
                append("x-duckie-device-name", Build.MODEL)
                append("x-duckie-version", BuildConfig.APP_VERSION_NAME)
                append("x-duckie-client", "android")
            }
        }
        install(plugin = DuckieAuthorizationHeaderOrNothingPlugin)
        install(plugin = ContentNegotiation) {
            jackson()
        }
        install(plugin = Logging) {
            logger = Logger.ANDROID
            level = LogLevel.ALL
        }
```

- engineFactory에 CIO을 제공하면, 그에 맞는 DSL 블럭을 사용할 수 있음
- install에 제공하는 plugin이 달라지면, 각각에 맞는 세팅을 할 수 있음
- **직관적이고 명확함**

## 사용자 정의 DSL 만들기

1. **함수 타입이란?**
    
    ```kotlin
    public inline fun <T, C : MutableCollection<in T>> Iterable<T>.filterTo(destination: C, predicate: (T) -> Boolean): C {
        for (element in this) if (predicate(element)) destination.add(element)
        return destination
    }
    ```
    
    - `predicate: (T) -> Boolean` 은 함수타입으로, **함수 타입은 함수로 사용할(나타낼) 수 있는 객체를 의미함**
        - **람다**, **익명 함수**, **함수 레퍼런스**로 만들 수 있음
    
2. **리시버를 나타내는 함수 타입**
    
    ```kotlin
    val myPlus: Int.(Int)->Int = fun Int.(other: Int) = this + other
    ```
    
    - `Int.(Int) -> Int` 는 리시버를 가진 함수 타입으로, **파라미터 앞에 리시버 타입이 추가되어 있음**
    - 호출 방법
    
    ```kotlin
    myPlus.invoke(1,2)
    myPlus(1,2)
    1.myPlus(2)
    ```
    
    - **this의 참조 대상을 변경할 수 있음**
    - **DSL을 구성하는 가장 기본적인 블럭**
    
3. **HTML를 Table을 표현하는 DSL 만들어보기**

```kotlin
fun createTable(): TableDsl = table {
		tr {
        for(i in 1..2) {
            td {
                + "This is column $i"
            }
				}
    }
}
```

- **현재 코드가 Top-level에 위치하고, 별도의 리시버를 갖지 않으므로 table함수도 top-levelp에 있어야함**
- **tr 함수는 table 정의 내부에서만 허용되어야 함**
    - table 함수의 argument는 tr 함수를 갖는 리시버를 가져야 함
    - td도 마찬가지

### 구현

- 각각의 단계에서 빌더를 만들고, 파라미터를 활용하여 적절히 초기화 하여 구현
- 빌더를 리턴하거나, 데이터를 보유한 다른 객체를 만들어 저장하여 사용

```kotlin
fun table(init: TableBuilder.() -> Unit): TableBuilder = TableBuilder
   = TableBuilder().apply(init)

class TableBuilder {
		var trs = listOf<TrBuilder>()

		fun tr(init: TrBuilder.() -> Unit) { 
				trs = trs + TrBuilder().apply(init) 
		} 
}

class TrBuilder {
		var text = ""
		var tds = listOf<TdBuilder>()
		
    fun td(init: TdBuilder.() -> Unit) {
		    tds = tds + TdBuilder().apply(init)
    } 

		operator fun String.unaryPlus() {
        text += this
    }
}
```

### 언제 사용하는가?

- **복잡한 자료 구조**
- **계층적인 구조**
- **거대한 양의 데이터**

## 정리

DSL은 익숙하지 않은 개발자에게 혼란과 어려움을 줄 수 있음

**복잡한 객체, 계층 구조를 만들 때에만 활용하는 것이 좋음**
