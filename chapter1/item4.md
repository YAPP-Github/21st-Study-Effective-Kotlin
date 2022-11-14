# 아이템 4 : inferred 타입으로 리턴하지 말라

### 타입 추론(type inference)

- **타입**을 명시하지 않아도, 변수가 선언될 때 할당된 값의 형태로 어떤 자료형을 가지는지 **추론**해 주는 기능
- 할당되는 피연산자에 맞게 타입이 설정됨

### **주의할 점**

- **조작할 수 없는 라이브러리를 사용할 때, 형변환 문제를 조심해야함**
    
    ```kotlin
    interface CarFactory{
        fun produce() = defaultCar
    }
    
    val defaultCar: Car = Fiat126P("normal")
    
    open class Car(
       val engine: String,
    )
    
    class Fiat126P(engine: String): Car(engine)
    ```
    
    - 위 상황에서, 다른 개발자에 의해 `Car`이라는 Type을 지워버리면, `Fiat126P`로 타입이 한정되어 버림
    - 특히나 외부 라이브러리일 때, 리턴 타입은 API를 잘 모르는 타인에게 중요한 정보이므로, **명시적으로 지정하는 것이 좋음**
- **타입을 확실하게 지정해야 하는 경우, 명시적으로 타입을 지정해 줘야 함**
