# 아이템 7: 추상화 규약을 지켜라

```kotlin
class Employee {
       private fun privateFunction() {
        println("Private function called")
    }
}

fun callPrivateFunction(employee: Employee) {
    employee::class.declaredMemberFunctions
        .first { it.name == "privateFunction" }
        .apply { isAccessible = true }
        .call(employee)
}
```

- 리플렉션을 활용하여, `class`의 `private funtion`을 호출할 수는 있다. 하지만 이는 **private을 외부에서 참조하면 안된다는 규약을 위반한 것이다.**
- 이렇듯 규약은 보증과 같이, **신뢰성**이 있어야만 한다.

## 상속 규약

- 모든 클래스는 `equals`와 `hashCode` 메서드를 가진 `Any` 클래스를 상속받음
    - `equals`와 `hashCode` 가 제대로 구현되지 않는다면, 제대로 동작하지 않을 수 있음
    

## 요약

- 규약을 잘 지키자
- 깰 수밖에 없다면 문서화를 필수로 하자
