# item 7: 이름 있는 아규먼트를 사용하라

이름 있는 아규먼트의 장점

1. 이름을 기반으로 값이 무엇을 나타내는지 알 수 있음 → 가독성
2. 파라미터 입력 순서와 상관 없으므로 안전함 → 안전성

worst

- 아규먼트의 의미가 명확하지 않은 경우
- 이 함수를 처음 본 사람은 separator로 생각하지 않을 수도 있음

```jsx
val text = (1..10).joinToString("|")
```

normal

- 변수 이름을 사용하는 경우
- 의도를 알아챌 수는 있지만, 변수 자체를 잘못 사용할수도 있음

```jsx
val separator = '|'
val text = (1..10).joinToString(separator)
```

better

- **이름 있는 아규먼트**를 사용했을 경우
- 잘못된 사용을 방지함

```jsx
val text = (1..10).joinToString(separator = "|")
```

- 변수와 “이름 있는 아규먼트”를 동시에 사용하는 경우

```jsx

val separator = '|'
val text = (1..10).joinToString(separator = separator)
```

### 이름 있는 아규먼트 사용례

1. 디폴트 아규먼트의 경우(optional로서 기본값을 갖는 것)
    - 함수 이름은 필수 파라미터와 관련 → 함수 이름만 보더라도 필수 파라미터의 역할을 파악할 수 있다.
    - 그만큼 optional 한 파라미터에 대한 정보가 부족하므로, 이름 있는 아규먼트를 사용하자.
2. 같은 타입의 파라미터가 많은 경우
    - 파라미터가 모두 다른 타입이면, 위치를 잘못 입력해도 컴파일러가 오류를 발생시켜줌
    - 같은 타입이라면 찾아내기 어려울 수 있음
    
    ```kotlin
    fun sendEmail(to: String, msg: String)
    sendEmail (to = "sejin", msg ="hello")
    ```
    
3. 함수 타입의 파라미터가 있는 경우
    - 함수형 파라미터가 마지막에 붙는 경우는, context에 따라서 처리
    
    bad case
    
    ```kotlin
    fun call(before: ()->Unit ={}, after: ()->Unit = {}){
    	before()
    	println("Middle")
    	after()
    }
    
    call({print("CALL")}) // before
    call{print("CALL")} // after
    
    ```
    
    이렇게 쓰세요
    
    ```kotlin
    call(before = {println("before"}){
    	println("after")
    }
    call(before = {println("before"}, after = {println("after")})
    
    thread{
    
    }
    repeat(N){
    
    }
    
    getUserRx(
    		authorization = auth,
        user_id = userId,
        success = { user ->
            asdf.value = user
    	  },
        error = { t -> Log.d("getTargetUser ", t.toString()) }
    )
    
    ```
    
    RxJava를 코틀린에서 사용할 경우,  named argument를 지원하지 않는다. 이런 경우 named argument를 도입할 수 있도록 별도의 함수를 정의해서 사용한다. 
    
    - Java코드를 코틀린에서 사용시 named argument가 지원되지 않음!
    
    정리
    
    1. 디폴트 값을 설정하는 것 외에도 가독성, 안정성 향상
    2. 같은 타입 아규먼트가 많을 때, 옵셔널 아규먼트가 있을 때, 함수 타입 아규먼트를 넘길 때 많이 사용
    3. 마지막 파라미터가 특수한 의미를 갖는 경우는 고려하지 않아도 된다. (thread, repeat 등 )