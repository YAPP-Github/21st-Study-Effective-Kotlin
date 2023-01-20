# 아이템 9 : 멤버 확장 함수의 사용을 피해라

### 요약

멤버 확장 함수를 사용해도 의미가 있는 경우 사용하는 것이 좋다.

**DSL을 제외하고는, class나 interface 내에 확장함수를 선언하는 것은 좋지 않음**

```java
@DuckieDsl
internal interface JsonBuilder {
    infix fun String.withInt(value: Int)
    infix fun String.withFloat(value: Float)
```

- **가시성을 제한하지 못함**
- 함수를 사용하기 어려워짐**(클래스 인스턴스 생성 후 사용해야함)**
- **리플렉션 제공하지 않음**(정적 바인딩 되기 때문에)
- 확장 함수가 외부 클래스를 리시버로 받을 때, 직관적이지 않음
    - `아래 경우 a는 A의 것이고, b는 B의 것으로 40이 나온다.`
    
    ```kotlin
    class A {
        val a = 10
    }
    
    class B {
        val a = 20
        val b = 30
    
        fun A.test() = a + b
    }
    
    ```
