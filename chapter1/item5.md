# 아이템 5 : 예외를 활용해 코드에 제한을 걸어라

확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어두는 것이 좋음

### 장점

- 일종의 **문서화**가 됨
- 함수가 **예상하지 못한 동작을 하지 않고** 예외를 throw
    - 문제 디버깅이 가능하고, 안정적으로 작동함
- **코드 검사**
- **캐스팅을 적게** 할 수 있음

### 예외를 활용하는 방법

- `require` 블록 : **Argument를 제한**함
- `check` 블록 : **상태와 관련된 동작을 제한**함
- `assert` 블록 : **어떤 것이 true인지 확인**할 수 있음(테스트 모드에서만 동작)
- `return, throw`와 함께 활용하는 Elvis 연산자

### 아규먼트에 제한을 걸 때

- **`require` 함수 사용**
- ex) 숫자를 argument로 받아 팩토리얼을 계산할 때, 양의 정수임을 판단해야 하는 경우
    
    ```kotlin
    fun factorial(n: Int): Long {
        require(n <= 0)
        return if (n <= 1) 1 else factorial(n - 1) * n
    }
    ```
    
- 유효성 검사 코드가 가장 앞부분에 배치되어, **가독성이 좋음**
- `IllegalArgumentException`을 발생시킴
- require 사용했던 예
    
    ```kotlin
    data class Feed(
        @PK val id: String,
        ...
    ) {
        init {
    	requireSize(
                min = 1,
                field = "categories",
                value = categories,
            )
    }
    
    internal fun requireSize(
        min: Int = 0,
        max: Int = Int.MAX_VALUE,
        field: String,
        value: Collection<*>,
    ) {
        require(
            value = value.size >= min,
        ) {
            "$field 의 최소 사이즈는 $min 입니다."
        }
        require(
            value = value.size <= max,
        ) {
            "$field 의 사이즈가 $max 보다 큽니다."
        }
    }
    ```
    

### 상태에 제한을 걸 때

- **check 함수 사용**
    - 상태가 올바른지 확인할 때 쓰임
    - ex) `speak`라는 함수를 사용할 때, `isInitialized`가 `true`일 때만 사용하고 싶을 때
    
    ```kotlin
    class People{
        var isInitialized : Boolean = false
        
        fun speak(text: String){
            check(isInitialized)
            print(text)
        }
    }
    ```
    
    - 사용자가 규약을 어기고, 사용하면 안될 곳에서 호출이 의심될 때 사용
    
- **assert 함수 사용**
    - **예상되는 결과가 나오는지 확인할 수 있는 테스트성  코드**
    - 단위 테스트 대신에 **assert**를 사용했을 때 얻을 수 얻는 이점
        - 코드를 자체 점검하고, 효율적으로 테스트가 가능
        - **모든 상황**에 대한 테스트가 가능
        - 실행 시점에 정확하게 어떻게 되는지 파악 가능
- assert 사용한 예제
-   ![Untitled](https://user-images.githubusercontent.com/70064912/201723287-8fefa40d-02c0-4582-93fa-abbd3328e5ef.png)
    ![Untitled 1](https://user-images.githubusercontent.com/70064912/201723273-8654aafc-8945-4af9-bd2c-0a72129ceac3.png)
    ![Untitled 2](https://user-images.githubusercontent.com/70064912/201723281-cb58e25b-6848-4f26-98b6-a54c04b9a047.png)

### nullability와 스마트 캐스팅

- require와 check 블록으로 어떤 조건을 확인하여 true가 나왔다면 **스마트캐스트**가 작동됨
    - 변수의 **unpack** 용도로 활용
- **nullabillity**일 때, 오른쪽에 throw, return을 두고 Elvis 연산자를 활용하면 **가독성이 좋고, 유연하게 사용 가능함**
    
    ```kotlin
    fun speak(text: String?){
            val text: String = text ?: return
        }
    
    fun speak(text: String?){
            val text: String = text ?: run{
    						log.d("TAG", "no text")
    						return
    				}
        }
    ```
    

### 정리

- 제한을 훨씬 더 쉽게 확인할 수 있음
- 애플리케이션을 더 안정적으로 지킬 수 있음
- 코드를 잘못 쓰는 상황을 막을 수 있음
- 스마트 캐스팅을 활용할 수 있음
